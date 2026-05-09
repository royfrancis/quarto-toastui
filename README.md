# toastui ![deploy](https://github.com/royfrancis/quarto-toastui/workflows/deploy/badge.svg) ![status: experimental](https://github.com/GIScience/badges/raw/master/status/experimental.svg)

A Quarto shortcode extension for embedding TOAST UI Calendar in HTML output.

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
    height: 700px
    isReadOnly: true
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
| height | string or number | `600px` | Calendar container height. Numeric values are coerced to pixels; string values are used as-is. |
| timegridHeight | string | `200%` | Height of the inner `.toastui-calendar-timegrid` element. Controls the scrollable time grid size in week/day views. |
| navigation | boolean-like | `true` | Shows or hides the built-in navigation controls: prev, today, next, and month/week/day buttons. |
| file | string | none | Path to a delimited text file containing events. Absolute paths are used directly; relative paths are resolved against the input document directory. |
| file-sep | string | `\t` | Delimiter used when parsing `file`. |
| date | string | unset | Initial calendar date passed to `new Date(...)`. |
| events | array of objects | none | Inline event data from YAML metadata. Ignored if `file` is also provided. |
| calendars | `CalendarInfo[]` | `[]` | Calendar definitions used for labels and colors. |

## Pass-through TOAST UI Calendar Options

These are passed into the Calendar constructor if present.

| Parameter | Type | Default | Description |
|---|---|---|---|
| defaultView | `'month' \| 'week' \| 'day'` | `'week'` | Sets the initial view mode. |
| useFormPopup | `boolean` | `false` | Enables the built-in event create/edit popup. Upstream date/time picker styles are also required when used. |
| useDetailPopup | `boolean` | `false` | Enables the built-in event detail popup. |
| isReadOnly | `boolean` | upstream: `false`; extension default: `true` | Makes the calendar non-editable. |
| usageStatistics | `boolean` | upstream: `true`; extension default: `false` | Controls TOAST UI usage statistics collection. |
| eventFilter | `(event) => boolean` | `(event) => !!event.isVisible` | Upstream option. Not practically configurable through YAML because it requires a JavaScript function. |
| gridSelection | `boolean \| { enableClick?: boolean, enableDblClick?: boolean }` | `true` | Configures click and double-click date selection behavior. |
| timezone | `TimezoneOptions` | `{ zones: [] }` | Configures calendar time zone handling. |
| theme | `ThemeObject` | `DEFAULT_THEME` | Applies TOAST UI theme customizations. |
| template | `TemplateObject` | `DEFAULT_TEMPLATE` | Provides custom render templates for events and labels. |
| week | `WeekOptions` | `DEFAULT_WEEK_OPTIONS` | Weekly and daily view configuration options. |
| month | `MonthOptions` | `DEFAULT_MONTH_OPTIONS` | Monthly view configuration options. |

### WeekOptions defaults

| Field | Type | Default | Description |
|---|---|---|---|
| startDayOfWeek | `number` | `0` | Start day of week (`0` Sunday to `6` Saturday). |
| dayNames | `string[7]` | `[]` | Optional custom labels for week/day views. |
| narrowWeekend | `boolean` | `false` | Narrows weekend columns in week/day views. |
| workweek | `boolean` | `false` | Excludes weekends in week/day views. |
| showNowIndicator | `boolean` | `true` | Shows current-time indicator in week/day view. |
| showTimezoneCollapseButton | `boolean` | `false` | Shows timezone collapse button when using multiple zones. |
| timezonesCollapsed | `boolean` | `false` | Starts sub-timezones collapsed. |
| hourStart | `number` | `0` | Start hour for time grid. |
| hourEnd | `number` | `24` | End hour for time grid. |
| eventView | `boolean \| ('allday' \| 'time')[]` | `true` | Controls allday/time event panels. |
| taskView | `boolean \| ('milestone' \| 'task')[]` | `true` | Controls milestone/task panels. |
| collapseDuplicateEvents | `boolean \| object` | `false` | Duplicate event collapsing behavior. |

### MonthOptions defaults

