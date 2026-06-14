# Notematic Player

<div align="center">

[![Modrinth Downloads](https://img.shields.io/modrinth/dt/notematic-player?style=for-the-badge&logo=modrinth&color=24b47e)](https://modrinth.com/plugin/notematic-player)
[![Minecraft Version Support](https://img.shields.io/badge/Minecraft-1.20.6%20--%2026.1.x-blue?style=for-the-badge&logo=minecraft)](https://www.minecraft.net/)
[![Server Software](https://img.shields.io/badge/Platform-Paper%20%7C%20Purpur%20%7C%20Spigot-gold?style=for-the-badge)](https://papermc.io)
[![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)](#)

A high-performance Minecraft Java plugin (Paper/Spigot 1.20.6 / 1.21 - 26.1.x) designed as the companion playback engine for **Notematic Studio**. It plays custom note block music sequences exported from the studio in `.mcfunction` or `.json` formats, extending standard note blocks to play any in-game `/playsound` audio (including custom resource pack sounds) directly.

</div>

---

> [!NOTE]
> **Notematic Player** operates 100% server-side. It does not require any client-side mods or resource packs from your players.

---

## Key Features

* 🎵 **Playsound Mapping:** Play native note blocks alongside any complex Minecraft `/playsound` effect (e.g., custom sound effects, instrument sounds).
* ⏱️ **Zero-Lag Ticking:** Uses an advanced virtual scheduler running asynchronously to ensure notes play back in perfect synchronization, independent of TPS fluctuations.
* 📍 **Positional 3D Playback:** Play sounds either globally for players (like a direct voice/sound) or positionally at physical coordinate locations with custom radii.
* 🔄 **Persistent Location Loops:** Location-based looping playbacks are stored in a database (`persistent_playbacks.yml`) and automatically resume at the exact tick progress after server restarts or crashes.
* 👥 **Offline Resync:** If a player disconnects, their active playback ticks virtually in the background and resynchronizes seamlessly the moment they rejoin.
* 🎚️ **Dynamic Volume Stacking:** Supports separate multipliers for global songs, individual playback instances, and personal player comfort volume.

---

## Commands & Permissions

### Command Quick Reference

| Command | Description | Permission Node | Default |
| :--- | :--- | :--- | :--- |
| `/notematic` | Displays plugin status, version, and help menu. | `notematic.use` | Everyone |
| `/notematic play ...` | Plays music for a player, everyone, or at a world location. | `notematic.use` | Everyone |
| `/notematic stop ...` | Stops active playbacks by ID, song, player, or location. | `notematic.use` | Everyone |
| `/notematic seek ...` | Seeks forward, backward, or to a specific time. | `notematic.use` | Everyone |
| `/notematic pause ...` | Pauses active playbacks. | `notematic.use` | Everyone |
| `/notematic resume ...` | Resumes paused playbacks. | `notematic.use` | Everyone |
| `/notematic volume <val>` | Sets your personal volume level (0-100%). | `notematic.use` | Everyone |
| `/notematic volume #ID <val>` | Adjusts a specific active playback instance's volume. | `notematic.use` | Everyone |
| `/notematic volume song ...` | Adjusts a specific song's global base volume (Admin). | `notematic.admin` | OP |
| `/notematic active` | Lists active playbacks, progress, and statuses. | `notematic.use` | Everyone |
| `/notematic list` | Lists all loaded songs in the library. | `notematic.use` | Everyone |
| `/notematic commands ...` | Globally enables or disables player command usage (Admin). | `notematic.admin` | OP |
| `/notematic reload` | Stops all active playbacks and reloads the song folder. | `notematic.admin` | OP |

---

### Command Guide

#### 1. Play Command: `/notematic play <song> [target] [loop]`
Starts playing a song. Playback can target yourself, other players, all online players, or a physical 3D location.
*   **Examples:**
    *   `/notematic play track_one` (Plays for yourself)
    *   `/notematic play track_one loop` (Plays looping for yourself)
    *   `/notematic play track_one @a` (Plays for everyone, *requires Admin*)
    *   `/notematic play track_one at ~ ~ ~ 16 80% loop` (Plays a loop positioned at your feet with a 16-block radius at 80% volume)

#### 2. Stop Command: `/notematic stop [#ID | song | player] [at <x> <y> <z>]`
Stops active playbacks. It dynamically searches by ID first, then song name, then target player.
*   **Examples:**
    *   `/notematic stop` (Stops your own playbacks)
    *   `/notematic stop #12` (Stops playback instance #12)
    *   `/notematic stop track_one` (Stops all active instances of "track_one")

#### 3. Seek Command: `/notematic seek <#ID | song | player> <value>`
Seeks active playbacks forward (`+`), backward (`-`), or to an absolute time.
*   **Examples:**
    *   `/notematic seek #5 +10s` (Seeks forward 10 seconds)
    *   `/notematic seek #5 30s` (Seeks to exactly 30 seconds)
    *   `/notematic seek #5 -150t` (Seeks backward 150 server ticks)

#### 4. Volume Management
*   **Personal Comfort:** `/notematic volume 80%` (UUID-bound, persisted in `player_volumes.yml`).
*   **Temporary Instance:** `/notematic volume #3 50%` (Resets when playback stops).
*   **Global Song Base:** `/notematic volume song track_one 60%` (Persisted in `volumes.yml`, *requires Admin*).

> [!TIP]
> **Volume Formula:** The final note block volume a player hears is calculated as:
> $$\text{Final Volume} = \text{Base Note Volume} \times \text{Song Multiplier} \times \text{Personal Comfort Volume}$$

---

## Music File Formats & Organization

Place your music files (`.mcfunction` or `.json`) in `plugins/NotematicPlayer/songs/`.

*   **Public Songs (`/songs/` or `/songs/public/`):**
    *   Accessible by all players.
*   **Private Songs (`/songs/private/`):**
    *   Only accessible by OPs or players with `notematic.admin` permission.

### Native JSON Structure
```json
{
  "tempo": 1.0,
  "notes": [
    {
      "instrument": "harp",
      "note": 1.4142,
      "volume": 0.8,
      "when": 0
    },
    {
      "instrument": "minecraft:entity.cow.ambient",
      "note": 1.0,
      "volume": 0.6,
      "when": 5
    }
  ]
}
```

---

## Developer API

Other plugins can interact with `NotematicPlayer` directly.

### Accessing the API
```java
import me.cayde26.notematicPlayer.NotematicPlayer;
import org.bukkit.Bukkit;

NotematicPlayer api = (NotematicPlayer) Bukkit.getPluginManager().getPlugin("NotematicPlayer");
if (api != null) {
    api.playSong(player, "chromatic_scale");
}
```

### Key API Methods

| Signature | Description |
| :--- | :--- |
| `boolean playSong(Player player, String songName)` | Plays song for a player. |
| `boolean playSong(Player player, String song, boolean chat, boolean positional)` | Plays song with custom chat & position preferences. |
| `void stopSong(Player player)` | Stops all active playbacks for a player. |
| `void pauseSong(Player player)` | Pauses all active playbacks for a player. |
| `void resumeSong(Player player)` | Resumes all paused playbacks for a player. |
| `double getPlayerVolume(Player player)` | Retrieves player's personal volume multiplier. |
| `void setPlayerVolume(Player player, double vol)` | Updates player's personal volume multiplier. |
