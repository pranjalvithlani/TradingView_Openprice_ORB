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
- Show open price: on
- Show ORB: on
- ORB duration: 15 minutes
- Open line: yellow
- ORB high line: green
- ORB low line: red
- Line width: 2

## Behavior

The indicator is optimized for SPX but can run on other symbols. It shows only the latest trading day loaded on the chart.

The script hides its levels on daily, weekly, and monthly charts. It shows the open price and ORB levels on all intraday chart intervals, including intervals that do not divide the ORB duration cleanly.

Labels sit two bars to the right of the latest regular-session bar for breathing room and stop at 4:00 PM ET.

Use **Show Open Price** and **Show ORB** in the indicator settings to hide either group of levels.

## Manual Validation

Use these checks in TradingView:

- On SPX 1-minute, 5-minute, and 15-minute charts, the open line appears after 9:30 AM ET.
- During the ORB window, ORB high and ORB low update live.
- After the ORB window ends, ORB high and ORB low stop changing.
- Lines project to 4:00 PM ET.
- Labels move two bars ahead of the latest regular-session bar and stop at 4:00 PM ET.
- Seconds charts and non-divisible intraday charts such as 2-minute and 10-minute still show levels.
- Turning off **Show Open Price** hides the open line and label.
- Turning off **Show ORB** hides ORB high and ORB low lines and labels.
- Daily, weekly, and monthly charts hide.
- Loading a new trading day clears prior levels.

## Scope

This indicator intentionally does not include alerts, breakout markers, ORB boxes, strategy entries, backtesting, symbol locking, or historical-session plotting.
