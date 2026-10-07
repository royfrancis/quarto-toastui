# toastui ![deploy](https://github.com/royfrancis/quarto-toastui/workflows/deploy/badge.svg) ![status: experimental](https://github.com/GIScience/badges/raw/master/status/experimental.svg)

A Quarto shortcode extension for embedding TOAST UI Calendar in HTML output.

> [!NOTE]
> All TOASTUI features are not supported by this extension.
> Interactive calendar input through the widget is not supported.

## Features

- Embed TOAST UI Calendar in Quarto HTML output
- Data input via YAML metadata, inline shortcode arguments, or text files
- Configuration via YAML metadata or inline shortcode arguments
- Multiple calendar views (month, week, day)
- Customizable event and popup details
- 12/24hr formats
- Timezone support
- Automatic time-grid hour-range fitting
- Responsive layout
- HTML format is supported

## Install

Add this extension to your project:

```bash
quarto add royfrancis/quarto-toastui
```

This will create an `_extensions/toastui/` folder in your project with all necessary files.

## Quick start

### YAML metadata

Define a calendar in YAML metadata. This example creates a calendar with a single event:

```yaml
toastui:
  calendar-1:
    defaultView: week
    calendars:
      - id: cal1
        name: Personal
        backgroundColor: "#03bd9e"
    events:
      - id: "1"
        calendarId: cal1
        title: "Meeting"
        start: "2026-04-09T09:00:00"
        end: "2026-04-09T10:00:00"
```

Then, add the shortcode to your document where you want the calendar to appear:

```markdown
{{< toastui calendar-1 >}}
```

