# OpenPrice ORB Indicator Implementation Plan

**Goal:** Build a focused TradingView Pine Script indicator that plots the 9:30 AM New York OpenPrice and current-day ORB high/low levels.

**Architecture:** One Pine Script v6 overlay indicator owns the inputs and drawing objects. It calculates levels from 1-minute data with `request.security()` and draws three timestamp-anchored horizontal `line` objects plus three bar-index-anchored `label` objects on the active chart.

**Files:**

- `indicators/open-orb.pine`: complete indicator source.
- `README.md`: install instructions and manual validation checklist.
- `docs/superpowers/specs/2026-07-02-open-orb-indicator-design.md`: design spec.

**Validation:**

- Compile the Pine source in TradingView.
- Check 1-minute, 5-minute, 38-minute, and 4-hour intraday charts.
- Check daily charts hide the levels.
- Check `Show OpenPrice` and `Show ORB` independently hide their levels.
- Confirm labels sit two visible chart bars to the right of the latest bar.
