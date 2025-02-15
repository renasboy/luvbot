![Screnshot](https://i.imgur.com//LMRjQJJs.png)

# luvbot
The infamous bot and playlist-manager for the iluvdrumandbass Spotify playlist.

## Setup
First setup a Spotify account and then visit https://developer.spotify.com/dashboard/ and follow the instructions to create an app with **Client ID** and a **Client Secret**
Then in the app settings add http://localhost:3333/callback as redirect URIs and then set up the services as follows:

### refresh-token-getter

In order to have a service that persistantly hits the Spotify API without requiring Oauth2 redirects and a login page
You'll need a refresh token. In order to get a refresh token do the following:

```shell
cd refresh-token-getter
```

copy example.env to a file called .env and edit the file to add the tokens you received from Spotify above.

```shell
cp example.env .env
vim .env
```

install dependency and start the refresh token getter

```shell
npm install
npm run start
```

visit [http://localhost:3333/login](http://localhost:3333/login) or whatever port you set the PORT variable to in your .env file.
**Login with Spotify** and then copy the **refresh token** that appears in the browser after login.

You will need this refresh token for your **now** credentials in the zeit-bot/now.json file you will create.


### zeit-bot  

First you will need a Telegram bot token, this article explains how to get one: https://www.siteguarding.com/en/how-to-get-telegram-bot-api-token

Make sure you have nodejs and npm installed to the latest versions

Install the NOW cli with:

```shell
sudo npm install -g now
```

Copy the example-now.json into a file called now.json and populate the new file with the tokens you got from spotify above
and the other variables described in this file. You can get the chat_id (ROOMS) from https://api.telegram.org/bot{YOUR TELEGRAM BOT TOKEN}/getUpdates
after inviting the bot to a chat

```shell
cp zeit-bot/example-now.json zeit-bot/now.json
```

From inside the zeit-bot folder in your terminal, deploy the service with the command:

```shell
cd zeit-bot/ 
now
```

Connect the instance to your telegram bot via webhook:
```shell
curl -F "url=https://{THE NOW DEPLOYMENT URL RETURNED BY THE COMMAND ABOVE}/new-message" https://api.telegram.org/bot{YOUR TELEGRAM BOT TOKEN}/setWebhook
```

You should now be able to chat with the bot via telegram.

### aws-lambda-scheduler
To prepare for AWS lambda, compress the contents of the folder into a zip named exports.zip (replace the example .zip which is the current deployment or just use serverless or AWS cli to upload to AWS, and set a cloudwatch event to trigger exports.handler on a cron interval.

The current schedule is half hourly exluding sunday:
>cron(1/30 * ? * Mon-Sat *)

The current conditional rule set is as follows:

Every time the playlist runs the script checks all the one track playlists that exist in the list to pull the first (which should really be the only playlist url) in that list.\
The script checks the updated-at date and the uploaded to Spotify date and disqualifies anything that has been added to the label playlist more than a week after its upload to Spotify.\
In order to follow rules for album tracks however it first checks that the track is not part of an album (containing four or more tracks), it will allow album tracks to be posted up to a month after release to allow a track every week from an album for four weeks, with a limit of up to four tracks.\
If the label breaks any of these rules, logging will be stored that the bot can pass to telegram to issue a list of labels added not added, and specific issues with the adding of specific tracks. to_be_logged ordinal specifies the order priority of an event when it is listed in the human readable log.

**Note that you will need to create a link containing the urls to add to the playlist, in the example this is
done using a dropbox link.**