For more examples and usage, see the [extension docs website](https://royfrancis.github.io/quarto-toastui/).

## Extension Options

These are extension-level options handled directly by the shortcode.

| Parameter | Type | Default | Description |
|---|---|---|---|
| calendar key (positional) | string | none | Selects the metadata block at `toastui.<key>`. |
| `height` | string or number | `600px` | Calendar container height. Numeric values are coerced to pixels; string values are used as-is. |
| `timegridHeight` | string | `200%` | Height of the inner `.toastui-calendar-timegrid` element. Controls the scrollable time grid size in week/day views. |
| `navigation` | boolean-like | `true` | Shows or hides the built-in navigation controls: prev, today, next, and month/week/day buttons. |
| `timeFormat` | `'12h' \| '24h'` | `'24h'` | Sets one clock format for rendered timed event labels, time-grid labels, the current-time indicator, and the event detail popup. |
| `eventDetailItems` | string or string[] | `[]` | Selects event fields to render inside timed events. Any field present in the event data is accepted; known TOAST UI fields like location, attendees, calendar, state etc. use their native icons, while custom fields render as `field: value`. |
| `popupDetailItems` | string or string[] | `[]` | Selects event fields to render in the detail popup. Known TOAST UI fields use their native popup rows and icons; custom fields are appended as `field: value`. |
| `file` | string | none | Path to a delimited text file containing events. Absolute paths are used directly; relative paths are resolved against the input document directory. |
| `file-sep` | string | `\t` | Delimiter used when parsing `file`. |
| `date` | string | unset | Initial calendar date passed to `new Date(...)`. |
| `events` | array of objects | none | Inline event data from YAML metadata. Ignored if `file` is also provided. |
| `calendars` | `CalendarInfo[]` | `[]` | Calendar definitions used for labels and colors. |

## Pass-through TOAST UI Calendar Options

These are passed into the Calendar constructor if present.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `defaultView` | `'month' \| 'week' \| 'day'` | `'week'` | Sets the initial view mode. |
| `useFormPopup` | `boolean` | `false` | Enables the built-in event create/edit popup. Upstream date/time picker styles are also required when used. |
| `useDetailPopup` | `boolean` | `false` | Enables the built-in event detail popup. |
| `isReadOnly` | `boolean` | upstream: `false`; extension default: `true` | Makes the calendar non-editable. |
| `usageStatistics` | `boolean` | upstream: `true`; extension default: `false` | Controls TOAST UI usage statistics collection. |
| `eventFilter` | `(event) => boolean` | `(event) => !!event.isVisible` | Upstream option. Not forwarded by this extension because it isn't in the allowed constructor-option list, and YAML can't express a JavaScript function anyway. Use `isVisible` on individual events, or post-init custom JS, instead. |
| `gridSelection` | `boolean \| { enableClick?: boolean, enableDblClick?: boolean }` | `true` | Configures click and double-click date selection behavior. |
| `timezone` | `TimezoneOptions` | `{ zones: [] }` | Configures calendar time zone handling. |
| `theme` | `ThemeObject` | `DEFAULT_THEME` | Applies TOAST UI theme customizations. |
| `template` | `TemplateObject` | `DEFAULT_TEMPLATE` | Provides custom render templates for events and labels. YAML can't express the real JavaScript functions TOAST UI expects here, so string values are instead treated as `${field}` interpolation templates, see [Templates](#templates). |
| `week` | `WeekOptions` | `DEFAULT_WEEK_OPTIONS` | Weekly and daily view configuration options. |
| `month` | `MonthOptions` | `DEFAULT_MONTH_OPTIONS` | Monthly view configuration options. |
| `autoHourRange` | `boolean-like` | `false` | Extension-only option (not forwarded to TOAST UI). Fills in whichever of `week.hourStart`/`week.hourEnd` isn't already set explicitly, from the exact min/max timed extent of the events, with no padding. Falls back to leaving the upstream `0`/`24` full-day default when there are no timed events. See [Automatic Hour Range](#automatic-hour-range) below. |

### Week Options

| Field | Type | Default | Description |
|---|---|---|---|
| `startDayOfWeek` | `number` | `0` | Start day of week (`0` Sunday to `6` Saturday). |
| `dayNames` | `string[7]` | `[]` | Optional custom labels for week/day views. |
| `narrowWeekend` | `boolean` | `false` | Narrows weekend columns in week/day views. |
| `workweek` | `boolean` | `false` | Excludes weekends in week/day views. |
| `showNowIndicator` | `boolean` | `true` | Shows current-time indicator in week/day view. |
| `showTimezoneCollapseButton` | `boolean` | `false` | Shows timezone collapse button when using multiple zones. |
| `timezonesCollapsed` | `boolean` | `false` | Starts sub-timezones collapsed. |
| `hourStart` | `number` | `0` | Start hour for time grid. |
| `hourEnd` | `number` | `24` | End hour for time grid. |
| `eventView` | `boolean \| ('allday' \| 'time')[]` | `true` | Controls allday/time event panels. |
| `taskView` | `boolean \| ('milestone' \| 'task')[]` | `true` | Controls milestone/task panels. |
| `collapseDuplicateEvents` | `boolean \| object` | `false` | Duplicate event collapsing behavior. |

### Month Options

| Field | Type | Default | Description |
|---|---|---|---|
| `dayNames` | `string[7]` | `['sun','mon','tue','wed','thu','fri','sat']` | Day labels in month view. |
| `startDayOfWeek` | `number` | `0` | Start day of week (`0` Sunday to `6` Saturday). |
| `narrowWeekend` | `boolean` | `false` | Narrows weekend columns in month view. |
| `visibleWeeksCount` | `number` | `0` | Number of visible weeks (`0` means six-week behavior). |
| `isAlways6Weeks` | `boolean` | `true` | Always render six rows in month view. |
| `workweek` | `boolean` | `false` | Excludes weekends in month view. |
| `visibleEventCount` | `number` | `6` | Max visible events per day cell. |

For complete option schemas and semantics, see TOAST UI Calendar docs:

- https://nhn.github.io/tui.calendar/latest/
- https://github.com/nhn/tui.calendar

### Time Zones

```yaml
timezone:
  zones:
    - timezoneName: Europe/Stockholm
```

Write event `start`/`end` as plain wall-clock timestamps (`2026-10-05T09:00:00`, no UTC offset) and set `timezone.zones[0].timezoneName` to the venue's IANA zone. The extension resolves each naive timestamp against that zone (DST-aware, via the browser's `Intl` API) before handing events to TOAST UI, so every viewer sees the same venue-local time regardless of their own device's timezone. A `start`/`end` that already carries an explicit offset or `Z` is left as-is. With no `timezone` configured, naive timestamps keep the upstream default: parsed as each viewer's own local clock.

Only `zones[0]` is used as the source zone for this resolution. Additional entries just add extra labeled time columns for comparison (`timezoneName` required, plus optional `displayLabel`/`tooltip`), see `showTimezoneCollapseButton`/`timezonesCollapsed` above and TOAST UI's own `TimezoneConfig` docs for the full shape.

The current-time indicator (`week.showNowIndicator`) isn't affected by this resolution. TOAST UI renders it from the browser's actual current moment, per configured zone, independent of event data.

