# OpenPrice and ORB Indicator Design

## Goal

Build a TradingView Pine Script indicator that plots only two concepts:

- the 9:30 AM New York regular-session OpenPrice
- the opening range breakout (ORB) high and low

The indicator is symbol-agnostic and hides only on daily, weekly, and monthly chart intervals.

## Recommended Approach

Use a single Pine Script v6 indicator file with TradingView drawing objects (`line` and `label`). Calculate levels from 1-minute data through `request.security()` so larger and custom intraday chart intervals still receive the same OpenPrice and ORB levels.

Rejected alternatives:

- Chart-timeframe-only calculation: simpler, but it misses the ORB window on larger or irregular intervals.
- Expanded ORB toolkit: unnecessary because alerts, breakout markers, boxes, and strategy logic are out of scope.

## Core Behavior

The indicator runs on intraday charts, including seconds charts, 4-hour charts, and custom minute intervals. It hides on daily, weekly, and monthly charts.

All session logic uses `America/New_York` time:

- regular session open: 9:30 AM ET
- ORB end: 9:30 AM ET plus the configured ORB duration
- regular session close: 4:00 PM ET

At the first 1-minute regular-session bar at or after 9:30 AM ET, the script captures that bar's open as the OpenPrice. It draws a horizontal OpenPrice line from 9:30 AM ET to 4:00 PM ET when `Show OpenPrice` is enabled.

During the ORB window, the script updates the ORB high and ORB low from 1-minute data. After the ORB window ends, those values lock for the rest of the session. ORB high and low lines use the same 9:30 AM ET to 4:00 PM ET span, with labels for each level, when `Show ORB` is enabled.

Only the current trading day is shown. Prior session drawings are deleted or reused when a new session begins.

## Inputs and Styling

Settings stay minimal:

- `Show OpenPrice`: boolean input, default `true`
- `Show ORB`: boolean input, default `true`
- `ORB Duration Minutes`: integer input, default `15`, minimum `1`
- `OpenPrice Line Color`: default yellow
- `ORB High Color`: default green
- `ORB Low Color`: default red
- `Line Width`: default `2`

No alerts, breakout markers, historical-session count, boxes, trend filters, tables, or symbol restrictions are included.

## Drawing Behavior

Each active session can have exactly three horizontal lines and three labels:

- `OpenPrice`
- `ORB High`
- `ORB Low`

Lines use timestamp anchors from the 9:30 AM ET regular-session open to the 4:00 PM ET regular-session close. Labels use chart bar indexes with `bar_index + 2` and left-aligned text so they sit two chart bars to the right on every intraday timeframe.

## Data Flow and Edge Cases

Before 9:30 AM ET, the current day has no plotted levels.

At or after 9:30 AM ET, 1-minute data captures OpenPrice.

During the ORB window, the ORB high is the highest 1-minute high seen during the window and the ORB low is the lowest 1-minute low seen during the window.

After the ORB window, ORB high and low stop changing. OpenPrice never changes after capture.

Invalid chart contexts stay quiet:

- daily, weekly, and monthly charts hide the indicator

## Validation Plan

Manual TradingView validation is sufficient for this focused Pine indicator:

- 1-minute, 5-minute, 38-minute, and 4-hour charts show OpenPrice and ORB levels.
- Seconds charts show the same levels.
- Daily charts hide.
- A new trading day clears prior levels and shows only the current day.
- Labels sit two chart bars ahead of the latest bar.
- Settings can independently hide OpenPrice levels or ORB levels.

## Out of Scope

The indicator does not include:

- alerts
- breakout signals
- chart markers
- ORB boxes
- strategy or backtesting logic
- multiple historical sessions
- symbol locking
