# Discord Giveaways



Discord Giveaways is a powerful [Node.js](https://nodejs.org) module that allows you to easily create giveaways!

## Features

-   ⏱️ Easy to use!
-   🔄 Automatic restart after bot crash!
-   🇫🇷 Support for translations: adapt the strings for your own language!
-   📁 Support for all databases! (default is json)
-   ⚙️ Very customizable! (prize, duration, winners, ignored permissions, bonus entries etc...)
-   🚀 Super powerful: start, edit, reroll, end, delete giveaways!
-   💥 Events: giveawayEnded, giveawayRerolled, giveawayDeleted, giveawayReactionAdded, giveawayReactionRemoved, endedGiveawayReactionAdded
-   🕸️ Support for shards!
-   and much more!

## Installation

```js
npm install --save discord-giveaways
```

## Examples

You can read this example bot on GitHub: [giveaways-bot](https://github.com/Androz2091/giveaways-bot)

### Launch of the module

```js
const Discord = require('discord.js'),
    client = new Discord.Client(),
    settings = {
        prefix: 'g!',
        token: 'Your Discord Bot Token'
    };

// Requires Manager from discord-giveaways
const { GiveawaysManager } = require('npg-discord-giveaways');
// Starts updating currents giveaways
const manager = new GiveawaysManager(client, {
    storage: './giveaways.json',
    updateCountdownEvery: 10000,
    hasGuildMembersIntent: false,
    default: {
        botsCanWin: false,
        exemptPermissions: ['MANAGE_MESSAGES', 'ADMINISTRATOR'],
        embedColor: '#FF0000',
        embedColorEnd: '#000000',
        reaction: '🎉'
    }
});
// We now have a giveawaysManager property to access the manager everywhere!
client.giveawaysManager = manager;

client.on('ready', () => {
    console.log('I\'m ready!');
});

client.login(settings.token);
```



#### ⚠ ATTENTION!
The command examples below (reroll, edit delete, end) can be executed on any server your bot is a member of if a person has the `prize` or the `messageID`of a giveaway. To prevent abuse we recommend to check if the `prize` or the `messageID` that was provided  by the command user is for a giveaway on the same server, if it is not, then cancel the command execution.

```js
let giveaway = 
// Search with giveaway prize
client.giveawaysManager.giveaways.find((g) => g.guildID === message.guild.id && g.prize === args.join(' ')) ||
// Search with messageID
client.giveawaysManager.giveaways.find((g) => g.guildID === message.guild.id && g.messageID === args[0]);

// If no giveaway was found
if (!giveaway) return message.channel.send('Unable to find a giveaway for `'+ args.join(' ') +'`.');
```

### Reroll a giveaway

```js
client.on('message', (message) => {
    const args = message.content.slice(settings.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();

    if (command === 'reroll') {
        const messageID = args[0];
        client.giveawaysManager.reroll(messageID).then(() => {
            message.channel.send('Success! Giveaway rerolled!');
        }).catch((err) => {
            message.channel.send('No giveaway found for ' + messageID + ', please check and try again');
        });
    }
});
```

### Edit a giveaway

```js
client.on('message', (message) => {
    const args = message.content.slice(settings.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();

    if (command === 'edit') {
        const messageID = args[0];
        client.giveawaysManager.edit(messageID, {
            addTime: 5000,
            newWinnerCount: 3,
            newPrize: 'New Prize!'
        }).then(() => {
            // Here, we can calculate the time after which we are sure that the lib will update the giveaway
            const numberOfSecondsMax = client.giveawaysManager.options.updateCountdownEvery / 1000;
            message.channel.send('Success! Giveaway will updated in less than ' + numberOfSecondsMax + ' seconds.');
        }).catch((err) => {
            message.channel.send('No giveaway found for ' + messageID + ', please check and try again');
        });
    }
});
```

-   **options.newWinnerCount**: the new number of winners.  
-   **options.newPrize**: the new prize.  
-   **options.addTime**: the number of milliseconds to add to the giveaway duration.
-   **options.setEndTimestamp**: the timestamp of the new end date (for example, for the giveaway to be ended in 1 hour, set it to `Date.now() + 60000`).
-   **options.newMessages**: the new giveaway messages
-   **options.newExtraData**: the new extra data value for the giveaway
-   **options.newBonusEntries**: the new BonusEntry objects (for example, to change the amount of entries).

⚠️ **Note**: to reduce giveaway time, define `addTime` with a negative number! For example `addTime: -5000` will reduce giveaway time by 5 seconds!

### Delete a giveaway

```js
client.on('message', (message) => {
    const args = message.content.slice(settings.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();

    if (command === 'delete') {
        const messageID = args[0];
        client.giveawaysManager.delete(messageID).then(() => {
            message.channel.send('Success! Giveaway deleted!');
        }).catch((err) => {
            message.channel.send('No giveaway found for ' + messageID + ', please check and try again');
        });
    }
});
```

-   **doNotDeleteMessage**: whether the giveaway message shouldn't be deleted.

⚠️ **Note**: when you use the delete function, the giveaway data and per default the message of the giveaway are deleted. You cannot restore a giveaway once you have deleted it!

### End a giveaway

```js
client.on('message', (message) => {
    const args = message.content.slice(settings.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();

    if (command === 'end') {
        const messageID = args[0];
        client.giveawaysManager.end(messageID).then(() => {
            message.channel.send('Success! Giveaway ended!');
        }).catch((err) => {
            message.channel.send('No giveaway found for ' + messageID + ', please check and try again');
        });
    }
});
```

### Fetch giveaways

```js
// A list of all the giveaways
const allGiveaways = client.giveawaysManager.giveaways; // [ {Giveaway}, {Giveaway} ]

// A list of all the giveaways on the server with ID "1909282092"
const onServer = client.giveawaysManager.giveaways.filter(g => g.guildID === '1909282092');

// A list of the current active giveaways (not ended)
const notEnded = client.giveawaysManager.giveaways.filter(g => !g.ended);
```

### Exempt Members

```js
client.giveawaysManager.start(message.channel, {
    time: 60000,
    winnerCount: 1,
    prize: 'Free Steam Key',
    // Only members who have the "Nitro Boost" role are able to win
    exemptMembers: (member) => !member.roles.cache.some((r) => r.name === 'Nitro Boost')
})
```

⚠️ **Note**: If the function should be customizable

```js
const roleName = 'Nitro Boost';

client.giveawaysManager.start(message.channel, {
    time: 60000,
    winnerCount: 1,
    prize: 'Free Steam Key',
    // Only members who have the the role which is assigned to "roleName" are able to win
    exemptMembers: new Function('member', `return !member.roles.cache.some((r) => r.name === \'${roleName}\')`),
})
```

### Last Chance

```js
client.giveawaysManager.start(message.channel, {
    time: 60000,
    winnerCount: 1,
    prize: 'Discord Nitro!',
    lastChance: {
        enabled: true,
        content: '⚠️ **LAST CHANCE TO ENTER !** ⚠️',
        threshold: 5000,
        embedColor: '#FF0000'
    }
})
```


### Bonus Entries

```js
client.giveawaysManager.start(message.channel, {
    time: 60000,
    winnerCount: 1,
    prize: 'Free Steam Key',
    bonusEntries: [
        // Members who have the "Nitro Boost" role get 2 bonus entries
        {
            bonus: (member) => member.roles.cache.some((r) => r.name === 'Nitro Boost') ? 2 : null,
            cumulative: false
        }
    ]
})
```

⚠️ **Note**: If the `bonus` function should be customizable

```js
const roleName = 'Nitro Boost';
const roleBonusEntries = 2;

client.giveawaysManager.start(message.channel, {
    time: 60000,
    winnerCount: 1,
    prize: 'Free Steam Key',
    bonusEntries: [
        // Members who have the role which is assigned to "roleName" get the amount of bonus entries which are assigned to "roleBonusEntries"
        {   
            bonus: new Function('member', `return member.roles.cache.some((r) => r.name === \'${roleName}\') ? ${roleBonusEntries} : null`),
            cumulative: false 
        }
    ]
})
```

## 🇫🇷 Translation

You can also pass a `messages` parameter for `start()` function, if you want to translate the bot text:

-   **options.messages.giveaway**: the message that will be displayed above the embeds.
-   **options.messages.giveawayEnded**: the message that will be displayed above the embeds when the giveaway is ended.
-   **options.messages.timeRemaining**: the message that displays the remaining time (the timer).
-   **options.messages.inviteToParticipate**: the message that invites users to participate.
-   **options.messages.winMessage**: the message that will be displayed to congratulate the winner(s) when the giveaway is ended.
-   **options.messages.embedFooter**: the message displayed at the bottom of the embeds.
-   **options.messages.noWinner**: the message that is displayed if no winner can be drawn.
-   **options.messages.winners**: simply the word "winner" in your language.
-   **options.messages.endedAt**: simply the words "Ended at" in your language.
-   **options.messages.units.seconds**: simply the word "seconds" in your language.
-   **options.messages.units.minutes**: simply the word "minutes" in your language.
-   **options.messages.units.hours**: simply the word "hours" in your language.
-   **options.messages.units.days**: simply the word "days" in your language.

**Note**: units should be in the plural.

For example:

```js
client.giveawaysManager.start(message.channel, {
    time: ms(args[0]),
    winnerCount: parseInt(args[1]),
    prize: args.slice(2).join(' '),
    messages: {
        giveaway: '@everyone\n\n🎉🎉 **GIVEAWAY** 🎉🎉',
        giveawayEnded: '@everyone\n\n🎉🎉 **GIVEAWAY ENDED** 🎉🎉',
        timeRemaining: 'Time remaining: **{duration}**',
        inviteToParticipate: 'React with 🎉 to participate!',
        winMessage: 'Congratulations, {winners}! You won **{prize}**!\n{messageURL}',
        embedFooter: 'Powered by the discord-giveaways package',
        noWinner: 'Giveaway cancelled, no valid participations.',
        hostedBy: 'Hosted by: {user}',
        winners: 'winner(s)',
        endedAt: 'Ended at',
        units: {
            seconds: 'seconds',
            minutes: 'minutes',
            hours: 'hours',
            days: 'days',
            pluralS: false // Not needed, because units end with a S so it will automatically removed if the unit value is lower than 2
        }
    }
});
```

And for the `reroll()` function:

```js
client.giveawaysManager.reroll(messageID, {
        messages: {
            congrat: ':tada: New winner(s): {winners}! Congratulations, you won **{prize}**!\n{messageURL}',
            error: 'No valid participations, no new winner(s) can be chosen!'
        }
    }).catch((err) => {
        message.channel.send('No giveaway found for ' + messageID + ', please check and try again');
    });
```

-   **options.messages.congrat**: the congratulatory message.  
-   **options.messages.error**: the error message if there is no valid participant.

## Custom Database

You can use your custom database to save giveaways, instead of the json files (the "database" by default for `discord-giveaways`). For this, you will need to extend the `GiveawaysManager` class, and replace some methods with your custom ones. There are 4 methods you will need to replace:

-   `getAllGiveaways`: this method returns an array of stored giveaways.
-   `saveGiveaway`: this method stores a new giveaway in the database.
-   `editGiveaway`: this method edits a giveaway already stored in the database.
-   `deleteGiveaway`: this method deletes a giveaway from the database (permanently).

**⚠️ All the methods should be asynchronous to return a promise!**

Here is an example, using `quick.db`, a SQLite database. The comments in the code below are very important to understand how it works!

Other examples:

- [MySQL](https://github.com/Androz2091/discord-giveaways/blob/master/examples/custom-databases/mysql.js)
- MongoDB
  - [Mongoose](https://github.com/Androz2091/discord-giveaways/blob/master/examples/custom-databases/mongoose.js)
  - [QuickMongo](https://github.com/Androz2091/discord-giveaways/blob/master/examples/custom-databases/quickmongo.js) ⚠️ Not recommended for high giveaway usage, use the `mongoose` example instead

```js
const Discord = require('discord.js'),
    client = new Discord.Client(),
    settings = {
        prefix: 'g!',
        token: 'Your Discord Bot Token'
    };

// Load quick.db - it's an example of custom database, you can use MySQL, PostgreSQL, etc...
const db = require('quick.db');
if (!Array.isArray(db.get('giveaways'))) db.set('giveaways', []);

const { GiveawaysManager } = require('npg-discord-giveaways');
const GiveawayManagerWithOwnDatabase = class extends GiveawaysManager {
    // This function is called when the manager needs to get all giveaways which are stored in the database.
    async getAllGiveaways() {
        // Get all giveaways from the database
        return db.get('giveaways');
    }

    // This function is called when a giveaway needs to be saved in the database.
    async saveGiveaway(messageID, giveawayData) {
        // Add the new giveaway to the database
        db.push('giveaways', giveawayData);
        // Don't forget to return something!
        return true;
    }

    // This function is called when a giveaway needs to be edited in the database.
    async editGiveaway(messageID, giveawayData) {
        // Get all giveaways from the database
        const giveaways = db.get('giveaways');
        // Remove the unedited giveaway from the array
        const newGiveawaysArray = giveaways.filter((giveaway) => giveaway.messageID !== messageID);
        // Push the edited giveaway into the array
        newGiveawaysArray.push(giveawayData);
        // Save the updated array
        db.set('giveaways', newGiveawaysArray);
        // Don't forget to return something!
        return true;
    }

    // This function is called when a giveaway needs to be deleted from the database.
    async deleteGiveaway(messageID) {
        // Get all giveaways from the database
        const giveaways = db.get('giveaways');
        // Remove the giveaway from the array
        const newGiveawaysArray = giveaways.filter((giveaway) => giveaway.messageID !== messageID);
        // Save the updated array
        db.set('giveaways', newGiveawaysArray);
        // Don't forget to return something!
        return true;
    }
};

// Create a new instance of your new class
const manager = new GiveawayManagerWithOwnDatabase(client, {
    updateCountdownEvery: 10000,
    default: {
        botsCanWin: false,
        exemptPermissions: ['MANAGE_MESSAGES', 'ADMINISTRATOR'],
        embedColor: '#FF0000',
        embedColorEnd: '#000000',
        reaction: '🎉'
    }
});
// We now have a giveawaysManager property to access the manager everywhere!
client.giveawaysManager = manager;

client.on('ready', () => {
    console.log('I\'m ready!');
});

client.login(settings.token);
```

## Support shards

To make `discord-giveaways` working with shards, you will need to extend the `GiveawaysManager` class and update the `refreshStorage()` method. This method should call the `getAllGiveaways()` method for **every** shard, so all `GiveawaysManager` synchronize their cache with the updated database.

```js
const Discord = require('discord.js'),
    client = new Discord.Client(),
    settings = {
        prefix: 'g!',
        token: 'Your Discord Bot Token'
    };

// Extends the GiveawaysManager class and update the refreshStorage method
const { GiveawaysManager } = require('npm-discord-giveaways');
const GiveawayManagerWithShardSupport = class extends GiveawaysManager {
    // Refresh storage method is called when the database is updated on one of the shards
    async refreshStorage() {
        // This should make all shard refreshing their cache with the updated database
        return client.shard.broadcastEval(() => this.giveawaysManager.getAllGiveaways());
    }
};

// Create a new instance of your new class
const manager = new GiveawayManagerWithShardSupport(client, {
    storage: './storage.json',
    updateCountdownEvery: 10000,
    default: {
        botsCanWin: false,
        exemptPermissions: ['MANAGE_MESSAGES', 'ADMINISTRATOR'],
        embedColor: '#FF0000',
        embedColorEnd: '#000000',
        reaction: '🎉'
    }
});
// We now have a giveawaysManager property to access the manager everywhere!
client.giveawaysManager = manager;

client.on('ready', () => {
    console.log('I\'m ready!');
});

client.login(settings.token);
```
