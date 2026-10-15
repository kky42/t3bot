---
name: bot-nudge
description: Phone escalation via Apple Reminders (Mac-iPhone only). Opt-in - a bot nudges the user's phone only for event classes the user listed under "Nudge-worthy events" in me/USER.md, or for active data loss or an active fleet outage. Use when such an event is happening and cannot wait for the next thread check; never for routine reports.
---

# Nudge (phone escalation)

`bin/nudge <title> [lead-minutes]` puts a reminder on the user's iPhone (iCloud) that fires as a banner + sound after the lead time. Lead is 10-120 minutes, default 15: shorter leads arrive past due after iCloud sync and iOS drops them silently; longer ones are not an escalation. The list comes from the `- Nudge list: <name>` line in `me/USER.md` (the shared profile; the phone is the user's, not one bot's); without it, the Reminders default list is used.

## When a nudge applies (opt-in)

Default is **no**. Nudge only when one of these holds:

- (a) The event's class is listed under "Nudge-worthy events" in `me/USER.md`.
- (b) Active data loss, or an active fleet outage, happening right now.

Nothing else qualifies, however urgent it feels. Never nudge for: routine results, CI failures, interesting findings, anything that can wait until morning.

Nudges are allowed during quiet hours (that is their purpose), but only for qualifying events. Any session or worker report that surfaces a qualifying event may nudge immediately; do not wait for the next heartbeat.

Caps (bin/nudge enforces them): 2 per bot per day (`me/.nudge-log`), 4 across all bots per day (`~/.t3codes/.nudge-log`), and none within 10 minutes of the last nudge from any bot - if another bot just nudged, add your news to the thread instead. A nudge is logged before it is created, so a failed attempt still counts.

## How

1. `bin/nudge "<what happened, one line>"` - the bot name and timestamp are added automatically. It prints `NUDGE: sent (<bot>, fires in <N>m) <id>` or `NUDGE: FAIL - <reason>`; on FAIL, tell the user in-thread and do not retry around the cap.
2. Add `- <date> <HH:MM> nudge: <title> (waiting on user)` under "Waiting on the user" in `me/NOW.md`, with the same date, time, and title as the log line (the phone buzzes, but the thread must carry the details).
3. Re-nudge: at most one per event, and it counts toward the cap. Only after the fire time has passed with no response from the user in-thread, and only if the event still qualifies and is still urgent. Never a loop.

The channel is Mac-iPhone by design (the user's call, 2026-10-15); other devices are unsupported. It replaces relying on T3 Code notifications, which do not reach the phone.
