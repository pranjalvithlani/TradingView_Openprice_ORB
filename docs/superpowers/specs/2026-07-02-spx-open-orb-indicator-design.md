# SPX Open and ORB Indicator Design

## Goal

Build a TradingView Pine Script indicator optimized for SPX intraday trading that plots only two concepts:

- the 9:30 AM New York regular-session open price
- the opening range breakout (ORB) high and low

The indicator is symbol-agnostic, but its defaults and session logic are designed for SPX-style regular trading hours.

## Recommended Approach

Use a single Pine Script v6 indicator file with TradingView drawing objects (`line` and `label`). This approach supports projected horizontal lines through the session close while allowing labels to move bar by bar with the latest active session bar.

Rejected alternatives:

- `plot()`-only series: simpler, but weaker for projected-to-close lines and moving labels.
- Expanded ORB toolkit: unnecessary for this project because alerts, breakout markers, boxes, and strategy logic are out of scope.

## Core Behavior

The indicator runs on all intraday charts, including seconds charts and intervals that do not divide the ORB duration cleanly. It hides on daily, weekly, and monthly charts.

All session logic uses `America/New_York` time:

- regular session open: 9:30 AM ET
- ORB end: 9:30 AM ET plus the configured ORB duration
- regular session close: 4:00 PM ET

At the first valid regular-session bar at or after 9:30 AM ET, the script captures that bar's open as the session open price. It draws a horizontal open line from the open bar to 4:00 PM ET and shows a label with the open price when `Show Open Price` is enabled.

During the ORB window, the script updates the ORB high and ORB low live. After the ORB window ends, those values lock for the rest of the session. ORB high and low lines also extend to 4:00 PM ET, with labels for each level, when `Show ORB` is enabled.

Only the current trading day is shown. Prior session drawings are deleted or reused when a new session begins.

## Inputs and Styling

Settings stay minimal:

- `ORB Duration Minutes`: integer input, default `15`, minimum `1`
- `Show Open Price`: boolean input, default `true`
- `Show ORB`: boolean input, default `true`
- `Open Line Color`: default bright neutral/yellow
- `ORB High Color`: default green
- `ORB Low Color`: default red
- `Line Width`: default `2`

No alerts, breakout markers, historical-session count, boxes, trend filters, tables, or symbol restrictions are included.

## Drawing Behavior

Each active session can have exactly three horizontal lines and three labels:

- `Open`
- `ORB High`
- `ORB Low`

Lines project immediately to 4:00 PM ET. Labels sit two bars to the right of the latest active session bar, move forward bar by bar, and show the current price text while levels are forming. Once a value locks, its label keeps displaying the locked price. Labels do not move beyond the regular session close.

## Data Flow and Edge Cases

Before 9:30 AM ET, the current day has no plotted levels.

At or after 9:30 AM ET, the script captures the first valid session bar open as the session open price.

During the ORB window, the ORB high is the highest high seen during the window and the ORB low is the lowest low seen during the window.

After the ORB window, ORB high and low stop changing. The open price never changes after capture.

After 4:00 PM ET, the completed current-day levels remain visible when viewing that date, but labels stop at the close.

Invalid chart contexts stay quiet:

- daily, weekly, and monthly charts hide the indicator

## Validation Plan

Manual TradingView validation is sufficient for this focused Pine indicator:

- SPX 1-minute, 2-minute, 5-minute, 10-minute, and 15-minute charts show the open, live ORB, then locked ORB.
- Seconds charts show the same behavior.
- Daily charts hide.
- A new trading day clears prior levels and shows only the current day.
- Labels move two bars ahead of the latest session bar and stop after 4:00 PM ET.
- Settings can independently hide open-price levels or ORB levels.

## Out of Scope

The first version does not include:

- alerts
- breakout signals
- chart markers
- ORB boxes
- strategy or backtesting logic
- multiple historical sessions
- symbol locking
- external test harnesses
