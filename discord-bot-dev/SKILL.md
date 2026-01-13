---
name: discord-bot-dev
description: |
  Expert assistance for Discord bot development using Discord.js. Covers bot setup, commands (slash & prefix), events, interactions (buttons, select menus, modals), permissions, embeds, voice features, database integration, and deployment.
  Triggers: discord bot, discord.js, slash commands, discord events, discord interactions, bot commands, discord embed, discord permissions, discord voice, discord api, message components, discord modals, discord roles
---

# Discord Bot Development Skill

This skill provides comprehensive guidance for creating Discord bots with any features using Discord.js (v14+).

## Table of Contents

1. [Getting Started](#getting-started)
2. [Project Structure](#project-structure)
3. [Bot Configuration](#bot-configuration)
4. [Core Concepts](#core-concepts)
5. [Command Systems](#command-systems)
6. [Event Handling](#event-handling)
7. [Interactions & Components](#interactions--components)
8. [Common Features](#common-features)
9. [Advanced Features](#advanced-features)
10. [Database Integration](#database-integration)
11. [Deployment](#deployment)
12. [Best Practices](#best-practices)
13. [Code Examples](#code-examples)

---

## Getting Started

### Prerequisites

- Node.js 16.11.0 or higher
- A Discord account
- Discord Developer Portal access

### Initial Setup

1. **Create a Discord Application**
   - Go to https://discord.com/developers/applications
   - Click "New Application"
   - Navigate to the "Bot" section
   - Click "Add Bot"
   - Enable necessary Privileged Gateway Intents (Message Content, Server Members, Presence)
   - Copy the bot token (keep it secret!)

2. **Generate Bot Invite URL**
   - Go to OAuth2 > URL Generator
   - Select scopes: `bot`, `applications.commands`
   - Select bot permissions based on your needs
   - Use the generated URL to invite the bot to your server

3. **Initialize Project**
   ```bash
   mkdir my-discord-bot
   cd my-discord-bot
   npm init -y
   npm install discord.js dotenv
   npm install --save-dev @types/node typescript
   ```

4. **Create .env file**
   ```env
   DISCORD_TOKEN=your_bot_token_here
   CLIENT_ID=your_client_id_here
   GUILD_ID=your_test_server_id_here
   ```

---

## Project Structure

### Recommended Directory Layout

```
my-discord-bot/
├── src/
│   ├── commands/           # Slash command files
│   │   ├── utility/
│   │   │   ├── ping.js
│   │   │   └── help.js
│   │   ├── moderation/
│   │   │   ├── ban.js
│   │   │   └── kick.js
│   │   └── fun/
│   │       └── meme.js
│   ├── events/            # Event handler files
│   │   ├── ready.js
│   │   ├── interactionCreate.js
│   │   └── messageCreate.js
│   ├── buttons/           # Button interaction handlers
│   ├── selectMenus/       # Select menu handlers
│   ├── modals/            # Modal handlers
│   ├── utils/             # Utility functions
│   │   ├── logger.js
│   │   └── embeds.js
│   ├── config/            # Configuration files
│   │   └── config.js
│   ├── models/            # Database models (if using DB)
│   ├── deploy-commands.js # Command deployment script
│   └── index.js           # Main bot file
├── .env                   # Environment variables
├── .gitignore
├── package.json
└── README.md
```

---

## Bot Configuration

### Main Bot File (index.js)

```javascript
const { Client, GatewayIntentBits, Collection } = require('discord.js');
const fs = require('fs');
const path = require('path');
require('dotenv').config();

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.GuildMembers,
        GatewayIntentBits.GuildVoiceStates,
    ]
});

// Collections for commands and cooldowns
client.commands = new Collection();
client.buttons = new Collection();
client.selectMenus = new Collection();
client.modals = new Collection();
client.cooldowns = new Collection();

// Load commands
const commandsPath = path.join(__dirname, 'commands');
const loadCommands = (dir) => {
    const files = fs.readdirSync(dir);
    for (const file of files) {
        const filePath = path.join(dir, file);
        const stat = fs.statSync(filePath);
        if (stat.isDirectory()) {
            loadCommands(filePath);
        } else if (file.endsWith('.js')) {
            const command = require(filePath);
            if ('data' in command && 'execute' in command) {
                client.commands.set(command.data.name, command);
            }
        }
    }
};
loadCommands(commandsPath);

// Load events
const eventsPath = path.join(__dirname, 'events');
const eventFiles = fs.readdirSync(eventsPath).filter(file => file.endsWith('.js'));

for (const file of eventFiles) {
    const filePath = path.join(eventsPath, file);
    const event = require(filePath);
    if (event.once) {
        client.once(event.name, (...args) => event.execute(...args));
    } else {
        client.on(event.name, (...args) => event.execute(...args));
    }
}

// Login
client.login(process.env.DISCORD_TOKEN);
```

### TypeScript Configuration (Optional)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

---

## Core Concepts

### Gateway Intents

Intents determine what events your bot receives:

```javascript
// Common intents
GatewayIntentBits.Guilds              // Guild/server info
GatewayIntentBits.GuildMessages       // Message events (without content)
GatewayIntentBits.MessageContent      // PRIVILEGED: Actual message content
GatewayIntentBits.GuildMembers        // PRIVILEGED: Member join/leave/update
GatewayIntentBits.GuildPresences      // PRIVILEGED: User status/activities
GatewayIntentBits.GuildVoiceStates    // Voice state updates
GatewayIntentBits.GuildMessageReactions // Reaction events
GatewayIntentBits.DirectMessages      // DM events
```

**Important:** Privileged intents must be enabled in the Discord Developer Portal.

### Permissions

```javascript
const { PermissionFlagsBits } = require('discord.js');

// Check if user has permission
if (interaction.member.permissions.has(PermissionFlagsBits.Administrator)) {
    // User is admin
}

// Check bot permissions
if (interaction.guild.members.me.permissions.has(PermissionFlagsBits.BanMembers)) {
    // Bot can ban members
}

// Common permissions
PermissionFlagsBits.Administrator
PermissionFlagsBits.ManageGuild
PermissionFlagsBits.ManageRoles
PermissionFlagsBits.ManageChannels
PermissionFlagsBits.KickMembers
PermissionFlagsBits.BanMembers
PermissionFlagsBits.ManageMessages
PermissionFlagsBits.SendMessages
PermissionFlagsBits.EmbedLinks
PermissionFlagsBits.AttachFiles
```

---

## Command Systems

### Slash Commands (Modern Approach)

#### Command File Structure

```javascript
// commands/utility/ping.js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('ping')
        .setDescription('Replies with Pong!'),
    async execute(interaction) {
        await interaction.reply('Pong!');
    },
};
```

#### Command with Options

```javascript
// commands/utility/user-info.js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('userinfo')
        .setDescription('Get information about a user')
        .addUserOption(option =>
            option
                .setName('target')
                .setDescription('The user to get info about')
                .setRequired(true)
        ),
    async execute(interaction) {
        const user = interaction.options.getUser('target');
        const member = interaction.options.getMember('target');

        await interaction.reply(
            `User: ${user.tag}\n` +
            `ID: ${user.id}\n` +
            `Joined: ${member.joinedAt?.toDateString()}`
        );
    },
};
```

#### Deploying Commands

```javascript
// deploy-commands.js
const { REST, Routes } = require('discord.js');
const fs = require('fs');
const path = require('path');
require('dotenv').config();

const commands = [];
const commandsPath = path.join(__dirname, 'commands');

const loadCommands = (dir) => {
    const files = fs.readdirSync(dir);
    for (const file of files) {
        const filePath = path.join(dir, file);
        const stat = fs.statSync(filePath);
        if (stat.isDirectory()) {
            loadCommands(filePath);
        } else if (file.endsWith('.js')) {
            const command = require(filePath);
            if ('data' in command) {
                commands.push(command.data.toJSON());
            }
        }
    }
};

loadCommands(commandsPath);

const rest = new REST().setToken(process.env.DISCORD_TOKEN);

(async () => {
    try {
        console.log(`Deploying ${commands.length} commands...`);

        // Deploy to single guild (faster for testing)
        await rest.put(
            Routes.applicationGuildCommands(process.env.CLIENT_ID, process.env.GUILD_ID),
            { body: commands }
        );

        // OR deploy globally (takes up to 1 hour to propagate)
        // await rest.put(
        //     Routes.applicationCommands(process.env.CLIENT_ID),
        //     { body: commands }
        // );

        console.log('Successfully deployed commands!');
    } catch (error) {
        console.error(error);
    }
})();
```

### Prefix Commands (Legacy)

```javascript
// events/messageCreate.js
const prefix = '!';

module.exports = {
    name: 'messageCreate',
    execute(message) {
        if (!message.content.startsWith(prefix) || message.author.bot) return;

        const args = message.content.slice(prefix.length).trim().split(/ +/);
        const commandName = args.shift().toLowerCase();

        const command = message.client.commands.get(commandName);
        if (!command) return;

        try {
            command.execute(message, args);
        } catch (error) {
            console.error(error);
            message.reply('There was an error executing that command!');
        }
    },
};
```

---

## Event Handling

### Ready Event

```javascript
// events/ready.js
module.exports = {
    name: 'ready',
    once: true,
    execute(client) {
        console.log(`Logged in as ${client.user.tag}!`);
        client.user.setActivity('with Discord.js', { type: 'PLAYING' });
    },
};
```

### Interaction Create (Slash Commands)

```javascript
// events/interactionCreate.js
module.exports = {
    name: 'interactionCreate',
    async execute(interaction) {
        // Handle slash commands
        if (interaction.isChatInputCommand()) {
            const command = interaction.client.commands.get(interaction.commandName);
            if (!command) return;

            try {
                await command.execute(interaction);
            } catch (error) {
                console.error(error);
                await interaction.reply({
                    content: 'There was an error executing this command!',
                    ephemeral: true
                });
            }
        }

        // Handle buttons
        if (interaction.isButton()) {
            const button = interaction.client.buttons.get(interaction.customId);
            if (button) await button.execute(interaction);
        }

        // Handle select menus
        if (interaction.isStringSelectMenu()) {
            const menu = interaction.client.selectMenus.get(interaction.customId);
            if (menu) await menu.execute(interaction);
        }

        // Handle modals
        if (interaction.isModalSubmit()) {
            const modal = interaction.client.modals.get(interaction.customId);
            if (modal) await modal.execute(interaction);
        }
    },
};
```

### Message Events

```javascript
// events/messageCreate.js
module.exports = {
    name: 'messageCreate',
    async execute(message) {
        if (message.author.bot) return;

        // Auto-respond to certain keywords
        if (message.content.toLowerCase().includes('hello')) {
            await message.reply('Hello there!');
        }

        // Log deleted messages
        const deletedMessages = new Map();
        deletedMessages.set(message.id, message);
    },
};
```

### Member Events

```javascript
// events/guildMemberAdd.js
const { EmbedBuilder } = require('discord.js');

module.exports = {
    name: 'guildMemberAdd',
    async execute(member) {
        const welcomeChannel = member.guild.channels.cache.find(
            ch => ch.name === 'welcome'
        );

        if (!welcomeChannel) return;

        const embed = new EmbedBuilder()
            .setColor('#00ff00')
            .setTitle('Welcome!')
            .setDescription(`Welcome to the server, ${member}!`)
            .setThumbnail(member.user.displayAvatarURL())
            .setTimestamp();

        await welcomeChannel.send({ embeds: [embed] });
    },
};
```

---

## Interactions & Components

### Buttons

```javascript
// Send a message with buttons
const { ActionRowBuilder, ButtonBuilder, ButtonStyle } = require('discord.js');

const row = new ActionRowBuilder()
    .addComponents(
        new ButtonBuilder()
            .setCustomId('primary')
            .setLabel('Primary')
            .setStyle(ButtonStyle.Primary),
        new ButtonBuilder()
            .setCustomId('success')
            .setLabel('Success')
            .setStyle(ButtonStyle.Success),
        new ButtonBuilder()
            .setCustomId('danger')
            .setLabel('Danger')
            .setStyle(ButtonStyle.Danger),
        new ButtonBuilder()
            .setLabel('Link')
            .setURL('https://discord.js.org')
            .setStyle(ButtonStyle.Link)
    );

await interaction.reply({ content: 'Choose an option:', components: [row] });

// Handle button click (in events/interactionCreate.js)
if (interaction.isButton()) {
    if (interaction.customId === 'primary') {
        await interaction.reply({ content: 'You clicked Primary!', ephemeral: true });
    }
}
```

### Select Menus

```javascript
const { StringSelectMenuBuilder, StringSelectMenuOptionBuilder } = require('discord.js');

const select = new StringSelectMenuBuilder()
    .setCustomId('color-select')
    .setPlaceholder('Select a color')
    .addOptions(
        new StringSelectMenuOptionBuilder()
            .setLabel('Red')
            .setValue('red')
            .setDescription('The color red'),
        new StringSelectMenuOptionBuilder()
            .setLabel('Blue')
            .setValue('blue')
            .setDescription('The color blue'),
        new StringSelectMenuOptionBuilder()
            .setLabel('Green')
            .setValue('green')
            .setDescription('The color green')
    );

const row = new ActionRowBuilder().addComponents(select);
await interaction.reply({ content: 'Choose a color:', components: [row] });

// Handle selection
if (interaction.isStringSelectMenu()) {
    const selectedValue = interaction.values[0];
    await interaction.reply(`You selected: ${selectedValue}`);
}
```

### Modals (Forms)

```javascript
const { ModalBuilder, TextInputBuilder, TextInputStyle } = require('discord.js');

// Show modal
const modal = new ModalBuilder()
    .setCustomId('feedback-modal')
    .setTitle('Feedback Form');

const nameInput = new TextInputBuilder()
    .setCustomId('name')
    .setLabel('What is your name?')
    .setStyle(TextInputStyle.Short)
    .setRequired(true);

const feedbackInput = new TextInputBuilder()
    .setCustomId('feedback')
    .setLabel('Your feedback')
    .setStyle(TextInputStyle.Paragraph)
    .setRequired(true);

const firstRow = new ActionRowBuilder().addComponents(nameInput);
const secondRow = new ActionRowBuilder().addComponents(feedbackInput);

modal.addComponents(firstRow, secondRow);
await interaction.showModal(modal);

// Handle modal submission (in events/interactionCreate.js)
if (interaction.isModalSubmit() && interaction.customId === 'feedback-modal') {
    const name = interaction.fields.getTextInputValue('name');
    const feedback = interaction.fields.getTextInputValue('feedback');
    await interaction.reply(`Thank you for your feedback, ${name}!`);
}
```

---

## Common Features

### Embeds

```javascript
const { EmbedBuilder } = require('discord.js');

const embed = new EmbedBuilder()
    .setColor('#0099ff')
    .setTitle('Example Embed')
    .setURL('https://discord.js.org')
    .setAuthor({
        name: 'Bot Name',
        iconURL: 'https://i.imgur.com/AfFp7pu.png',
        url: 'https://discord.js.org'
    })
    .setDescription('This is an example embed')
    .setThumbnail('https://i.imgur.com/AfFp7pu.png')
    .addFields(
        { name: 'Field 1', value: 'Value 1', inline: true },
        { name: 'Field 2', value: 'Value 2', inline: true },
        { name: 'Field 3', value: 'Value 3' }
    )
    .setImage('https://i.imgur.com/AfFp7pu.png')
    .setTimestamp()
    .setFooter({
        text: 'Footer text',
        iconURL: 'https://i.imgur.com/AfFp7pu.png'
    });

await interaction.reply({ embeds: [embed] });
```

### Pagination

```javascript
const { ActionRowBuilder, ButtonBuilder, ButtonStyle } = require('discord.js');

async function createPagination(interaction, pages) {
    let currentPage = 0;

    const row = new ActionRowBuilder()
        .addComponents(
            new ButtonBuilder()
                .setCustomId('prev')
                .setLabel('Previous')
                .setStyle(ButtonStyle.Primary)
                .setDisabled(true),
            new ButtonBuilder()
                .setCustomId('next')
                .setLabel('Next')
                .setStyle(ButtonStyle.Primary)
        );

    const message = await interaction.reply({
        embeds: [pages[currentPage]],
        components: [row],
        fetchReply: true
    });

    const collector = message.createMessageComponentCollector({
        time: 60000 // 1 minute
    });

    collector.on('collect', async i => {
        if (i.customId === 'prev') {
            currentPage--;
        } else if (i.customId === 'next') {
            currentPage++;
        }

        row.components[0].setDisabled(currentPage === 0);
        row.components[1].setDisabled(currentPage === pages.length - 1);

        await i.update({ embeds: [pages[currentPage]], components: [row] });
    });

    collector.on('end', () => {
        row.components.forEach(btn => btn.setDisabled(true));
        message.edit({ components: [row] });
    });
}
```

### Cooldowns

```javascript
// In command execution
const { cooldowns } = interaction.client;

if (!cooldowns.has(command.data.name)) {
    cooldowns.set(command.data.name, new Collection());
}

const now = Date.now();
const timestamps = cooldowns.get(command.data.name);
const cooldownAmount = (command.cooldown || 3) * 1000;

if (timestamps.has(interaction.user.id)) {
    const expirationTime = timestamps.get(interaction.user.id) + cooldownAmount;

    if (now < expirationTime) {
        const timeLeft = (expirationTime - now) / 1000;
        return interaction.reply({
            content: `Please wait ${timeLeft.toFixed(1)} more seconds before using this command again.`,
            ephemeral: true
        });
    }
}

timestamps.set(interaction.user.id, now);
setTimeout(() => timestamps.delete(interaction.user.id), cooldownAmount);
```

### Role Management

```javascript
// Add role to member
const role = interaction.guild.roles.cache.find(r => r.name === 'Member');
await interaction.member.roles.add(role);

// Remove role
await interaction.member.roles.remove(role);

// Check if member has role
if (interaction.member.roles.cache.has(role.id)) {
    // Member has the role
}

// Create role
const newRole = await interaction.guild.roles.create({
    name: 'New Role',
    color: '#ff0000',
    permissions: [PermissionFlagsBits.SendMessages]
});
```

### Channel Management

```javascript
const { ChannelType, PermissionFlagsBits } = require('discord.js');

// Create text channel
const channel = await interaction.guild.channels.create({
    name: 'new-channel',
    type: ChannelType.GuildText,
    topic: 'Channel topic',
    permissionOverwrites: [
        {
            id: interaction.guild.id,
            deny: [PermissionFlagsBits.SendMessages]
        },
        {
            id: interaction.user.id,
            allow: [PermissionFlagsBits.SendMessages]
        }
    ]
});

// Create voice channel
const voiceChannel = await interaction.guild.channels.create({
    name: 'Voice Channel',
    type: ChannelType.GuildVoice,
    userLimit: 10
});

// Create category
const category = await interaction.guild.channels.create({
    name: 'Category',
    type: ChannelType.GuildCategory
});

// Set channel parent
await channel.setParent(category);
```

---

## Advanced Features

### Voice Features

```javascript
const { joinVoiceChannel, createAudioPlayer, createAudioResource } = require('@discordjs/voice');

// Join voice channel
const connection = joinVoiceChannel({
    channelId: voiceChannel.id,
    guildId: guild.id,
    adapterCreator: guild.voiceAdapterCreator
});

// Play audio
const player = createAudioPlayer();
const resource = createAudioResource('./audio.mp3');
player.play(resource);
connection.subscribe(player);

// Leave voice channel
connection.destroy();
```

### Scheduled Tasks

```javascript
const cron = require('node-cron');

// Run task every day at midnight
cron.schedule('0 0 * * *', () => {
    const channel = client.channels.cache.get('CHANNEL_ID');
    channel.send('Daily reminder!');
});

// Run task every 5 minutes
cron.schedule('*/5 * * * *', () => {
    console.log('Running every 5 minutes');
});
```

### Message Reactions

```javascript
// React to message
await message.react('👍');
await message.react('<:custom_emoji:123456789>');

// Reaction collector
const filter = (reaction, user) => {
    return reaction.emoji.name === '👍' && user.id === interaction.user.id;
};

const collector = message.createReactionCollector({ filter, time: 60000 });

collector.on('collect', (reaction, user) => {
    console.log(`${user.tag} reacted with ${reaction.emoji.name}`);
});

collector.on('end', collected => {
    console.log(`Collected ${collected.size} reactions`);
});
```

### Context Menus (User & Message Commands)

```javascript
const { ContextMenuCommandBuilder, ApplicationCommandType } = require('discord.js');

// User context menu
module.exports = {
    data: new ContextMenuCommandBuilder()
        .setName('Get User Info')
        .setType(ApplicationCommandType.User),
    async execute(interaction) {
        const user = interaction.targetUser;
        await interaction.reply(`User: ${user.tag}, ID: ${user.id}`);
    }
};

// Message context menu
module.exports = {
    data: new ContextMenuCommandBuilder()
        .setName('Report Message')
        .setType(ApplicationCommandType.Message),
    async execute(interaction) {
        const message = interaction.targetMessage;
        await interaction.reply({
            content: `Reported message from ${message.author.tag}`,
            ephemeral: true
        });
    }
};
```

### Webhooks

```javascript
// Create webhook
const webhook = await channel.createWebhook({
    name: 'Bot Webhook',
    avatar: 'https://i.imgur.com/AfFp7pu.png'
});

// Send via webhook
await webhook.send({
    content: 'Message from webhook',
    username: 'Custom Name',
    avatarURL: 'https://i.imgur.com/AfFp7pu.png'
});

// Fetch webhooks
const webhooks = await channel.fetchWebhooks();
```

---

## Database Integration

### SQLite (better-sqlite3)

```javascript
const Database = require('better-sqlite3');
const db = new Database('bot.db');

// Create table
db.exec(`
    CREATE TABLE IF NOT EXISTS users (
        user_id TEXT PRIMARY KEY,
        points INTEGER DEFAULT 0,
        level INTEGER DEFAULT 1
    )
`);

// Insert/update
const insert = db.prepare('INSERT OR REPLACE INTO users (user_id, points) VALUES (?, ?)');
insert.run(userId, points);

// Query
const select = db.prepare('SELECT * FROM users WHERE user_id = ?');
const user = select.get(userId);
```

### MongoDB

```javascript
const mongoose = require('mongoose');

// Connect
await mongoose.connect(process.env.MONGODB_URI);

// Schema
const userSchema = new mongoose.Schema({
    userId: { type: String, required: true, unique: true },
    points: { type: Number, default: 0 },
    level: { type: Number, default: 1 }
});

const User = mongoose.model('User', userSchema);

// Usage
const user = await User.findOne({ userId: interaction.user.id });
if (!user) {
    const newUser = new User({ userId: interaction.user.id });
    await newUser.save();
}

user.points += 10;
await user.save();
```

### PostgreSQL (pg)

```javascript
const { Pool } = require('pg');

const pool = new Pool({
    connectionString: process.env.DATABASE_URL
});

// Query
const result = await pool.query(
    'SELECT * FROM users WHERE user_id = $1',
    [userId]
);

// Insert
await pool.query(
    'INSERT INTO users (user_id, points) VALUES ($1, $2) ON CONFLICT (user_id) DO UPDATE SET points = $2',
    [userId, points]
);
```

---

## Deployment

### Environment Variables

```env
# Required
DISCORD_TOKEN=your_bot_token
CLIENT_ID=your_client_id

# Optional
GUILD_ID=test_server_id
DATABASE_URL=postgresql://...
MONGODB_URI=mongodb://...
NODE_ENV=production
```

### Process Managers

#### PM2

```bash
npm install -g pm2

# Start bot
pm2 start src/index.js --name discord-bot

# Monitor
pm2 monit

# Logs
pm2 logs discord-bot

# Restart on file changes (development)
pm2 start src/index.js --name discord-bot --watch
```

#### Docker

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

CMD ["node", "src/index.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  bot:
    build: .
    env_file: .env
    restart: unless-stopped
```

### Hosting Platforms

- **Free Tier**: Railway, Render, Fly.io
- **Traditional VPS**: DigitalOcean, Linode, AWS EC2
- **Serverless**: Not recommended (bots need persistent connections)

---

## Best Practices

### Error Handling

```javascript
// Wrap command execution
try {
    await command.execute(interaction);
} catch (error) {
    console.error(`Error executing ${interaction.commandName}:`, error);

    const errorMessage = {
        content: 'There was an error executing this command!',
        ephemeral: true
    };

    if (interaction.replied || interaction.deferred) {
        await interaction.followUp(errorMessage);
    } else {
        await interaction.reply(errorMessage);
    }
}
```

### Logging

```javascript
// utils/logger.js
const winston = require('winston');

const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
        new winston.transports.Console({
            format: winston.format.simple()
        })
    ]
});

module.exports = logger;
```

### Rate Limiting

```javascript
const rateLimit = new Map();

function checkRateLimit(userId, commandName, limit = 5, window = 60000) {
    const key = `${userId}-${commandName}`;
    const now = Date.now();

    if (!rateLimit.has(key)) {
        rateLimit.set(key, { count: 1, resetAt: now + window });
        return true;
    }

    const data = rateLimit.get(key);

    if (now > data.resetAt) {
        data.count = 1;
        data.resetAt = now + window;
        return true;
    }

    if (data.count >= limit) {
        return false;
    }

    data.count++;
    return true;
}
```

### Security

```javascript
// Never expose sensitive data
// ❌ BAD
await interaction.reply(`Token: ${process.env.DISCORD_TOKEN}`);

// ✅ GOOD
await interaction.reply('Operation completed successfully');

// Validate user input
const userInput = interaction.options.getString('input');
if (userInput.length > 2000) {
    return interaction.reply('Input too long!');
}

// Use ephemeral messages for sensitive info
await interaction.reply({
    content: 'Your secret data...',
    ephemeral: true // Only visible to command user
});
```

### Code Organization

- **Modular structure**: One file per command/event
- **Reusable utilities**: Create helper functions
- **Constants**: Store in config files
- **TypeScript**: Consider for type safety
- **ESLint/Prettier**: Maintain code quality

---

## Code Examples

### Moderation Bot

```javascript
// commands/moderation/ban.js
const { SlashCommandBuilder, PermissionFlagsBits } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('ban')
        .setDescription('Ban a user')
        .addUserOption(option =>
            option
                .setName('target')
                .setDescription('User to ban')
                .setRequired(true)
        )
        .addStringOption(option =>
            option
                .setName('reason')
                .setDescription('Reason for ban')
        )
        .setDefaultMemberPermissions(PermissionFlagsBits.BanMembers),
    async execute(interaction) {
        const target = interaction.options.getUser('target');
        const reason = interaction.options.getString('reason') || 'No reason provided';

        if (target.id === interaction.user.id) {
            return interaction.reply('You cannot ban yourself!');
        }

        try {
            await interaction.guild.members.ban(target, { reason });
            await interaction.reply(`Successfully banned ${target.tag}`);
        } catch (error) {
            console.error(error);
            await interaction.reply('Failed to ban user!');
        }
    }
};
```

### Level/XP System

```javascript
// events/messageCreate.js - XP System
const xpCooldowns = new Map();

module.exports = {
    name: 'messageCreate',
    async execute(message) {
        if (message.author.bot) return;

        const cooldown = xpCooldowns.get(message.author.id);
        if (cooldown && Date.now() < cooldown) return;

        xpCooldowns.set(message.author.id, Date.now() + 60000); // 1 min cooldown

        // Get user from database
        let user = await User.findOne({ userId: message.author.id });
        if (!user) {
            user = new User({ userId: message.author.id });
        }

        // Add random XP (15-25)
        const xpGain = Math.floor(Math.random() * 11) + 15;
        user.xp += xpGain;

        // Level up calculation
        const xpNeeded = user.level * 100;
        if (user.xp >= xpNeeded) {
            user.level++;
            user.xp = 0;
            await message.reply(`🎉 Congratulations! You leveled up to level ${user.level}!`);
        }

        await user.save();
    }
};
```

### Ticket System

```javascript
// commands/utility/ticket.js
const { SlashCommandBuilder, ChannelType, PermissionFlagsBits } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('ticket')
        .setDescription('Create a support ticket'),
    async execute(interaction) {
        await interaction.deferReply({ ephemeral: true });

        const ticketChannel = await interaction.guild.channels.create({
            name: `ticket-${interaction.user.username}`,
            type: ChannelType.GuildText,
            parent: 'CATEGORY_ID', // Replace with your category ID
            permissionOverwrites: [
                {
                    id: interaction.guild.id,
                    deny: [PermissionFlagsBits.ViewChannel]
                },
                {
                    id: interaction.user.id,
                    allow: [
                        PermissionFlagsBits.ViewChannel,
                        PermissionFlagsBits.SendMessages,
                        PermissionFlagsBits.ReadMessageHistory
                    ]
                },
                {
                    id: 'SUPPORT_ROLE_ID', // Replace with support role ID
                    allow: [PermissionFlagsBits.ViewChannel]
                }
            ]
        });

        await ticketChannel.send(
            `${interaction.user}, welcome! Support will be with you shortly.`
        );

        await interaction.editReply(`Ticket created: ${ticketChannel}`);
    }
};
```

### Giveaway System

```javascript
// commands/fun/giveaway.js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('giveaway')
        .setDescription('Start a giveaway')
        .addStringOption(option =>
            option.setName('prize').setDescription('Prize').setRequired(true)
        )
        .addIntegerOption(option =>
            option.setName('duration').setDescription('Duration in minutes').setRequired(true)
        ),
    async execute(interaction) {
        const prize = interaction.options.getString('prize');
        const duration = interaction.options.getInteger('duration');

        const giveawayEmbed = {
            color: 0x0099ff,
            title: '🎉 GIVEAWAY 🎉',
            description: `Prize: **${prize}**\nReact with 🎉 to enter!\nEnds: <t:${Math.floor((Date.now() + duration * 60000) / 1000)}:R>`,
            footer: { text: `Duration: ${duration} minutes` }
        };

        const msg = await interaction.reply({
            embeds: [giveawayEmbed],
            fetchReply: true
        });

        await msg.react('🎉');

        setTimeout(async () => {
            const reactions = await msg.reactions.cache.get('🎉').users.fetch();
            const participants = reactions.filter(u => !u.bot);

            if (participants.size === 0) {
                return msg.reply('No valid entries. Giveaway cancelled.');
            }

            const winner = participants.random();
            await msg.reply(`🎊 Congratulations ${winner}! You won **${prize}**!`);
        }, duration * 60000);
    }
};
```

### Reaction Roles

```javascript
// Setup reaction role message
const { ActionRowBuilder, ButtonBuilder, ButtonStyle } = require('discord.js');

const roles = {
    'role1': 'ROLE_ID_1',
    'role2': 'ROLE_ID_2',
    'role3': 'ROLE_ID_3'
};

const row = new ActionRowBuilder()
    .addComponents(
        new ButtonBuilder()
            .setCustomId('role1')
            .setLabel('Role 1')
            .setStyle(ButtonStyle.Primary),
        new ButtonBuilder()
            .setCustomId('role2')
            .setLabel('Role 2')
            .setStyle(ButtonStyle.Success),
        new ButtonBuilder()
            .setCustomId('role3')
            .setLabel('Role 3')
            .setStyle(ButtonStyle.Danger)
    );

await channel.send({
    content: 'Click a button to get a role!',
    components: [row]
});

// Handle button clicks (in interactionCreate.js)
if (interaction.isButton() && roles[interaction.customId]) {
    const roleId = roles[interaction.customId];
    const role = interaction.guild.roles.cache.get(roleId);

    if (interaction.member.roles.cache.has(roleId)) {
        await interaction.member.roles.remove(role);
        await interaction.reply({
            content: `Removed ${role.name}!`,
            ephemeral: true
        });
    } else {
        await interaction.member.roles.add(role);
        await interaction.reply({
            content: `Added ${role.name}!`,
            ephemeral: true
        });
    }
}
```

### Auto-Moderation

```javascript
// events/messageCreate.js - Auto-mod
const bannedWords = ['badword1', 'badword2'];

module.exports = {
    name: 'messageCreate',
    async execute(message) {
        if (message.author.bot) return;

        const content = message.content.toLowerCase();

        // Check for banned words
        if (bannedWords.some(word => content.includes(word))) {
            await message.delete();
            await message.channel.send(
                `${message.author}, please watch your language!`
            ).then(msg => setTimeout(() => msg.delete(), 5000));
            return;
        }

        // Spam detection
        const messages = await message.channel.messages.fetch({ limit: 5 });
        const userMessages = messages.filter(m => m.author.id === message.author.id);

        if (userMessages.size === 5) {
            const timestamps = userMessages.map(m => m.createdTimestamp);
            const timespan = Math.max(...timestamps) - Math.min(...timestamps);

            if (timespan < 5000) { // 5 messages in 5 seconds
                await message.member.timeout(60000, 'Spam'); // 1 min timeout
                await message.channel.send(
                    `${message.author} has been timed out for spam.`
                );
            }
        }
    }
};
```

---

## Quick Reference Commands

### Search for specific patterns
```bash
# Find all command files
find src/commands -name "*.js"

# Search for specific function usage
grep -r "interaction.reply" src/

# Find all event handlers
grep -r "module.exports.*name:" src/events/

# Search for permission usage
grep -r "PermissionFlagsBits" src/
```

### Useful Discord.js Methods

```javascript
// Client
client.user.setActivity('text', { type: 'PLAYING' })
client.guilds.cache.size
client.users.cache.get('userId')

// Guild
guild.members.fetch()
guild.roles.create()
guild.channels.cache.find(ch => ch.name === 'general')

// Member
member.roles.add(role)
member.timeout(duration, reason)
member.ban({ reason })
member.kick(reason)

// Channel
channel.send()
channel.bulkDelete(amount)
channel.setName(name)
channel.createWebhook()

// Message
message.reply()
message.react(emoji)
message.delete()
message.pin()

// Interaction
interaction.reply()
interaction.deferReply()
interaction.editReply()
interaction.followUp()
interaction.showModal(modal)
```

---

## Additional Resources

- **Discord.js Guide**: https://discordjs.guide/
- **Discord.js Documentation**: https://discord.js.org/
- **Discord Developer Portal**: https://discord.com/developers/docs
- **Discord API Server**: https://discord.gg/discord-api

---

**Ready to build amazing Discord bots! Use this skill whenever you need help with Discord bot development, from basic setup to advanced features.**
