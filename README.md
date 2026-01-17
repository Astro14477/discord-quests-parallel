# discord-quests-parallel (BETA)

Discord parallel quest execution script with experimental task handling and beta testing support.


⚠️ **BETA / UNSTABLE RELEASE**

This project is an **experimental rewrite** of an existing Discord quest script, focused on **parallel execution**, **better task handling**, and **improved performance**.

It is currently **unstable**, **under active testing**, and released **publicly to gather bug reports and edge cases**.

---

delete window.$;

let wpRequire;
try {
    wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
    webpackChunkdiscord_app.pop();
} catch (e) {
    console.error("Webpack chunk push failed. Make sure you're in the Discord web/desktop app and logged in.");
    throw e;
}

let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getStreamerActiveStreamMetadata)?.exports.Z;
let RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames)?.exports.ZP;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getQuest)?.exports.Z;
let ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getAllThreadsForParent)?.exports.Z;
let GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getSFWDefaultChannel)?.exports.ZP;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.flushWaitQueue)?.exports.Z;
let api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get)?.exports.tn;

if (!api || !QuestsStore || !FluxDispatcher) {
    console.error("Failed to find required Discord modules. Try refreshing the page.");
    throw new Error("Module extraction failed");
}

const supportedTasks = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"];

let quests = [...QuestsStore.quests.values()]
    .filter(x => 
        x.userStatus?.enrolledAt && 
        !x.userStatus?.completedAt && 
        new Date(x.config.expiresAt).getTime() > Date.now() &&
        supportedTasks.some(y => (x.config.taskConfig ?? x.config.taskConfigV2)?.tasks?.[y])
    );

let isApp = typeof DiscordNative !== "undefined";

