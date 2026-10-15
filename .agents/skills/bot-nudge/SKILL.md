---
name: bot-nudge
description: Phone escalation via Apple Reminders (Mac-iPhone only). Opt-in by judgment - a bot nudges kky's phone only for urgent, time-critical events it would want woken for. Use when something is production-impacting and cannot wait for the next thread check; never for routine reports.
---

# Nudge (phone escalation)

`bin/nudge <title> [lead-minutes]` puts a reminder on kky's iPhone (default list "提醒", iCloud) that fires as a banner + sound after the lead time (default 15 minutes; never below 3 - iCloud sync arrives late and iOS then drops it silently).

## When a nudge applies (opt-in judgment)

Ask first: "would kky want his phone to buzz for this, right now?" Default is **no**. Nudge only when ALL hold:

- Production-impacting and urgent: something is broken or about to be (fleet down, alert firing, deadline in minutes) - not merely annoying or interesting.
- In-thread contact cannot wait: the next heartbeat/check is too late for the consequence.
- kky has asked for this class of event, or it is an emergency by any reasonable reading.

Never nudge for: routine results, CI failures with a workaround, interesting findings, anything that can wait until morning. Daily hard cap: 2 per bot (bin/nudge enforces it; the count lives in `me/.nudge-log`).

## How

1. `bin/nudge "<what happened, one line>"` - the bot name and timestamp are added automatically. Default 15-minute lead; pass a second argument to change it.
2. Log the same line under "Waiting on the user" in `me/NOW.md` (the phone buzzes, but the thread must carry the details).
3. After the fire time, check whether kky responded in-thread; if not and it is still urgent, one rephrase-level re-nudge is allowed - never a loop.

The channel is Mac-iPhone by design (kky's call, 2026-10-15); other devices are unsupported. It replaces relying on T3 Code notifications, which do not reach his phone.