**Quick reference, what each combination actually displays:**

| Event `start`/`end` | `timezone.zones` | Displayed event time | Current-time indicator |
|---|---|---|---|
| Absolute (`...T09:00:00Z`) | Defined | Same wall-clock time for every viewer | Shared/fixed, the configured zone's current time, same for every viewer |
| Absolute (`...T09:00:00Z`) | Not defined | Correctly converted to each viewer's own local time | Per-viewer, each viewer's own real local time |
| Naive (`...T09:00:00`) | Defined | Same wall-clock digits for every viewer (venue-fixed) | Shared/fixed, the configured zone's current time, same for every viewer |
| Naive (`...T09:00:00`) | Not defined | Same literal digits shown to every viewer | Per-viewer, each viewer's own real local time |

### Automatic Hour Range

```yaml
week:
  hourStart: 0 # omit, or set explicitly to opt out for that side
autoHourRange: true
```

Setting `autoHourRange: true` fills in whichever of `week.hourStart`/`week.hourEnd` you didn't set explicitly, based on the events shown:

- Scans each event's `start`/`end`, ignoring all-day events.
- Sets the range to the exact min/max timed extent of the events, with no padding.
- A timed event that spans midnight (its `start` and `end` fall on different dates) is treated as spanning the full day, since a single hour window can't bound it.
- If there are no timed events to measure, `hourStart`/`hourEnd` are left unset and the upstream `0`/`24` full-day default applies.
- If you set one of `week.hourStart`/`week.hourEnd` explicitly, that value is kept and only the other side is auto-fit.

