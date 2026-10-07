---
title: Versions
format: html
---

## v1.2.0

- Added `autoHourRange` to automatically fill in whichever of `week.hourStart`/`week.hourEnd` isn't explicitly set, based on the exact min/max timed extent of the events (no padding), falling back to the full day when there are no timed events. Computed client-side at calendar initialization, after event times are resolved (see below), so it reflects how events actually render for each viewer rather than a value fixed at Quarto render time.
- Event `start`/`end` values without a UTC offset are now resolved against `timezone.zones[0]`'s IANA zone (DST-aware, via the browser's `Intl` API) when `timezone` is configured, instead of each viewer's own local clock. Lets a fixed-venue schedule use plain wall-clock timestamps in source data (no manual `+02:00`/`+01:00`, no DST tracking) while still rendering the same time for every viewer. Values that already carry an explicit offset or `Z` are unaffected. With no `timezone` configured, naive timestamps keep the previous behavior.
- String `template.*` values (e.g. `template: { milestone: "Custom: ${title}" }`) are now converted into `${field}`-interpolating render functions, since YAML cannot express the JavaScript functions TOAST UI expects there. Previously, any string value in `template` was passed through as-is and TOAST UI calling it as a function threw uncaught, silently breaking every calendar rendered after it on the page.
- Metadata `events:` is no longer built or validated when `file` is also set, since `file` always wins. Previously an invalid field (e.g. a non-table `attendees`) in a discarded `events:` block could still surface a validation error.
- Events resolving to a YAML mapping instead of a list (from `events:` or a file) now produce a clear validation error (`events from <source> must be a list, not a mapping`) instead of failing silently later.
- `file` paths are now recognized as absolute on Windows (e.g. `C:\data\events.csv`), not just POSIX-style paths starting with `/`, so they resolve correctly regardless of the document's directory.
- Fixed validation error messages not rendering line breaks between multiple errors, caused by a literal `&#10;` check that `escape_html_attr` never actually produces.
- `navigation: "FALSE"` and other case variations are now recognized the same as `navigation: false`, matching the already-documented behavior.
- Fixed `theme.common.backgroundColor` bleeding across calendar instances: TOAST UI Calendar shares its theme state across every calendar on the page, so a custom background set on one instance could end up applied to other calendar instances on the same page too. Each instance's background is now reinforced via scoped CSS so it stays isolated regardless of other calendars on the page. Other `theme.*` fields are not covered by this fix (see README Limitations).

## v1.1.0

- Added `eventDetailItems` for optional details inside timed events and `popupDetailItems` for popup. Both are optional and default to none, while popup title and date remain visible. They accept any arbitrary field present in event data. Known TOAST UI fields retain native icons, while custom fields render as labeled values. Popup line-height slightly decreased.
- PNG icons replaced with SVG. Icons take the same color as text.
- Added `timeFormat`, defaulting to `24h`, to use a consistent clock for rendered timed events, time-grid labels, the current-time indicator, and detail popups. Allows switching between `24h` and `12h` formats.
- Fixed timed events shorter than 30 minutes overlapping adjacent, non-overlapping events.
- Date text in the header scale down for narrow screens to avoid overlap
- Event and popup formatting improved.

## v1.0.0

- Initial shortcode extension created for TOAST UI Calendar.
- Bundled TOAST UI assets in extension assets directory.
- Added metadata-driven and inline shortcode configuration support.
- Added file-based event ingestion with configurable separators.
- Added custom navigation toolbar with view switching.
- Expanded README with full option and parameter documentation.
