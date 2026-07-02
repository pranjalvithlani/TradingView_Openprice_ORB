# SPX Open + ORB

A focused TradingView Pine Script v6 indicator for SPX-style intraday trading.

It plots only:

- the 9:30 AM New York regular-session open
- the current-day ORB high and low

## Install

1. Open TradingView.
2. Open the Pine Editor.
3. Paste the contents of `indicators/spx-open-orb.pine`.
4. Click **Add to chart**.

## Defaults

- Session timezone: `America/New_York`
- Regular session open: 9:30 AM ET
- Regular session close: 4:00 PM ET
- ORB duration: 15 minutes
- Open line: yellow
- ORB high line: green
- ORB low line: red
- Line width: 2

## Behavior

The indicator is optimized for SPX but can run on other symbols. It shows only the latest trading day loaded on the chart.

The script hides itself on daily, weekly, and monthly charts. It also hides when the chart timeframe is greater than the selected ORB duration or when the timeframe does not divide the ORB duration cleanly. This keeps ORB levels from accidentally including price action after the opening range window.

For example, a 15-minute ORB works on compatible seconds charts, 1-minute, 3-minute, 5-minute, and 15-minute charts. It hides on 10-minute and 30-minute charts.

## Manual Validation

Use these checks in TradingView:

- On SPX 1-minute, 5-minute, and 15-minute charts, the open line appears after 9:30 AM ET.
- During the ORB window, ORB high and ORB low update live.
- After the ORB window ends, ORB high and ORB low stop changing.
- Lines project to 4:00 PM ET.
- Labels move with the latest regular-session bar and stop at 4:00 PM ET.
- Compatible seconds charts show the same behavior.
- A 10-minute chart hides when ORB duration is 15 minutes.
- A 30-minute chart hides when ORB duration is 15 minutes.
- Daily, weekly, and monthly charts hide.
- Loading a new trading day clears prior levels.

## Scope

This indicator intentionally does not include alerts, breakout markers, ORB boxes, strategy entries, backtesting, symbol locking, or historical-session plotting.