This computation runs in the browser, once, when the calendar initializes (not in Lua at Quarto render time). That's intentional: when `timezone.zones` is set, event `start`/`end` values are repositioned to `zones[0]`, the configured venue timezone, not necessarily the viewer's own clock (see [Time Zones](#time-zones) below), and that repositioning itself happens client-side. A build-time-computed range would be wrong for viewers in other timezones; running the scan client-side keeps the computed range consistent with how the events actually render for that viewer.

**Limitation:** `week.hourStart`/`week.hourEnd` is a single fixed window. If you configure `timezone.zones` with multiple named zones and a user-toggleable primary zone, the range is computed once at calendar init and does not refit if a viewer switches the primary zone afterward, the same is true of a manually-set `hourStart`/`hourEnd` today.

### Templates

```yaml
template:
  milestone: "Custom: ${title}"
```

Upstream `template.*` options are JavaScript functions `(model) => string`, which plain YAML cannot express. Instead, give any `template.*` entry a string, and the extension turns it into a function that substitutes `${field}` placeholders with the matching property from the model object TOAST UI passes to that template (e.g. `${title}`, `${start}`, or a nested path like `${raw.project}`); missing/`null` values resolve to an empty string, and the inserted values are HTML-escaped. This only applies to string values; `template` entries that are objects/arrays/numbers/booleans are passed through unchanged (and, per `TemplateObject`, upstream will ignore or error on those since it expects a function there too).

For which template names exist and what fields their model objects expose, see TOAST UI's own `TemplateObject`/`Template` docs.

## Event Options

Event objects (from `events:` YAML or event files) map to TOAST UI `EventObject`.

### Required by this extension validation

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | `string` | yes | Event title shown on the card. |
| `start` | `string \| number \| Date` | yes | Event start date/time. |
| `end` | `string \| number \| Date` | yes | Event end date/time. |

### Commonly used EventObject fields

| Field | Type | Default | Description |
|---|---|---|---|
| `id` | `string` | auto/internal if omitted | Event identifier. Recommended for updates/deletes. |
| `calendarId` | `string` | none | Calendar id this event belongs to (should match a `calendars[].id` entry). |
| `title` | `string` | none | Event title text. |
| `body` | `string` | empty | Additional text content. |
| `category` | `'milestone' \| 'task' \| 'time' \| 'allday'` | inferred/upstream behavior | Event type affecting rendering panel. |
| `isAllday` | `boolean` | `false` | Marks event as all-day. |
| `start` | `string \| number \| Date \| TZDate` | none | Start date/time. |
| `end` | `string \| number \| Date \| TZDate` | none | End date/time. |
| `location` | `string` | empty | Location label on event detail/card templates. |
| `attendees` | `string[]` | `[]` | Optional attendee list. |
| `state` | `'Busy' \| 'Free' \| string` | hidden | Free/busy state. Only shown in event or popup details when selected and explicitly set. |
| `dueDateClass` | `string` | empty | Optional class/tag used by task/milestone displays. |
| `recurrenceRule` | `string` | empty | Recurrence rule text. |
| `isVisible` | `boolean` | `true` | Visibility flag (also used by default `eventFilter`). |
| `isPending` | `boolean` | `false` | Marks event as pending. |
| `isFocused` | `boolean` | `false` | Focus state metadata. |
| `isReadOnly` | `boolean` | inherits calendar/global behavior | Per-event read-only override. |
| `isPrivate` | `boolean` | `false` | Marks event as private. |
| `color` | `string` | inherited | Event text color. |
| `backgroundColor` | `string` | inherited | Event card background color. |
| `dragBackgroundColor` | `string` | inherited | Event background while dragging. |
| `borderColor` | `string` | inherited | Event border color. |

### Notes

- This extension forwards event fields directly to TOAST UI (`createEvents`).
- Event files are plain delimited text (TSV/CSV style): header names must match field names.
- String values `true` and `false` in files are converted to booleans.
- Text color does not auto-contrast in this extension; set `color` explicitly if needed.
- When using `useDetailPopup: true`, events should have a `calendarId` that references a defined `calendars` entry. Without this, the popup may render with incorrect colours and fail to dismiss on click.
- Optional timed-event details are opt-in through `eventDetailItems`; optional popup rows are independently opt-in through `popupDetailItems`.
- Both options accept arbitrary event fields, including columns loaded from delimited files. Known TOAST UI fields retain their native icons; custom fields are labeled with their field names.
- A selected detail appears only when the event provides matching data; upstream defaults such as `state: "Busy"` remain suppressed for events that do not define them.
- For full upstream definitions, see EventObject docs:
  - https://nhn.github.io/tui.calendar/latest/EventObject

## Event File Format

The first line is a header row. Each following row becomes an event object.

- Column names become object keys
- true and false are converted to booleans
- `goingDuration` and `comingDuration` are converted to numbers
- A non-empty `attendees` value is wrapped into a single-element array
- Other values are left as strings

### Minimum event file requirements

To pass extension validation, an event file must satisfy all of the following:

- Include a header row as the first line
- Include required columns: `title`, `start`, `end`
- Include at least one data row after the header
- Use the correct delimiter for the file and set `file-sep` accordingly (`\t` for TSV, `,` for CSV)

If any of these requirements are not met, the extension does not crash. It renders a styled error message in the output document instead.

Example TSV file:

```text
id  calendarId  title  category  start  end  isAllday  location  backgroundColor
1  cal1  Team Meeting  time  2026-04-09T09:00:00  2026-04-09T10:00:00  false  Room A  #03bd9e
```

## Typical Usage

### Metadata-driven widget

```yaml
---
format: html
toastui:
  calendar-1:
    defaultView: week
    height: 700px
    calendars:
      - id: cal1
        name: Personal
        backgroundColor: "#03bd9e"
    events:
      - id: "1"
        calendarId: cal1
        title: "Standup"
        start: "2026-04-09T09:00:00"
        end: "2026-04-09T09:30:00"
---

{{< toastui calendar-1 >}}
```

### Metadata with file input

```yaml
---
format: html
toastui:
  calendar-1:
    defaultView: month
    file: events.txt
    file-sep: "\t"
---

{{< toastui calendar-1 >}}
```

### Fully inline shortcode

```markdown
{{< toastui file="events.txt" file-sep="\t" defaultView="month" height="520px" >}}
```

## Limitations

- RevealJS is supported but known to be flaky
- For other formats, the shortcode emits no output
- Short duration events may not be displayed legibly depending on the max day duration
- The calendar may not display legibly on very small screens
- TOAST UI Calendar keeps `theme` state in a store shared across every calendar instance on the page, not per instance: setting a custom theme on one calendar can eventually change the appearance of other calendar instances on the same page too (including ones that render earlier in the document), regardless of their own `theme` setting. This extension works around it for `theme.common.backgroundColor` only (reinforced per instance via scoped CSS, so that one field stays isolated); other `theme.*` fields remain plain pass-through and are still subject to this upstream behavior. If you rely on per-instance theming beyond the background color, test with every calendar that will appear on the same page.

## Acknowledgements

- [Toast UI](https://ui.toast.com/tui-calendar) for the calendar library
- [Quarto](https://quarto.org/) for the publishing framework

---

2026 • Roy Francis
