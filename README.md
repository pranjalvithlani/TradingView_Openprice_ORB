# OpenPrice + ORB

A focused TradingView Pine Script v6 indicator for intraday regular-session levels.

It plots only:

- the 9:30 AM New York regular-session OpenPrice
- the current-day ORB high and low

## Install

1. Open TradingView.
2. Open the Pine Editor.
3. Paste the contents of `indicators/open-orb.pine`.
4. Click **Add to chart**.

## Defaults

- Session timezone: `America/New_York`
- Regular session open: 9:30 AM ET
- Regular session close: 4:00 PM ET
- Show OpenPrice: on
- Show ORB: on
- ORB duration: 15 minutes
- OpenPrice line: yellow
- ORB high line: green
- ORB low line: red
- Line width: 2

## Behavior

The script hides its levels on daily, weekly, and monthly charts. It shows OpenPrice and ORB levels on intraday chart intervals, including larger and custom intervals such as 4-hour, 7-minute, and 38-minute charts.

Levels are calculated from 1-minute data so the ORB window is not limited by the visible chart timeframe.

Lines run from the 9:30 AM ET regular-session open to the 4:00 PM ET regular-session close. Labels sit exactly two chart bars to the right of the latest bar and use left-aligned text for breathing room.

Use **Show OpenPrice** and **Show ORB** in the indicator settings to hide either group of levels.

## Manual Validation

Use these checks in TradingView:

- On 1-minute, 5-minute, 38-minute, and 4-hour charts, OpenPrice appears after 9:30 AM ET.
- During the ORB window, ORB high and ORB low update live.
- After the ORB window ends, ORB high and ORB low stop changing.
- Lines extend from 9:30 AM ET to 4:00 PM ET.
- Labels sit two bars ahead of the latest chart bar.
- Turning off **Show OpenPrice** hides the OpenPrice line and label.
- Turning off **Show ORB** hides ORB high and ORB low lines and labels.
- Daily, weekly, and monthly charts hide.
- Loading a new trading day clears prior levels.

## Scope

This indicator intentionally does not include alerts, breakout markers, ORB boxes, strategy entries, backtesting, symbol locking, or historical-session plotting.
