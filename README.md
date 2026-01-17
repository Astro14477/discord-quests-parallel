# discord-quests-parallel (BETA)

Discord quest execution script that can handle **multiple quests at the same time** (parallel), instead of forcing you to **re-paste the code after every quest**.

⚠️ **BETA / UNSTABLE RELEASE**

This is an **experimental rewrite / heavy modification** of an existing Discord quest script, focused on:

- **Parallel execution**
- **Better task handling**
- **Less manual effort**
- **More logging + debugging visibility**

It is **unstable**, **under active testing**, and released publicly to collect bug reports + edge cases.


---

## Features

* Runs **multiple quests in parallel** (instead of sequential blocking)
* Supports these quest task types:

  * `WATCH_VIDEO`
  * `WATCH_VIDEO_ON_MOBILE`
  * `PLAY_ON_DESKTOP`
  * `STREAM_ON_DESKTOP`
  * `PLAY_ACTIVITY`
* Each quest runs **independently** (one failure won’t stall the others)
* Randomized timing + jitter to reduce collisions
* Defensive module extraction + error handling
* Verbose console logging for debugging

---

## Status

**BETA — EXPECT BUGS**

* Race conditions may exist
* Parallel execution is **not rate-limited**
* Store patching may fail if the script crashes or gets force-stopped
* Discord internal changes can break module extraction anytime

If something breaks, **open an issue** with logs.

---

## Usage

1. Open **Discord Desktop**
2. Open **Developer Tools → Console**
3. Paste the script
4. Watch console logs for quest progress

---

## Known Issues / Risks

* No concurrency limit (**all quests may start at once**)
* Heartbeat listeners may stack in edge cases
* Store cleanup may fail if the script is force-stopped
* Mixed quest types running together can behave unpredictably

---

## What This Is NOT

* Not stable
* Not production-ready
* Not guaranteed to work for everyone
* Not protected against Discord updates

---

## Reporting Bugs

When opening an issue, include:

* Quest type(s)
* Desktop or Web
* Full console logs
* Whether multiple quests were running

Low-effort reports without logs may be ignored.

---

## Credits

This is a **heavily modified / parallelized version** of **aamiaa’s original quest script**:
[https://gist.github.com/aamiaa/204cd9d42013ded9faf646fae7f89fbb](https://gist.github.com/aamiaa/204cd9d42013ded9faf646fae7f89fbb)

---

## Disclaimer

Provided **as-is** for educational + experimental purposes.
Use at your own risk.

---
