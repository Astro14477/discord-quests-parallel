# discord-quests-parallel (BETA)

Discord parallel quest execution script with experimental task handling and beta testing support.


⚠️ **BETA / UNSTABLE RELEASE**

This project is an **experimental rewrite** of an existing Discord quest script, focused on **parallel execution**, **better task handling**, and **improved performance**.

It is currently **unstable**, **under active testing**, and released **publicly to gather bug reports and edge cases**.

---

## Features

- Parallel quest execution (instead of sequential blocking)
- Supports multiple quest task types:
  - WATCH_VIDEO
  - WATCH_VIDEO_ON_MOBILE
  - PLAY_ON_DESKTOP
  - STREAM_ON_DESKTOP
  - PLAY_ACTIVITY
- Independent quest handling (one failure won’t stall others)
- Randomized timing & jitter to reduce collisions
- Defensive module extraction & error handling
- Verbose logging for debugging

---

## Status

**BETA — EXPECT BUGS**

- Race conditions may exist
- Parallel execution is not rate-limited yet
- Store patching may fail on unexpected crashes
- Behavior may change with Discord updates

If something breaks, **open an issue**.

---

## Usage

1. Open Discord **Desktop**
2. Open **Developer Tools → Console**
3. Paste the script
4. Watch logs for progress

> Some quest types require the **Discord Desktop app**.

---

## Known Issues

- No concurrency limit (all quests may start at once)
- Heartbeat listeners may stack in edge cases
- Store cleanup may fail if the script is force-stopped
- Experimental behavior on mixed quest types

---

## What This Is NOT

- Not a stable release
- Not production-ready
- Not guaranteed to work for everyone
- Not actively maintained for breaking Discord changes (yet)

---

## Reporting Bugs

When opening an issue, include:
- Quest type(s)
- Desktop or Web
- Console logs
- Whether multiple quests were running

Low-effort reports without logs may be ignored.

---

## Credits

Based on **aamiaa’s original quest script**,  
**heavily modified, parallelized, and optimized.**

---

## Disclaimer

This project is provided **as-is** for educational and experimental purposes.
Use at your own risk.
