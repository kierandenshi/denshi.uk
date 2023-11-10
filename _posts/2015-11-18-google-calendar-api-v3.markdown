---
layout: post
title:  "Google Calendar API v3"
date:   2015-11-18 18:05:21 +0000
categories: google python oauth
---
This month Google will shut down the public XML feeds for their calendar application, which poses a bit of a problem for anyone using that feed to display a parts of their calendar on their website. One of the sites we look after was doing exactly that - pulling the XML, then using a PHP script to parse and select the things they wanted to display on their Wordpress site. While there are a couple of solutions you can use if you are prepared to make your calendar publicly shared (e.g. https://spunmonkey.design/display-contents-google-calendar-php), however if that's not an option you are going to have to roll up your sleeves and get stuck into the api properly, which does mean authorisation using OAuth2. Fortunately, Google has a tool which makes this a less painful experience.

Here's how to get hold of your calendar's events in JSON format. You can use this process to get whatever you need (see https://developers.google.com/google-apps/calendar/v3/reference for full details), but for my purposes I only need the events. Note that you will need a Google account that can access the calendar you want to use, so if you are doing this for a client you may need to create a Google account for this purpose and then ensure that the calendar is shared with this user ID.

First step is to go to https://console.developers.google.com and create a new project. Enable the Calendar API and create an OAuth2 client ID for a web application. Make a note of the resulting Client ID and Client secret. We are going to use https://developers.google.com/oauthplayground to generate authorise the app and generate access tokens (which will save us having to create our own login process which would only be run once) so include "https://developers.google.com/oauthplayground" as a valid callback URI.

Now go to the oauth playground. Open the configuration settings, check "Use your own OAuth credentials" and add your app's Client ID and secret. Make sure "Access type" is set to offline, the rest of the settings can stay as they are.

Having offline access gives us a way to get around our access tokens periodically expiring by granting a refresh token. From [the docs](https://developers.google.com/identity/protocols/OAuth2):

> Access tokens have limited lifetimes. If your application needs access to a Google API beyond the lifetime of a single access token, it can obtain a refresh token. A refresh token allows your application to obtain new access tokens.

To get those tokens, close the settings popup and go through the steps on the left. In Step 1, open up "Calendar API v3" and choose the readonly scope. Hit the "Authorize APIs" button. You'll be presented with an authorisation screen, hit "Allow" to continue. Note that you need to make this authorisation with the Google account that has access to the calendar you want to use.

This will create an authorisation code that can be exchanged for tokens - in Step 2, hit the "Exchange authorization code for tokens" button to get your access and refresh tokens. Note that the access token will expire in an hour, but you can use the refresh token any time to get another one. If you hit the "Refresh access token" button you'll see the request you need to make, and the response you'll get back, in the "Request / Response" panel on the right. In curl, something like this (using your own client_secret, client_id and refresh_token values):

```sh
curl --data "client_secret=************&client_id=************.apps.googleusercontent.com&refresh_token=************_lJvSJLWR3akiL8KibbErbwF6Tyb5Z4rHBactUREZofsF9C7PrpE-j&grant_type=refresh_token" https://www.googleapis.com/oauth2/v3/token
```

You test a request for the calendar data in Step 3 - just add the request's uri (see: https://developers.google.com/google-apps/calendar/v3/reference). For the events, something like:

`https://www.googleapis.com/calendar/v3/calendars/YOUR_CALENDAR_ID@group.calendar.google.com/events`

Hit "Send the request" to see the request and response in the panel. Note the "Authorization:" part of the header in the request - this is the access token generated in Step 2. In curl you would do something like (using your own access_token) :

```sh
curl -H "Authorization: Bearer ya29.************DuZoZuOYINpCYePwyY3aJJsJHnUM-y-CCoLkykApHTdm2cElnlk0MHDHJ" https://www.googleapis.com/calendar/v3/calendars/YOUR_CALENDAR_ID@group.calendar.google.com/events
```

What you get back is JSON document. All we need now is a script that we can run periodically that can go fetch this and stash it somewhere our website scripts can find it - for example, with Python, something like this:

```python
import requests
import json
from datetime import datetime

calendar_id = "YOUR_CALENDAR_ID@group.calendar.google.com"

#set a minimum date as we only want future events
d = datetime.utcnow()
timeMin = d.isoformat("T") + "Z"

#get an access token
payload = {
    "client_id" : "************.apps.googleusercontent.com",
    "client_secret" : "************",
    "refresh_token" : "************-qkqfXJtXbOMtX_DoqDNIgOrJDtdun6zK6XiATCKT",
    "grant_type" : "refresh_token"
}
response = requests.post("https://www.googleapis.com/oauth2/v3/token", data = payload)
data = response.json()

url = "https://www.googleapis.com/calendar/v3/calendars/" + calendar_id + "/events?timeMin=" + timeMin + "&orderBy=startTime&singleEvents=True&futureEvents=True&sortOrder=a"
headers = {
    "Authorization" : "Bearer " + data["access_token"],
    "Content-type" : "Application/json"
}

events = requests.get(url, headers = headers)
data = events.json()
with open('/path/to/save/gcal_data.json', 'w') as f:
    json.dump(data, f)
```

Now your website scripts can open that JSON file using Javascript/PHP/whatever and parse out and format the parts you want to display.