if (quests.length === 0) {
    console.log("You don't have any uncompleted supported quests!");
} else {
    console.warn(`Launching ALL ${quests.length} quests IN PARALLEL.`);
    console.warn("VIDEO quests = relatively safe. GAME/STREAM/ACTIVITY quests in parallel = HIGH BAN/REVOKE RISK in 2026.");
    console.warn("If you see errors/429s or weird progress, CLOSE THIS TAB and stop.");

    async function startQuest(quest) {
        // Small random delay to stagger starts slightly (helps a tiny bit vs detection)
        await new Promise(r => setTimeout(r, 1000 + Math.random() * 5000));

        const pid = Math.floor(Math.random() * 30000) + 1000;
        const applicationId = quest.config.application.id;
        const applicationName = quest.config.application.name || "Unknown App";
        const questName = quest.config.messages?.questName || "Unnamed Quest";
        const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2;
        const taskName = supportedTasks.find(x => taskConfig.tasks?.[x] != null);
        const secondsNeeded = taskConfig.tasks[taskName].target;
        let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0;

        console.log(`Starting quest: ${questName} (${taskName}) — ${secondsDone}/${secondsNeeded} s`);

        if (taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
            const maxFuture = 10;
            const speed = 7;
            const interval = 1;
            const enrolledAt = new Date(quest.userStatus.enrolledAt).getTime();
            let completed = false;

            async function updateVideo() {
                while (true) {
                    const maxAllowed = Math.floor((Date.now() - enrolledAt) / 1000) + maxFuture;
                    const diff = maxAllowed - secondsDone;
                    const timestamp = secondsDone + speed;

                    if (diff >= speed) {
                        try {
                            const res = await api.post({
                                url: `/quests/${quest.id}/video-progress`,
                                body: { timestamp: Math.min(secondsNeeded, timestamp + Math.random() * 2 - 1) }
                            });
                            completed = res.body.completed_at != null;
                            secondsDone = Math.min(secondsNeeded, timestamp);
                            console.log(`[${questName}] Video progress → ${secondsDone}/${secondsNeeded}`);
                        } catch (err) {
                            console.error(`[${questName}] Video post failed:`, err);
                            break;
                        }
                    }

                    if (secondsDone >= secondsNeeded) break;

                    await new Promise(r => setTimeout(r, interval * 1000));
                }

                if (!completed) {
                    try {
                        await api.post({ url: `/quests/${quest.id}/video-progress`, body: { timestamp: secondsNeeded } });
                    } catch {}
                }
                console.log(`[${questName}] Video quest completed!`);
            }

            updateVideo();
        } 
        else if (taskName === "PLAY_ON_DESKTOP") {
            if (!isApp) {
                console.warn(`[${questName}] PLAY_ON_DESKTOP requires Discord Desktop app — skipping.`);
                return;
            }

            try {
                const res = await api.get({ url: `/applications/public?application_ids=${applicationId}` });
                const appData = res.body[0];
                if (!appData?.executables) throw new Error("No executable data");

                const exeName = appData.executables.find(x => x.os === "win32")?.name?.replace(">", "") || "unknown.exe";
                const fakeGame = {
                    cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
                    exeName,
                    exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
                    hidden: false,
                    isLauncher: false,
                    id: applicationId,
                    name: appData.name,
                    pid: pid,
                    pidPath: [pid],
                    processName: appData.name,
                    start: Date.now(),
                };

                const realGetRunningGames = RunningGameStore.getRunningGames;
                const realGetGameForPID = RunningGameStore.getGameForPID;

                RunningGameStore.getRunningGames = () => [fakeGame];
                RunningGameStore.getGameForPID = () => fakeGame;

                FluxDispatcher.dispatch({
                    type: "RUNNING_GAMES_CHANGE",
                    removed: RunningGameStore.getRunningGames() || [],
                    added: [fakeGame],
                    games: [fakeGame]
                });

                const onHeartbeat = data => {
                    let progress = quest.config.configVersion === 1 
                        ? data.userStatus.streamProgressSeconds 
                        : Math.floor(data.userStatus.progress?.PLAY_ON_DESKTOP?.value ?? 0);
                    console.log(`[${questName}] Game progress: ${progress}/${secondsNeeded}`);

                    if (progress >= secondsNeeded) {
                        console.log(`[${questName}] PLAY_ON_DESKTOP completed!`);
                        RunningGameStore.getRunningGames = realGetRunningGames;
                        RunningGameStore.getGameForPID = realGetGameForPID;
                        FluxDispatcher.dispatch({ type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: [] });
                        FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", onHeartbeat);
                    }
                };

                FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", onHeartbeat);
                console.log(`[${questName}] Spoofed game → ${applicationName}. Keep app open.`);
            } catch (err) {
                console.error(`[${questName}] PLAY_ON_DESKTOP setup failed:`, err);
            }
        } 
        else if (taskName === "STREAM_ON_DESKTOP") {
            if (!isApp) {
                console.warn(`[${questName}] STREAM_ON_DESKTOP requires Discord Desktop app — skipping.`);
                return;
            }

            const realStreamFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata;
            ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({ id: applicationId, pid, sourceName: null });

            const onHeartbeat = data => {
                let progress = quest.config.configVersion === 1 
                    ? data.userStatus.streamProgressSeconds 
                    : Math.floor(data.userStatus.progress?.STREAM_ON_DESKTOP?.value ?? 0);
                console.log(`[${questName}] Stream progress: ${progress}/${secondsNeeded}`);

                if (progress >= secondsNeeded) {
                    console.log(`[${questName}] STREAM_ON_DESKTOP completed!`);
                    ApplicationStreamingStore.getStreamerActiveStreamMetadata = realStreamFunc;
                    FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", onHeartbeat);
                }
            };

            FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", onHeartbeat);
            console.log(`[${questName}] Spoofed stream → ${applicationName}. Actually stream something in VC (needs ≥1 other person).`);
        } 
        else if (taskName === "PLAY_ACTIVITY") {
            const channel = ChannelStore.getSortedPrivateChannels()[0] ?? 
                Object.values(GuildChannelStore.getAllGuilds()).flatMap(g => g.VOCAL ?? []).find(c => c)?.channel;
            
            if (!channel?.id) {
                console.error(`[${questName}] No voice channel found for PLAY_ACTIVITY — skipping.`);
                return;
            }

            const streamKey = `call:${channel.id}:1`;

            async function activityLoop() {
                while (true) {
                    try {
                        const res = await api.post({
                            url: `/quests/${quest.id}/heartbeat`,
                            body: { stream_key: streamKey, terminal: false }
                        });
                        const progress = res.body.progress?.PLAY_ACTIVITY?.value ?? 0;
                        console.log(`[${questName}] Activity progress: ${progress}/${secondsNeeded}`);

                        if (progress >= secondsNeeded) {
                            await api.post({ url: `/quests/${quest.id}/heartbeat`, body: { stream_key: streamKey, terminal: true } });
                            console.log(`[${questName}] PLAY_ACTIVITY completed!`);
                            break;
                        }
                    } catch (err) {
                        console.error(`[${questName}] Heartbeat failed:`, err);
                        break;
                    }
                    await new Promise(r => setTimeout(r, 18000 + Math.random() * 4000)); // ~20s ±2s
                }
            }

            activityLoop();
            console.log(`[${questName}] Sending heartbeats in VC → join a voice channel with at least 1 other person.`);
        }
    }

    // Launch everything in parallel
    Promise.all(quests.map(startQuest))
        .then(() => console.log("All quest handlers started in parallel. Watch the console for updates."))
        .catch(err => console.error("Fatal error in parallel execution:", err));
}


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