| Field | Type | Default | Description |
|---|---|---|---|
| dayNames | `string[7]` | `['sun','mon','tue','wed','thu','fri','sat']` | Day labels in month view. |
| startDayOfWeek | `number` | `0` | Start day of week (`0` Sunday to `6` Saturday). |
| narrowWeekend | `boolean` | `false` | Narrows weekend columns in month view. |
| visibleWeeksCount | `number` | `0` | Number of visible weeks (`0` means six-week behavior). |
| isAlways6Weeks | `boolean` | `true` | Always render six rows in month view. |
| workweek | `boolean` | `false` | Excludes weekends in month view. |
| visibleEventCount | `number` | `6` | Max visible events per day cell. |

For complete option schemas and semantics, see TOAST UI Calendar docs:

- https://nhn.github.io/tui.calendar/latest/
- https://github.com/nhn/tui.calendar

## Event Options

Event objects (from `events:` YAML or event files) map to TOAST UI `EventObject`.

### Required by this extension validation

| Field | Type | Required | Description |
|---|---|---|---|
| title | `string` | yes | Event title shown on the card. |
| start | `string \| number \| Date` | yes | Event start date/time. |
| end | `string \| number \| Date` | yes | Event end date/time. |

### Commonly used EventObject fields

| Field | Type | Default | Description |
|---|---|---|---|
| id | `string` | auto/internal if omitted | Event identifier. Recommended for updates/deletes. |
| calendarId | `string` | none | Calendar id this event belongs to (should match a `calendars[].id` entry). |
| title | `string` | none | Event title text. |
| body | `string` | empty | Additional text content. |
| category | `'milestone' \| 'task' \| 'time' \| 'allday'` | inferred/upstream behavior | Event type affecting rendering panel. |
| isAllday | `boolean` | `false` | Marks event as all-day. |
| start | `string \| number \| Date \| TZDate` | none | Start date/time. |
| end | `string \| number \| Date \| TZDate` | none | End date/time. |
| location | `string` | empty | Location label on event detail/card templates. |
| attendees | `string[]` | `[]` | Optional attendee list. |
| state | `'Busy' \| 'Free' \| string` | hidden | Free/busy state. Only shown in the detail popup when explicitly set. |
| dueDateClass | `string` | empty | Optional class/tag used by task/milestone displays. |
| recurrenceRule | `string` | empty | Recurrence rule text. |
| isVisible | `boolean` | `true` | Visibility flag (also used by default `eventFilter`). |
| isPending | `boolean` | `false` | Marks event as pending. |
| isFocused | `boolean` | `false` | Focus state metadata. |
| isReadOnly | `boolean` | inherits calendar/global behavior | Per-event read-only override. |
| isPrivate | `boolean` | `false` | Marks event as private. |
| color | `string` | inherited | Event text color. |
| backgroundColor | `string` | inherited | Event card background color. |
| dragBackgroundColor | `string` | inherited | Event background while dragging. |
| borderColor | `string` | inherited | Event border color. |

### Notes

- This extension forwards event fields directly to TOAST UI (`createEvents`).
- Event files are plain delimited text (TSV/CSV style): header names must match field names.
- String values `true` and `false` in files are converted to booleans.
- Text color does not auto-contrast in this extension; set `color` explicitly if needed.
- When using `useDetailPopup: true`, events should have a `calendarId` that references a defined `calendars` entry. Without this, the popup may render with incorrect colours and fail to dismiss on click.
- The detail popup only shows `location`, `attendees`, and `state` sections when the event explicitly provides those fields. Upstream TOAST UI defaults (e.g. `state: "Busy"`) are suppressed so that the popup stays clean for events that don't define them.
- For full upstream definitions, see EventObject docs:
  - https://nhn.github.io/tui.calendar/latest/EventObject

## Event File Format

The first line is a header row. Each following row becomes an event object.

- Column names become object keys
- true and false are converted to booleans
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

## Output and Format Notes

- Renders for standard HTML output
- RevealJS is supported but known to be flaky
- For other formats, the shortcode emits no output

## Acknowledgements

- [Toast UI](https://ui.toast.com/tui-calendar) for the calendar library
- [Quarto](https://quarto.org/) for the publishing framework

---

2026 • Roy Francis
