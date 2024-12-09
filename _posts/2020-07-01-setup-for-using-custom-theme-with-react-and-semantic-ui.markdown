---
layout: post
title:  "Setup for using a custom theme with React and Semantic-UI"
date:   2020-07-01 15:20:00 +0000
categories: react css semantic
published: false
---
[Semantic UI][6], like Bootstrap, is a common way to get a UI project up and running before having a design team onboard. At some point it's going to be neccessary to make a few changes to the styling, and Semantic does have comprehensive theming capabilities. Read on for help with using themes with [the React version of Semantic][1] when you have a custom `webpack.config.js`.

CREDIT: The following is an update to [this article][4], replacing the deprecated [Extract Text Webpack Plugin][7] with [Mini CSS Extract Plugin][3] and adding in additional loader instructions in case these are not already present in our Webpack config.

Ok, so first we create a directory at the root of the React project named `semantic-ui`

> we can use any name for this directory, but remember to update uses of `semantic-ui` for the paths in the examples shown.

Clone or download [https://github.com/Semantic-Org/Semantic-UI-LESS][2] and copy:

- `_site`
- `themes`
- `theme.config.example`

into this directory, renaming `_site` to `site` and `theme.config.example` to `theme.config`.

Open up `theme.config` and update the paths and imports sections

```css
/*******************************
            Folders
*******************************/

/* Path to theme packages */
@themesFolder : 'themes';

/* Path to site override folder */
@siteFolder  : '../semantic-ui/site';


/*******************************
         Import Theme
*******************************/

@import (multiple) "~semantic-ui-less/theme.less";
@fontPath : "../../../themes/@{theme}/assets/fonts";
```

Open up `semantic-ui/site/globals/site.variables` and add an override so we can check it's working - lets make the background orange...

```css
/*******************************
     User Global Variables
*******************************/

@pageBackground: #ff800;
```

Ok so now lets update our Webpack config.

Add Semantic UI's less package, some required Webpack loaders, and the Mini CSS Extract Plugin.

```sh
yarn add --dev semantic-ui-less less less-loader css-loader file-loader mini-css-extract-plugin
```


Update `webpack.config.js` to:
- import mini css extract plugin
- add a rule for `.less` files
- add rules for the fonts and images in the Semantic UI base
- add mini css plugin to the plugins list
- create an alias to resolve the path to the theme directory we just created

```javascript
...
const MiniCssExtractPlugin = require('mini-css-extract-plugin');


module.exports = {
    ...,
    module: {
        rules: [
            ...,
            {
                test: /\.(less)$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'less-loader'
                ]
            },
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                use: {
                    loader: 'file-loader'
                }
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: {
                    loader: 'file-loader'
                }
            }
        ],
        plugins: [
            ...,
            new MiniCssExtractPlugin()
        ],
        resolve: {
            ...,
            alias: {
                '../../theme.config$': join(workdir, '/semantic-ui/theme.config'),
                '../semantic-ui/site': join(workdir, '/semantic-ui/site')
            }
        ...
        }
    }
}
```

Lastly, import the Semantic UI less file into our app's entry file (usually this is `/index.js`)

```javascript
import 'semantic-ui-less/semantic.less'
```

As we changed our Webpack config, we need to restart the dev server to have the changes take effect.

More information for theming can be found in the [Semantic UI documentation][5].


[1]: https://react.semantic-ui.com
[2]: https://github.com/Semantic-Org/Semantic-UI-LESS
[3]: https://webpack.js.org/plugins/mini-css-extract-plugin/
[4]: https://medium.com/@marekurbanowicz/how-to-customize-fomantic-ui-with-less-and-webpack-applicable-to-semantic-ui-too-fbf98a74506c
[5]: https://semantic-ui.com/usage/theming.html
[6]: https://semantic-ui.com
[7]: https://webpack.js.org/plugins/extract-text-webpack-plugin/
