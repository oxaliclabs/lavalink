<div align="center">

# Self Managed Lavalink

A ready to run [Lavalink](https://github.com/lavalink-devs/Lavalink) v4 node for Discord music bots.<br>
YouTube, Spotify, SoundCloud and JioSaavn out of the box.

![Java](https://img.shields.io/badge/Java-21%2B-e76f00?style=flat-square)
![Lavalink](https://img.shields.io/badge/Lavalink-4.2.2-5865f2?style=flat-square)
![Config](https://img.shields.io/badge/config-application.yml-2b2d31?style=flat-square)

[Setup](#setup) · [Configuration](#configuration) · [Connect your bot](#connect-your-bot) · [Troubleshooting](#troubleshooting) · [Support](https://discord.gg/CQeUx93rft/)

</div>

---

## What you get

| Area | Details |
|---|---|
| **Playback** | YouTube (via the YouTube plugin), SoundCloud, Bandcamp, Twitch, Vimeo, Niconico, direct HTTP streams |
| **Spotify** | Tracks, albums and playlists are read from Spotify and played from YouTube / SoundCloud |
| **JioSaavn** | Playback through an API you host on Vercel |
| **Extras** | SponsorBlock, LavaSearch, LavaLyrics, and the DuncteBot source pack (TTS, Mixcloud, Reddit, TikTok and others) |
| **Filters** | Volume, equalizer, karaoke, timescale, tremolo, vibrato, distortion, rotation, channel mix, low pass |

> **Heads up:** this project is for personal and educational use. Some of the sources here are unofficial and can break at any time, or go against a service's terms. Use them at your own risk.

---

## Requirements

| | Version | Notes |
|---|---|---|
| **Java** | 21 or newer | Temurin 21 recommended. The JioSaavn plugin is built for Java 21 and crashes on 17 with `UnsupportedClassVersionError`. |
| **Lavalink** | 4.2.2 | `Lavalink.jar` from the [releases page](https://github.com/lavalink-devs/Lavalink/releases). |
| **Network** | outbound HTTPS | Plugins are downloaded on the first start. |

Check what you have:

```bash
java -version
```

<details>
<summary><b>Installing Java 21</b></summary>

<br>

**Windows**

```powershell
winget install EclipseAdoptium.Temurin.21.JDK
```

Close the terminal and open a new one afterwards, otherwise the old Java stays on your PATH.

**Ubuntu 24.04 / Debian**

```bash
sudo apt update
sudo apt install openjdk-21-jre-headless
```

**macOS**

```bash
brew install --cask temurin@21
```

</details>

---

## Setup

Lavalink ships as a finished jar. There is nothing to compile and no `npm install` or `build` step.

**1. Get the files**

```bash
mkdir lavalink && cd lavalink
wget https://github.com/lavalink-devs/Lavalink/releases/download/4.2.2/Lavalink.jar
```

Put your `application.yml` next to the jar.

**2. Fill in the config**

Open `application.yml` and replace the placeholders. The [Configuration](#configuration) section lists every value you need to touch.

**3. Add the JioSaavn plugin jar**

The JioSaavn plugin is not pulled from Maven like the others, so it is added by hand. Download `jiosaavn-plugin-1.0.6.jar` from the [plugin releases](https://github.com/c0ders-io/jiosaavn-plugin/releases) and drop it into `plugins/`. Create the folder if it does not exist yet.

**4. Start it**

```bash
java -jar Lavalink.jar
```

Run this from the folder that holds both `Lavalink.jar` and `application.yml`. The first start downloads the remaining plugins into `./plugins`, so give it a minute. You are good once you see:

```
Lavalink is ready to accept connections.
```

**5. Check that it works**

```bash
curl -H "Authorization: YOUR_PASSWORD" http://localhost:9999/version
curl -H "Authorization: YOUR_PASSWORD" http://localhost:9999/v4/info
```

The first returns the Lavalink version. The second returns JSON that includes every loaded plugin, which is the quickest way to confirm that all of them started.

### Folder layout

```
lavalink/
├── Lavalink.jar
├── application.yml
├── logs/
└── plugins/
    ├── youtube-plugin-<version>.jar
    ├── lavasrc-plugin-<version>.jar
    ├── lavasearch-plugin-<version>.jar
    ├── lavalyrics-plugin-<version>.jar
    ├── sponsorblock-plugin-<version>.jar
    ├── skybot-lavalink-plugin-<version>.jar
    └── jiosaavn-plugin-1.0.6.jar
```

Only plugin jars belong in `plugins/`. `Lavalink.jar` stays in the root.

---

## Running it in the background

<details open>
<summary><b>pm2</b></summary>

<br>

```bash
pm2 start "java -jar Lavalink.jar" --name lavalink
pm2 save
```

Start it from the Lavalink folder so it finds `application.yml`.

</details>

<details>
<summary><b>systemd (Linux)</b></summary>

<br>

Create `/etc/systemd/system/lavalink.service`:

```ini
[Unit]
Description=Lavalink
After=network.target

[Service]
User=lavalink
WorkingDirectory=/opt/lavalink
ExecStart=/usr/bin/java -Xmx1G -jar Lavalink.jar
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now lavalink
journalctl -u lavalink -f
```

Adjust `-Xmx` to what your server can spare. 1 GB is enough for a small bot.

</details>

---

## Configuration

Everything lives in `application.yml`. These are the values you have to change:

| Key | Placeholder in the file | What to use |
|---|---|---|
| `server.port` | `9999` | Any free port. Your bot connects here. |
| `lavalink.server.password` | `youshallnotpass` | A long random string. Generate one with `openssl rand -base64 24`. |
| `plugins.youtube.oauth.refreshToken` | `youtube_refresh_token` | Printed in the log on first start. See [YouTube](#youtube). |
| `plugins.lavasrc.spotify.clientId` | `spotify_client_id` | From the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard). |
| `plugins.lavasrc.spotify.clientSecret` | `spotify_client_secret` | Same place as the client ID. |
| `plugins.jiosaavn.apiURL` | a sample Vercel URL | Your own deployment. See [JioSaavn](#jiosaavn). |

The other placeholders in the file (Apple Music, Deezer, Yandex, VK Music, Tidal, Qobuz) belong to sources that are switched off with `false`. Leave them alone until you have real credentials for them.

> **Keep `application.yml` out of Git.** It holds your password and API keys. Add it to `.gitignore` before the first commit.

### YouTube

The built-in Lavaplayer YouTube source is turned off (`lavalink.server.sources.youtube: false`). All YouTube traffic goes through the YouTube plugin instead.

```yaml
plugins:
  youtube:
    enabled: true
    remoteCipher:
      url: "https://cipher.kikkia.dev/"
    oauth:
      enabled: true
      refreshToken: "youtube_refresh_token"
    clients:
      - TV
      - ANDROID_VR
      - ANDROID_MUSIC
      - IOS
      - MWEB
      - WEB
      - MUSIC
      - WEBEMBEDDED
```

Two things make this work reliably:

- **Remote cipher.** Signature decoding is handed to a hosted [yt-cipher](https://github.com/kikkia/yt-cipher) server. Without it, tracks fail with `Must find sig function` whenever YouTube changes its player.
- **OAuth.** Some IPs, especially datacenter ones, get `This video requires login`. OAuth with a throwaway Google account fixes that.

**Getting the refresh token**

1. Make a new Google account just for this. **Do not use your main account.**
2. Set `oauth.enabled: true` and leave `refreshToken` empty on the first run.
3. Start Lavalink and watch the log. It prints a `google.com/device` link and a code.
4. Open the link, enter the code, and sign in with the throwaway account.
5. The log then prints a refresh token. Copy it into `refreshToken` and restart.

The YouTube plugin is pinned to a specific snapshot commit (`2be8e54...`) because it carries the TV client fix that has not reached a stable release. See [Plugins](#plugins).

### Spotify

Spotify only supplies metadata. Audio is found elsewhere, in this order, until something matches:

1. YouTube search by ISRC
2. YouTube Music search
3. YouTube search
4. SoundCloud search

That order is the `lavasrc.providers` list, so you can reorder it. `playlistLoadLimit` and `albumLoadLimit` count pages, not tracks, so a low number keeps huge playlists from flooding your queue.

### JioSaavn

The plugin does not talk to JioSaavn directly. It asks an API that you host yourself.

1. Fork `appujet/jiosaavn-plugin-api` on GitHub.
2. Import the fork into [Vercel](https://vercel.com).
3. Use these build settings:

| Setting | Value |
|---|---|
| Framework Preset | **Other** |
| Build Command | *(empty)* |
| Output Directory | *(empty)* |
| Install Command | `npm install` |
| Root Directory | `./` |
| Function Region | Mumbai, India (South) `bom1` |

4. When the deploy finishes, open `https://your-project.vercel.app/api`. You should get a JSON message back.
5. Put that URL in the config:

```yaml
plugins:
  jiosaavn:
    apiURL: "https://your-project.vercel.app/api"
    playlistTrackLimit: 50
    recommendationsTrackLimit: 10
```

If the preset stays on `Node`, the build fails with `No entrypoint found`.

### Sources and lyrics

| Source | State | Notes |
|---|---|---|
| YouTube | on | Through the YouTube plugin |
| Spotify | on | Mirrored to YouTube / SoundCloud |
| JioSaavn | on | Needs your Vercel API |
| SoundCloud, Bandcamp, Twitch, Vimeo, Niconico, HTTP | on | Built into Lavalink |
| DuncteBot pack | on | TTS, GetYarn, Clyp.it, Reddit, OCRemix, TikTok, Mixcloud, Soundgasm, PixelDrain, Pornhub |
| Apple Music, Deezer, Yandex, VK Music, Tidal, Qobuz, yt-dlp | off | Need credentials or extra setup |
| Local files | off | `lavalink.server.sources.local: false` |

Lyrics are installed but idle. Every flag under `lavasrc.lyrics-sources` is `false`, so no lyrics are returned. Turn on the ones you want, keeping in mind that some providers need extra credentials.

If your bot is public, set `pornhub: false` under `plugins.dunctebot.sources`.

### Search prefixes

When your bot searches, the prefix picks the source:

| Prefix | Source |
|---|---|
| `ytsearch:` | YouTube |
| `ytmsearch:` | YouTube Music |
| `scsearch:` | SoundCloud |
| `spsearch:` | Spotify |

---

## Connect your bot

| Setting | Value |
|---|---|
| Host | `localhost`, or the server's IP |
| Port | `server.port` from `application.yml` (`9999` by default) |
| Password | `lavalink.server.password` from `application.yml` |
| Secure | `false` |

Example with [Shoukaku](https://github.com/shipgirlproject/Shoukaku):

```js
const nodes = [{
  name: 'main',
  url: 'localhost:9999',
  auth: 'your-password',
  secure: false,
}];
```

Wait for the node's `ready` / `connected` event before handling play commands. If a search runs before the node is up, you get `No Lavalink Node was provided`.

If the bot runs on a different machine, open the port in your firewall and keep the password strong. Lavalink listens on `0.0.0.0` in this config.

---

## Plugins

| Plugin | Version | Purpose |
|---|---|---|
| youtube-plugin | snapshot `2be8e542d3f6f178e048dca565892684c2e40177` | YouTube playback, includes the TV client fix |
| lavasrc-plugin | 4.7.3 | Spotify and other mirrored sources |
| lavasearch-plugin | 1.0.0 | Search support |
| lavalyrics-plugin | 1.1.0 | Lyrics support |
| sponsorblock-plugin | 3.0.1 | SponsorBlock segments |
| skybot-lavalink-plugin (DuncteBot) | 1.7.0 | Extra sources |
| jiosaavn-plugin | 1.0.6 | JioSaavn through your hosted API |

### Why LavaSrc stays on 4.7.x

LavaSrc 4.8.0 and newer ship their own JioSaavn code. It clashes with the standalone `jiosaavn-plugin` and Lavalink refuses to boot. Until you drop one of the two, stay on 4.7.x.

### Updating a plugin

1. Change the version in the `lavalink.plugins` list.
2. **Delete the old jar** for that plugin from `plugins/`. Two versions of the same plugin side by side cause startup errors.
3. Restart Lavalink. The new jar downloads on boot.

---

## Troubleshooting

| Problem | Cause and fix |
|---|---|
| `UnsupportedClassVersionError ... class file version 65.0` | A plugin needs Java 21. Install it and start Lavalink from a **new** terminal. |
| `bean name 'jioSaavnConfig' conflicts` | LavaSrc 4.8.x and `jiosaavn-plugin` can't run together. Use LavaSrc 4.7.x. |
| `Must find sig function` | The local cipher is out of date. Make sure `remoteCipher` is set in the YouTube config. |
| `This video requires login` | YouTube is restricting your IP or session. Enable OAuth with a throwaway account. |
| `The page needs to be reloaded` | Bug in the released YouTube plugin 1.18.2. Use the pinned snapshot from this repo. |
| JioSaavn `NullPointerException ... getPlaybackUrl` | The API returned no stream link. Deploy your own API and set `apiURL`. |
| `route not found` from the JioSaavn API | `apiURL` points at a different project (saavn.dev, for example), not `jiosaavn-plugin-api`. |
| `No Lavalink Node was provided` | The bot searched before the node connected. Wait for the connected event. |
| Bot gets `401` or can't authenticate | The password in the bot does not match `lavalink.server.password`. |
| `Connection refused` | Lavalink isn't running, the port is wrong, or a firewall is blocking it. |
| `Port 9999 was already in use` | Another process has the port. Stop it or change `server.port`. |
| Parse error on start | Almost always YAML indentation. Use spaces, never tabs. |
| A plugin is missing from `/v4/info` | Check `logs/` for the download error, then confirm the jar is in `plugins/` and not nested in a subfolder. |

---

## Security notes

- Change `youshallnotpass` before this node touches the internet.
- Request logging is on and includes headers, which means the `Authorization` password can end up in `logs/`. Switch `includeHeaders` and `includePayload` off, or lock down the log folder.
- Use a burner Google account for YouTube OAuth. Accounts used this way can get flagged.
- Don't publish `application.yml`, your refresh token, or your Spotify secret.

---

## Credits

This setup stands on other people's work.

- **[Lavalink](https://github.com/lavalink-devs/Lavalink)** by the lavalink-devs team, the audio server
- **[youtube-source](https://github.com/lavalink-devs/youtube-source)** by lavalink-devs and contributors, the YouTube plugin
- **[yt-cipher](https://github.com/kikkia/yt-cipher)** by kikkia, the remote cipher server
- **[LavaSrc](https://github.com/topi314/LavaSrc)**, **LavaSearch**, **LavaLyrics** and **SponsorBlock** by [topi314](https://github.com/topi314) and contributors
- **[skybot-lavalink-plugin](https://github.com/DuncteBot/skybot-lavalink-plugin)** by DuncteBot
- **[jiosaavn-plugin](https://github.com/c0ders-io/jiosaavn-plugin)** and **jiosaavn-plugin-api** by [appujet](https://github.com/appujet) (BotXLab) and contributors
- **[Lavaplayer](https://github.com/lavalink-devs/lavaplayer)**, the audio library under Lavalink

Setup and configuration maintained by [Oxalic Labs](https://github.com/oxaliclabs).
[Website](https://oxalic.xyz/) · [Support server](https://discord.gg/CQeUx93rft/)

---

## Disclaimer

This project is not affiliated with or endorsed by YouTube, Spotify, JioSaavn, Apple, Deezer or any other service mentioned here. All trademarks belong to their owners. You are responsible for how you use this software and for following the terms of every service you connect to.