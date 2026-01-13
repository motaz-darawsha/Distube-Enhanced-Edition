# DisTube Enhanced Edition

<div align="center">
  <p>
    <a href="https://github.com/motaz-darawsha/Distube-Enhanced-Edition">
      <img src="https://img.shields.io/github/stars/motaz-darawsha/Distube-Enhanced-Edition?style=social" alt="GitHub stars">
    </a>
    <a href="https://github.com/motaz-darawsha/Distube-Enhanced-Edition">
      <img src="https://img.shields.io/github/forks/motaz-darawsha/Distube-Enhanced-Edition?style=social" alt="GitHub forks">
    </a>
    <a href="https://github.com/motaz-darawsha/Distube-Enhanced-Edition">
      <img src="https://img.shields.io/github/license/motaz-darawsha/Distube-Enhanced-Edition" alt="License">
    </a>
  </p>
  <h3>🎵 Enhanced Discord Music Library</h3>
  <p><em>Forked from DisTube with Auto Refresh for uninterrupted streaming</em></p>
</div>

## 📦 Installation

```bash
npm install https://github.com/motaz-darawsha/Distube-Enhanced-Edition.git
npm install discord.js @discordjs/voice @discordjs/opus
```

## 🚀 Quick Start

```javascript
const { DisTube } = require('distube');
const { Client, GatewayIntentBits } = require('discord.js');

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
  ],
});

const distube = new DisTube(client, {
  autoRefresh: true, // 🆕 Auto refresh feature
});

distube.on('playSong', (queue, song) =>
  queue.textChannel.send(`🎵 Playing: **${song.name}**`)
);

client.on('messageCreate', message => {
  if (message.content.startsWith('!play')) {
    distube.play(message.member.voice.channel, message.content.slice(6));
  }
});

client.login('YOUR_TOKEN');
```

## ⚙️ Configuration Options

### Core Options [1](#19-0) 

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `plugins` | `DisTubePlugin[]` | `[]` | Music source plugins (YouTube, Spotify, etc.) |
| `emitNewSongOnly` | `boolean` | `false` | Only emit `playSong` for new songs (not loops) |
| `savePreviousSongs` | `boolean` | `true` | Enable song history and `previous()` method |
| `customFilters` | `Filters` | `{}` | Custom FFmpeg audio filters |
| `nsfw` | `boolean` | `false` | Allow age-restricted content |
| `emitAddSongWhenCreatingQueue` | `boolean` | `true` | Emit `addSong` when creating new queue |
| `emitAddListWhenCreatingQueue` | `boolean` | `true` | Emit `addList` when creating new queue |
| `joinNewVoiceChannel` | `boolean` | `true` | Auto-join new voice channels |
| `autoRefresh` | `boolean` | `false` | 🆕 Auto refresh stream URLs |

### FFmpeg Options [2](#19-1) 

```javascript
ffmpeg: {
  path?: string,        // FFmpeg executable path (default: "ffmpeg")
  args?: {
    global?: object,    // Global FFmpeg arguments
    input?: object,     // Input-specific arguments  
    output?: object     // Output-specific arguments
  }
}
```

## 🔄 Auto Refresh Feature

### Overview
Automatically refreshes stream URLs before they expire during song loops, preventing interruptions.

### How It Works [3](#19-2) 
1. Extracts `expire` parameter from stream URLs
2. Tracks expiry time with song metadata  
3. Refreshes URLs 2 minutes before expiry
4. Works only with `RepeatMode.SONG`

### Example Usage
```javascript
const distube = new DisTube(client, {
  autoRefresh: true // Enable auto refresh
});

// Play and loop a song
await distube.play(voiceChannel, "https://youtube.com/watch?v=example");
const queue = distube.getQueue(guildId);
queue.setRepeatMode(RepeatMode.SONG); // Auto refresh activates
```

### Source Compatibility
| Source | Support | Notes |
|--------|---------|-------|
| YouTube | ✅ Full | Extracts expire from URLs |
| SoundCloud | ❌ None | No expire parameter |
| Spotify | ❌ None | Uses alternative songs |

## 📝 Complete Example

```javascript
const { DisTube, RepeatMode } = require('distube');
const { Client, GatewayIntentBits } = require('discord.js');
// import the plugins

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
  ],
});

const distube = new DisTube(client, {
  plugins: [
    new YouTubePlugin(),
    new SpotifyPlugin({ clientId: 'ID', clientSecret: 'SECRET' })
  ],
  autoRefresh: true,
  emitNewSongOnly: true,
  customFilters: {
    bassboost: "bass=g=20",
    nightcore: "asetrate=48000*1.25"
  },
  ffmpeg: {
    args: {
      global: { loglevel: "error" },
      output: { acodec: "libopus" }
    }
  }
});

// Events
distube
  .on('playSong', (queue, song) => 
    queue.textChannel.send(`🎵 **${song.name}** - \`${song.formattedDuration}\``)
  )
  .on('addSong', (queue, song) => 
    queue.textChannel.send(`✅ Added **${song.name}** to queue`)
  )
  .on('error', (channel, error) => {
    console.error(error);
    channel.send(`❌ Error: ${error.message}`);
  });

// Commands
client.on('messageCreate', async message => {
  if (message.author.bot) return;
  
  const args = message.content.trim().split(/ +/);
  const command = args.shift().toLowerCase();
  const queue = distube.getQueue(message.guildId);
  
  switch (command) {
    case '!play':
      if (!message.member.voice.channel) 
        return message.reply('❌ Join a voice channel!');
      await distube.play(message.member.voice.channel, args.join(' '));
      break;
      
    case '!loop':
      if (!queue) return message.reply('❌ No music playing!');
      queue.setRepeatMode(RepeatMode.SONG);
      message.reply('🔁 Looping current song');
      break;
      
    case '!skip':
      if (!queue) return message.reply('❌ No music playing!');
      await queue.skip();
      message.reply('⏭️ Skipped');
      break;
  }
});

client.login('YOUR_TOKEN');
```

## 📋 Requirements

- **Node.js**: 22.12.0+
- **discord.js**: v14
- **FFmpeg**: System-wide installation required

## Notes

- Forked from original DisTube [4](#19-3) 
- Auto Refresh disabled by default
- Works only with `RepeatMode.SONG` [5](#19-4) 
- Shows warnings for unsupported sources [6](#19-5) 
