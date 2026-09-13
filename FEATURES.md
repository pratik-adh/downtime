# Downtime - full feature reference

Downtime is a macOS menu-bar app that's concerned about your health: it
looks after your eyes, your back and your hydration through a working day at the
screen. It also knows how to stay out of your way - holding reminders during a
meeting or a fullscreen app.
This document covers every feature, every setting, every default, and the
behaviour behind each one. For the overview and install instructions, see
[README.md](README.md).

- [Menu bar](#menu-bar)
- [The dashboard](#the-dashboard-left-click)
- [Reminders](#reminders)
- [Break screens](#break-screens)
- [Snooze, skip and done](#snooze-skip-and-done)
- [Presence: idle, lock and sleep](#presence-idle-lock-and-sleep)
- [Work hours](#work-hours)
- [Fullscreen apps](#fullscreen-apps)
- [Calendar-aware pausing](#calendar-aware-pausing)
- [Pausing and resuming](#pausing-and-resuming)
- [Water tracking](#water-tracking)
- [Stats and streaks](#stats-and-streaks)
- [Custom wording](#custom-wording)
- [Sound](#sound)
- [Open at login](#open-at-login)
- [Shortcuts (App Intents)](#shortcuts-app-intents)
- [Settings backup](#settings-backup)
- [Appearance and design tokens](#appearance-and-design-tokens)
- [Settings reference](#settings-reference)
- [Keyboard shortcuts](#keyboard-shortcuts)
- [Where your data lives](#where-your-data-lives)

---

## Menu bar

The app runs as an `LSUIElement` accessory: **menu bar only, no Dock icon, no
main window.**

The status item shows the icon of whichever reminder is next, plus a live
countdown to it:

| State             | Icon                       | Text                                           |
| ----------------- | -------------------------- | ---------------------------------------------- |
| Counting down     | the next reminder's symbol | `20m`, `1h04`, or `19:58` with seconds         |
| Break on screen   | that break's symbol        | the break's own countdown, always with seconds |
| Paused            | the pause reason's symbol  | _(none)_                                       |
| All reminders off | `bell.slash`               | _(none)_                                       |

**Countdown format** - under an hour it reads `45m`; over an hour, `1h04`. Turn
on seconds and it becomes `44:59` / `1:04:32`. Both are rendered in a
monospaced-digit font so the width doesn't jitter as the numbers tick.

**Hover tooltip** lists every enabled reminder and its remaining time at once,
or the current pause reason.

**Stable position.** The status item's own width changes as its content does -
a countdown collapsing to a bare paused icon is a ~40pt difference - which
slides it and everything left of it sideways. When the dashboard is open, it
tracks the button's real on-screen position and re-centres itself, so pausing or
resuming never leaves the popover stranded off-anchor.

**Click behaviour:**

- **Left-click** - opens the dashboard popover.
- **Right-click** (or Control-click) - a native menu: current status, Take a
  break now, Restart all timers, Pause reminders ▸, Settings… (`⌘,`),
  Quit (`⌘Q`).

---

## The dashboard (left-click)

A 332pt-wide card, sized to its contents on every open:

1. **Header** - app name, a pause/resume menu, and a Settings button.
2. **Hero ring** - a large progress ring counting down to the next reminder,
   with its symbol and label ("until eyes"). While paused it reads `paused` and
   the ring greys out. Below it: _"At the screen for 1h 12m"_ - your longest
   current unbroken stretch - or the pause reason if reminders are on hold.
3. **Reminder rows** - one per reminder: progress ring, name, interval,
   remaining time, and a switch to toggle it. Toggling takes effect instantly.
4. **Water tracker** - a row of droplets with `−` / `+`.
5. **Break now** + **Restart all timers**.
6. **Today's stats** - breaks taken, day streak, screen time.
7. **7-day chart** - a bar per day against your goal, today in bold.
8. **Footer** - Settings (gear), Pause/Resume, Quit.

The footer's **Pause** is a plain toggle: one tap holds reminders until you press
Resume. The timed choices ("for 20 minutes", "until tomorrow") live in the header
menu. When reminders are held by something you can't lift - you're away, or a
fullscreen app is up - the button is disabled rather than pretending to work.

---

## Reminders

Three independent reminders, each with its own timer, interval, break length,
colour, symbol, copy and tips.

|                  | Stand & Stretch                                           | Drink Water                                        | Rest Your Eyes                                               |
| ---------------- | --------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| Looks after      | back, hips, circulation                                   | hydration, energy levels                           | eye strain, dryness                                          |
| Default interval | 60 min                                                    | 45 min                                             | 20 min                                                       |
| Default break    | 3 min                                                     | 30 s                                               | 20 s                                                         |
| Symbol           | `figure.walk`                                             | `drop.fill`                                        | `eye.fill`                                                   |
| Instruction      | "Stand, straighten your back, and reach for the ceiling." | "Drink at least a few mouthfuls - not just a sip." | "Look at something about 20 feet (6 m) away for 20 seconds." |

**Interval choices:** 20, 30, 45 min, 1 h, 1.5 h, 2 h, 3 h, 4 h, 5 h.
**Break-length choices:** 20 s, 30 s, 1, 2, 3, 5, 10 min.

**Changing an interval restarts that timer** from zero, as does switching a
reminder back on - so a change takes effect predictably instead of firing
immediately because the old counter was already past the new interval.

### When two come due at once

You get **one** card, not a queue. The reminder with the **longest break** owns
the card, because a long break also serves the short ones. The others ride along
as _"also due"_ chips.

A reminder is only folded in if the owning break is **at least as long** as its
own - so a 20-second eye break never silently counts as your 3-minute stretch.
Anything that was cut short is pushed out rather than fired again immediately.

### The 20-20-20 preset

Settings → Reminders → **Apply the 20-20-20 rule** sets the eye reminder to
every 20 minutes / 20 second break and switches it on.

---

## Break screens

| Style                   | Behaviour                                                                                                                                      |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Gentle**              | Small card in the top-right corner. Never steals focus, never covers your work.                                                                |
| **Focused** _(default)_ | Frosted dim over the whole screen with the card in the middle. Skippable.                                                                      |
| **Strict**              | Same, but Okay and Esc are locked out until the timer reaches zero.                                                                            |
| **Notification**        | No window at all - a native macOS notification with Done / Snooze / Skip actions. Asks for notification permission the first time you pick it. |

The card shows: the reminder name, a countdown ring, the headline, the message
(with your real screen time substituted in), the instruction, any "also due"
chips, an optional wellbeing tip, and the buttons.

**Multiple displays** - Focused and Strict dim every display (optional, on by
default) and put the card on your active screen. The overlay windows join all
Spaces and sit at screen-saver level.

**Strict mode** only holds you for _automatic_ breaks - a break you asked for
yourself with "Break now" is always skippable, so the strict setting can't lock
you out of a break you opted into.

**When the timer hits zero** the card switches to "Break complete", optionally
chimes, and by default closes itself.

The **Notification** style has no window and no keyboard shortcuts - use the
notification's own Done / Snooze / Skip buttons instead. Everything else
(counting down, "also due" folding, stats) works exactly the same underneath;
only how the break is announced changes.

---

## Snooze, skip and done

**Snooze** - default 10 minutes. Choices: 3, 5, 10, 15, 20, 30 min. A chevron
next to the button offers the other lengths without opening Settings.

**Snooze cap** - default 3 per break; also 1, 2, 5, or Unlimited. Once the cap is
hit the button disappears for that break. The card shows how many you have left.

Snoozing pushes the reminder out by the full snooze length even if that's longer
than the reminder's own interval, and takes any "also due" reminders with it.

**Done / Okay** - counts as a break taken if the timer finished _or_ you gave it
at least half the break. Otherwise it counts as skipped, honestly.

**Skip** - always counts as skipped.

Finishing a **water** break (or one that served water) logs a glass
automatically.

After any break there's a **45-second grace period** before another can fire,
so finishing one break never immediately triggers the next.

---

## Presence: idle, lock and sleep

Idle time comes from `CGEventSource.secondsSinceLastEventType` across keyboard,
mouse, drag and scroll events. **No Accessibility permission required.**

Two thresholds:

| Setting                              | Default | Choices                         | Meaning                                                      |
| ------------------------------------ | ------- | ------------------------------- | ------------------------------------------------------------ |
| **Hold the timer after**             | 90 s    | 30 s, 1 min, 90 s, 3 min, 5 min | Stop counting screen time after this much inactivity         |
| **Count time away as a break after** | 5 min   | 2, 3, 5, 10, 15, 30 min         | Being away this long _resets_ the timers and credits a break |

So: a short pause holds the clock; a real absence counts as the break it was.
Coming back from lunch doesn't mean an instant reminder.

**Screen lock, display sleep and system sleep** count the same way, via
`NSWorkspace` and the `com.apple.screenIsLocked` notifications - and have to
last just as long as walking away does to earn the credit.

After an away-credit there's a 30-second grace period before anything can fire.

---

## Work hours

Off by default. Pick weekdays and a start/end time, and reminders go quiet
outside them.

- **Overnight windows work** - e.g. 21:00 → 03:00 correctly spans midnight and
  attributes the early-morning hours to the previous day's weekday selection.
- **Start == end means "all day"** on the selected weekdays, not "never".
- Deselecting every weekday is prevented - the last one stays on.

If you hit **Resume** while outside your work hours, that's read as "remind me
anyway": the schedule is overridden until it next comes back around on its own,
rather than re-pausing you a second later.

---

## Fullscreen apps

Off by default. When on, reminders are suppressed while another app owns a
window exactly the size of a display - presentations, films, games.

It's a heuristic (`CGWindowListCopyWindowInfo`, throttled to once every five
seconds), which is exactly why it ships off. If reminders start going missing,
turn it off.

---

## Calendar-aware pausing

Off by default. When on, reminders hold automatically while an event on your
calendar is happening right now - no schedule to configure, it just checks the
current moment.

- Reads calendar events via **EventKit**, polling once every 30 seconds for
  anything active right now (a 2-second window around "now"). It never reads
  your calendar in bulk, never stores an event, and never sends anything
  anywhere - the only thing kept is a live yes/no plus the current event's
  title, shown in Settings so you can see why reminders are quiet.
- **Ignore all-day events** is on by default, so a birthday or a holiday
  doesn't hold your reminders for the whole day.
- Cancelled events are ignored.
- Turning this on is the moment Downtime actually asks macOS for Calendar
  access - it's the one feature in the app that needs a system permission. If
  you decline (or later revoke it in System Settings → Privacy & Security →
  Calendars), Settings → Schedule explains that plainly instead of silently
  doing nothing.
- Like fullscreen detection and being away, this is a fact about the world
  rather than something you chose - the Resume button doesn't appear for it;
  it clears itself when the meeting ends.

---

## Pausing and resuming

From the dashboard header menu or the right-click menu:

- Pause for **20 minutes**, **1 hour**, or **2 hours**
- Pause **until tomorrow** (resumes 5 a.m.)
- Pause **until I turn it back on**

The footer's **Pause** button is the simple version of the last one - one
click stops reminders until you press **Resume**.

Pausing closes any break already on screen cleanly, so a reminder is never lost
mid-break. A timed pause expires by itself.

**While manually paused, the dashboard collapses** the reminder list, water
tracker, actions and stats into a single "Reminders are paused" notice with
its own Resume Break Mode button - the hero ring stays (it already shows the
paused state clearly) but the rest has nothing fresh to show. This only
happens for a real pause; being away, in a fullscreen app, in a meeting, or
outside work hours are transient world-states, and the dashboard stays fully
visible during those since it's still useful to glance at.

**Honest affordances.** The app distinguishes holds you set from facts about the
world. Manual pauses and the work-hours schedule are yours to lift, so Resume
appears and works. Being away or in a fullscreen app clears itself when the
situation does - there, the menu shows the reason instead of a Resume button
that couldn't do anything.

---

## Water tracking

A row of droplets in the dashboard; `+` and `−` to adjust. Finishing a water
break logs one automatically. The daily goal is configurable (1–16, default 8).
Going over the goal shows a `+n` badge rather than hiding the overflow.

---

## Stats and streaks

Recorded per day: **breaks taken**, **breaks skipped**, **snoozes**, **break
seconds**, **screen seconds**, **glasses**, plus (see below) **per-reminder
breaks taken**.

- **7-day bar chart** in the dashboard and in Settings → Stats.
- **Overall streak** - consecutive days that met your daily break goal (1–30,
  default 8). Today only counts once you've _hit_ the goal, so the streak
  doesn't read as broken every morning before you've earned it.
- **Per-reminder streaks** - Stand & Stretch, Drink Water and Rest Your Eyes
  each keep their own streak against their own daily goal (Settings → General
  → _Per-reminder goals_, 1–30 each, defaults 5 / 6 / 8). A day away from your
  desk long enough to be credited as a break counts toward every enabled
  reminder's streak, same as it does for the overall one.
- **This week vs. last week** - Settings → Stats shows breaks taken and
  compliance for the last 7 days against the 7 before that, so a good or bad
  week has something to compare against besides a flat number.
- History is kept for **120 days**, saved to disk every 30 seconds and on quit.
- Settings → Stats → **Erase all history**, with a confirmation.

---

## Custom wording

Settings → Reminders → _Custom wording_ rewrites any reminder's headline and
message. Placeholders are substituted for real:

| Token        | Becomes                                               |
| ------------ | ----------------------------------------------------- |
| `{duration}` | `1 hour 5 minutes` - your actual unbroken screen time |
| `{minutes}`  | `65`                                                  |
| `{time}`     | `4:32 PM`, in your locale                             |
| `{count}`    | which break of the day this is                        |

Leave a field empty to fall back to the default copy.

**Wellbeing tips** - five per reminder, one picked at random per break. Can be
switched off.

---

## Sound

A system sound when a break starts, and optionally a chime when it finishes.

**Choices:** Silent, Ping _(default)_, Glass, Tink, Pop, Purr, Submarine,
Bottle, Blow, Hero, Sosumi. Volume slider with a **Test** button. Choosing
Silent disables the chime and volume controls too.

---

## Open at login

Settings → General → **Open Downtime at login**.

Registers with `SMAppService`. If macOS refuses - which unsigned local builds
sometimes do - it falls back to a LaunchAgent at
`~/Library/LaunchAgents/com.downtime.mac.launcher.plist`. Either way the
toggle reports what actually happened rather than silently failing.

---

## Shortcuts (App Intents)

Four actions are exposed to Shortcuts and Siri, built with **App Intents** -
no separate extension, no entitlement, just Swift in the main app target:

| Shortcut                 | Does                                           |
| ------------------------ | ---------------------------------------------- |
| **Take a Break Now**     | Starts whichever reminder is next up           |
| **Pause Reminders**      | Pauses for a number of minutes (parameterized) |
| **Resume Reminders**     | Resumes reminders                              |
| **Log a Glass of Water** | Adds one glass to today's count                |

Build a Shortcut around these the same way as any other app's actions - e.g.
"When my Focus turns on, Pause Reminders for 60 minutes."

---

## Settings backup

Settings → General → **Export Settings…** / **Import Settings…** saves your
whole configuration (reminders, break style, schedule, calendar
settings, per-reminder goals - everything except your stats history) to a
JSON file and loads it back. Useful for moving to another Mac or keeping a
known-good configuration on hand. Import uses the same forgiving decode as
every other settings load, so a file from an older or newer build still works.

---

## Appearance and design tokens

The palette is defined as **appearance-aware tokens** that resolve differently in
light and dark, rather than one value expected to work against both.

The three brand hues keep their hue angle but get a darker, more saturated
variant for light mode, because a colour tuned to glow on a near-black window
reads as pale on white:

|               | Light                   | Dark                    |
| ------------- | ----------------------- | ----------------------- |
| Stand (green) | `rgb(0.12, 0.58, 0.41)` | `rgb(0.24, 0.80, 0.60)` |
| Water (blue)  | `rgb(0.13, 0.42, 0.69)` | `rgb(0.26, 0.62, 0.96)` |
| Eyes (purple) | `rgb(0.43, 0.31, 0.68)` | `rgb(0.64, 0.49, 0.95)` |

Surface tokens work the same way - card fills, borders and the popover's own
background are each specified per appearance, so card-against-background
contrast lands in the same place in both themes (≈1.20 light, ≈1.16 dark)
instead of light trailing badly behind.

Everything follows the system setting automatically; there is no theme switch to
set.

---

## Settings reference

Seven tabs.

### Reminders

Per reminder: on/off, interval, break length, custom headline, custom message.
Plus the 20-20-20 preset button.

### Break Screen

Style (Gentle / Focused / Strict / Notification) · Dim other displays ·
Snooze length · Snoozes per break · Close automatically when the timer ends ·
Show a wellbeing tip · Preview a break now.

### Schedule

Only remind me during work hours · From/to times · Weekday picker · live
"Right now" status · Hold the timer after · Count time away as a break after ·
Don't interrupt fullscreen apps · **Calendar: pause during meetings, ignore
all-day events, access status** · current idle seconds · Resume now.

### General

Open at login · Show the countdown in the menu bar · Include seconds ·
Reminder sound, volume, Test · Chime when a break finishes · Breaks per day
goal · Glasses per day goal · **per-reminder daily goals** ·
**Export/Import Settings** · About · Restore all defaults.

### Stats

Today's chips (taken / skipped / snoozes / glasses) · screen time · break time ·
7-day chart · overall streak · **this week vs. last week** ·
**per-reminder streaks** · breaks this week · Erase all history.

### Support

Buy me a coffee · feedback note.

**Settings are saved on every change** and decoded forgivingly: every key falls
back to its default and out-of-range values snap to the nearest valid option, so
a settings file from an older or newer build still loads instead of being thrown
away.

---

## Keyboard shortcuts

**While a break is showing:**

| Key                | Action          |
| ------------------ | --------------- |
| `return` / `space` | Okay / I'm done |
| `S`                | Snooze          |
| `esc`              | Skip / dismiss  |

In Strict mode these are inert until the break finishes. Keys are only captured
when the break window itself is focused, so typing in Settings during a break
still works normally.

**From the menu bar menu:** `⌘,` Settings · `⌘Q` Quit.

---

## Where your data lives

| What                           | Where                                                                                                                      |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| Settings                       | user defaults, `com.downtime.mac`, key `downtime.settings.v1`                                                              |
| History                        | `~/Library/Application Support/Downtime/stats.json`                                                                        |
| Login item fallback            | `~/Library/LaunchAgents/com.downtime.mac.launcher.plist`                                                                   |
| Calendar events                | never stored - checked live via EventKit, only a yes/no and the current event's title are kept in memory                   |
| Notifications                  | delivered locally via `UNUserNotificationCenter`; nothing is sent to a server                                              |

No analytics, no accounts, no update check. Nothing leaves your Mac.