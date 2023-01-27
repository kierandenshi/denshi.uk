---
layout: post
title:  "Formatting localised monetary values with Javascript"
date:   2017-05-08 16:30:00 +0000
categories: javascript money currencies localisation
published: true
---
Displaying numbers and currencies as a part of a product description, generated quotation or as values in a checkout is likely a requirement for many apps and services. With the US-English hegemony of the web, appropriately formatting these values for users from other locales could be seen as non-essential, but if a product is truly global, I think it's worth taking the time with the details. For example, _"Forty Five Thousand Euro"_ will usually be displayed as `€45,000.00` in the US (and UK), in France and Germany the same number would be more commonly be displayed as `45 000,00 €` and `45.000,00 €` respectively. While there is almost certainly a very comprehensive Javascript localisation library available somewhere, if only a few locale and currency options are required it's possible to save some kilobytes by making our own helper module.

With monetary values there are two variables which affect the final string - currency and lang/locale. The currency will dictate which symbol to use and the currency's subdivision, which will dictate how many decimal places to display (examples: the Euro is subdivided 1/100, which requires 2 decimal places; the Kuwaiti Dinar is subdivided 1/1000, requiring 3 decimal places; Japanese Yen has no subdivision, requiring 0 decimal places). The lang/locale will introduce variances in the character used for the decimal seperator, the character used for the thousands seperator, and the placement of the currency code or symbol.


### Getting the currency symbol
Nothing fancy here, just a simple hash table lookup. For brevity, only the symbols relevant to this example will be included here. A more comprehensive list can be found [in this gist][1].

```js
const getSymbolFromCurrencyCode = (currencyCode = 'EUR') => {
  const map = {
    EUR: '€',
    GBP: '£',
    USD: '$'
  };

  const code = currencyCode.toUpperCase();
  return map.hasOwnProperty(code) ? map[code] : '';
};
```


### Adding appropriate separators
A straight forward way to do this is with a recursive regular expression string replacement, so this function will take a String argument which will be the full, fractional representation of the number to be displayed. An optional second argument can provide overrides to the default configuration for fractional and thousands separators.

```js
const addSeparators = (numString, { fractionalSeparator = '.', thousandSeparator = ',' }) => {
  // variables to store the numString sub-parts before and after the decimal point
  let numStringStart = numString;
  let numStringEnd = '';

  // find the position of the decimal point
  const dpos = numString.indexOf('.');

  // if there is a decimal point, update the numString sub-part variables
  if (dpos !== -1) {
    numStringStart = numString.substring(0, dpos);
    numStringEnd = `${fractionalSeparator}${numString.substring(dpos + 1, numString.length)}`;
  }

  // the regex has 2 capturing groups - the first group is a greedy
  // match for numeric characters, the second group matches exactly 3 numbers.
  // use the matching groups to update the numString, placing the thousands
  // separator between the first and second match groups.
  // keep doing this until there are no more regex matches.
  const rgx = /(\d+)(\d{3})/;
  while (rgx.test(numStringStart)) {
    numStringStart = numStringStart.replace(rgx, `$1${thousandSeparator}$2`);
  }

  // return the reconstructed numString sub-parts
  return `${numStringStart}${numStringEnd}`;
};
```

### The orchestrator function
This is the function that will be exported. In addition to taking parameters for the number value, local and currency code strings, the function will take a boolean to indicate use of the currency symbol, and the number of decimal points required<sup>1</sup>.   

```js
const formatCurrency = (value, localeCode = 'en', currencyCode = 'EUR', useSymbol = true, precision = 2) => {
  // get the appropriate currency symbol (if required)
  const sym = useSymbol ? getSymbolFromCurrencyCode(currencyCode) : currencyCode;

  // cast the numeric value into a string with appropriate fixed fractional part
  const numString = String(Number(value).toFixed(precision));

  // put it all together
  switch (localeCode) {

    case 'de':
    case 'de-DE':
    return `${addSeparators(numString, { fractionalSeparator: ',', thousandSeparator: '.' })} ${sym}`;

    case 'fr':
    return `${addSeparators(numString, { fractionalSeparator: ',', thousandSeparator: ' ' })} ${sym}`;

    case 'en':
    case 'en-GB':
    default:
    return `${sym}${addSeparators(numString)}`;
  }
};
```

In real world use it's likely best to curry this function to make individual incantations more concise - for example, as all that usually changes is the `value` parameter, the helper module can export the `formatCurrency` function like this

```js
export default (localeCode = 'en', currencyCode = 'EUR', useSymbol = true, precision = 2) => value => {
  ...
}
```

and then import and use like this

```js
import formatCurrency from './path/to/module';

const GB_money = formatCurrency('en-GB', 'GBP');
const FR_money = formatCurrency('fr', 'EUR');
const DE_money = formatCurrency('de', 'EUR', false);

console.log(GB_money(45000)); // £45,000.00
console.log(FR_money(45000)); // 45 000,00 €
console.log(DE_money(45000)); // 45.000,00 EUR
```

### Notes
<sup>1</sup>Rather than relying on decimal precision as a parameter, it would be safer to include this value in the currencies hash table and rework the `getSymbolFromCurrencyCode` helper function to return this value along with the currency symbol. In practise, I am yet to work on an application that supports currencies with anything other than a 1/100 sub unit.

[1]: https://gist.github.com/kierandenshi/8a0a41c57a894bf5f89e744849c6f893
