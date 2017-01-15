#Discord.RSS

Driven by the lack of comprehensive RSS bots available, I have decided to try my hand at creating one of my own. Designed with as much customization as possible for both users and bot hosters, while also (or should be) easy to understand.

####Table of Contents
- [Starting the Bot](#)
	- [Built With](#)
- [Configuration](#)
	- [Database Selection](#)
- [RSS Storage](#)
	- [Feed Customization](#)
		- [Tags](#Tags)
	- [RSS Management](#)
- [Controlling RSS Feeds through Discord](#)
- [Noteworthy Details](#)
- [Author's Note](#)

##Starting the Bot

1. Install Node https://nodejs.org/en/.
2. Clone files into a directory.
3. Use `npm install` in the directory from terminal/command prompt/etc.
4. Create and get a bot token from https://discordapp.com/developers/applications/me.
5. Invite your bot to your server with a link generated from https://discordapi.com/permissions.html, putting your bot's client ID there.
6. Put your bot token and change whatever else you need to in [config.json](https://github.com/synzen/discord-rss#configuration)
7. Create a folder named "sources" in the same directory as server.js
8. Start the bot by `node server.js` in terminal/command prompt/etc.
9. Add feeds either [via Discord](https://github.com/synzen/discord-rss#Controlling-RSS-Feeds-through-Discord), or [manually create](https://github.com/synzen/discord-rss#rss-storage) and [customize](https://github.com/synzen/discord-rss#feed-customization) in the sources folder.

###Built With
* [Node] (https://nodejs.org/en/)
* [Discord.js] (https://www.npmjs.com/package/discord.js)
* [Feedparser] (https://www.npmjs.com/package/feedparser)
* [Request] (https://www.npmjs.com/package/request)
* [Moment] (https://www.npmjs.com/package/moment)
* [striptags] (https://www.npmjs.com/package/striptags)
* Datebase Manager
 * [sqlite3] (https://www.npmjs.com/package/sqlite3) (recommended)
 * [mysql] (https://www.npmjs.com/package/mysql)

##Configuration
(config.json)

1. `token` : Bot token to login through server.js

2. `sqlType`: See [Database Selection](https://github.com/synzen/discord-rss#database-selection)

3. `timezone`: (Optional) By default adding {date}s to your feeds will not show the timezone. Manually specify it here.
For example, normally it will show `Sat, January 7th 2017, 7:18 AM` as the feed's date. Specifying PST timezone will make it print `Sat, January 7th 2017, 7:18 AM (PST)`. This is *purely for visual purposes*.

4. `refreshTimeMinutes`: The bot will check for new feeds regularly at every interval specified in minutes here.

5. `databaseName`: Name of database that will be created.

6. `sendOldMessages`: Send unseen messages that were not caught during bot downtime after it has restarted - this may result in message spam.

7. `defaultMaxAge`: The max aged feed in days that the bot will grab if it unexpected stops.

8. `defaultMessage`: If no custom message is defined for a specific feed, this will be the message the feed will fallback to.

9. `maxFeeds`: The maximum amount of feeds each server is allowed to have.

###Database Selection
I recommend leaving this on `sqlite3`. It can be set to sqlite3 or mysql, however the bot *may* have connection failures after some time with MySQL. sqlite3 however should be working fine.

Should you wish to try and use MySQL (and given that you already know what it is and have it installed), it is quite simple. If you don't already have MySQL installed on your system, [install it](https://dev.mysql.com/downloads/mysql/) and set up the root account password. Then use `npm install mysql` to install the node package for it in the same directory as server.js. Change the login details in mysqlCred.json as well as the `sqlType` in config.json to `mysql`. The bot will handle everything else.

SQLite on the otherhand requires no setup. It will create the database in the same directory as server.js on first startup.

##RSS Storage
Everything is organized by guild ID and handled through the folder  `./sources`. Each JSON file is named with their guild ID, and contains that guild's RSS feeds and customizations. The basic information it must have is `name`, `id`, and `sources` where `sources` is the list of feeds along with their customizations. 

The bottom is an example of what would be in a guild source file, for example `./sources/guild_id_here.json`.

```javascript
"name": "My First Guild!",
"id": "1234567890",
"sources": [{
    //feed #1 settings
    "name": "there",
    "link": "http://somewebsite.com/rss/",
    "channel": "website-feeds"
    }, {
    //feed #2 settings
    }, {
    //feed #3 settings
    }]
}
```

###Feed Customization
The bare minimum for a source must be `name`, `link`, and `channel` for it to be functional. But of course customization is possible! 

1. `name`: Feed Name. If you can, try not to add spaces.

2. `link`: RSS Feed link.

3. `channel`: Can be the channel's ID, or a name. IDs are highly recommended.

4. `message`: Define a custom message for a feed. Use ```\n``` for a new line.

5. `maxAge`: (Optional) If the bot stops unexpectedly, it will grab feeds younger than the maxAge in days and send on bot restart.

6. `filters`: The bot will then only send feeds to Discord with the words defined in these filters.
   * There are three filters available: `title`, `description` and `summary` - they are added as properties of `filters`.
   * For each filter, they can be a string or an array (["filter one!", "two"]) to specify more than one word/phrase. For an feed to pass the filters, every word/phrase defined in filters must exist in their respective filter (case-insensitive).

7. `embedMessage`: Define a custom embed message to go along with the text message.
   * Can be enabled or disabled with the property `enabled` (boolean).
   * This will override the normal embed that Discord sends whenever a link is posted.
   * Properties are defined through embedMessage.properties (as exemplified through the example). Properties include `color` ([*integer* format](https://www.shodor.org/stella2java/rgbint.html)), `authorTitle`, `authorAvatarURL`, `thumbnailURL`, `message`, `footerText`, and `attachURL` (boolean). Note that not all properties are available in every feed as some may return as undefined.

```javascript
	"sources": {
		"name": "there",
		"link": "http://somewebsite.com/rss/",
		"channel": "website-feeds",
		"filters": {
			"title": ["important", "key phrase"],
			"description": "stuff"
		},
		"embedMessage": {
			"enabled": 1,
			"properties": {
				"color": 8816239,
				"message": "My embed message is here!"
			}
		},
		"maxAge": 3
	}
```

####Tags
Putting tags such as {title}, {description}, {summary}, {author}, {link}, {image}, {date} will add the feed's respective information into the text. This can either be in the main message, or in the embed. Regular [Markdown formatting] (https://support.discordapp.com/hc/en-us/articles/210298617-Markdown-Text-101-Chat-Formatting-Bold-Italic-Underline-) is possible wherever Discord allows.

`"message": "{date}\nA new feed has arrived!\n\n**{title}**\n{description}"`

###RSS Management
I don't advise tampering with the `name` of feeds. Everytime a new feed is initialized, a table is created in the database. Manually changing the name of a feed will create a new table for that feed, leaving the old one unmanaged and undeleted unless you manually delete it (or change the name back and remove it through Discord). The names are there more for database management than anything.

In general if you don't want trash lying around in your database don't remove manually remove feeds from `sources`. Instead, remove them from Discord with the command `~rssremove` as explained in the section below. Deleting the channel or removing the bot from the server will also purge any traces of the guild from the configs and the database.

##Controlling RSS Feeds through Discord

Uncomfortable with JSON? No problem! I have scrounged up some commands for you to use. Each command must start with the prefix defined in config.json (literally the first line, set to default as `~`). All of the above features are included.

Each command will open a menu for you to select the RSS in that channel to modify, except `rssadd` which must have a link after it. Whatever you're trying to customize, if it is a non-URL/number field, you can use [tags](https://github.com/synzen/discord-rss#tags) to add the feed's information. The user must have Manage Channels permission to use the commands.

[`rsshelp`]: List the commands to use for Discord.RSS

[`rssadd`]: Add feeds for that specific channel. `(prefix)rssadd rss_link_here`. A new entry will be made in config.json with its name in the format of channelID_feedLink, and will use the default message formatting unless customized otherwise.

[`rssremove`]: To remove feeds. After menu selection, the feed will automatically be removed from config.json.

[`rssmessage`]: Set the custom text message of the feed that will be sent.

[`rssembed`]: Enable and set embed properties to be sent in addition to its regular message.

[`rssfilteradd`]: Add filters for specific categories for a feed.

[`rssfilterremove`]: Remove filters for specific categories for a feed.

[`rsstest`]: Print out the properties for that specific RSS feed and its filter status on whether it passed (if filters exist), along with a randomly chosen feed of any age - in the defined message/embed format in config.json. This was to ease the pains of having to wait for an RSS feed to come just to see how it would look once you designed it in the config.

This is especially useful when you want to add the feed's title and/or description, but you don't know if they'll turn out undefined. However, if the message is too long (that is, over the 2000 character limit), it will not send.


##Noteworthy Details

   * Custom emojis use a different format - it must be in the format of `<:emoji_name:12345>` with 12345 being the emoji's URL ID. The ID can be retrieved by getting the emoji's URL and copying the number in the URL.

   * This bot was made with private server owners in mind. Its stability beyond that is unpredictable.

   * Upon starting the bot with a never before seen RSS feed, it will all store available feeds at that time and put it into the database instead of sending it to Discord. This will prevent your server from being spammed by the bot with messages.
      * Upon starting the bot with an already seen RSS feed, it will retrieve feeds and send it to the Discord server with respect to its `maxAge`, UNLESS `sendOldMessages` is set to false in the config.

   * If you already have a bot active, you can simply use that bot's token and that bot will inherit the functionality of this RSS bot.

   * You can check the validity of your configuration through [JSONLint](http://jsonlint.com/).

##Author's Note

This is in fact my first Javascript project. If you have any potential suggestions/improvements, I'm very open to hearing them out.
