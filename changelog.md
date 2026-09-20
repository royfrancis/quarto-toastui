---
title: Versions
format: html
---

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
