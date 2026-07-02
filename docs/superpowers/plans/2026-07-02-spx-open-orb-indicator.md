# SPX Open ORB Indicator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a focused TradingView Pine Script indicator that plots the 9:30 AM New York open and current-day ORB high/low levels.

**Architecture:** One Pine Script v6 overlay indicator owns all state, drawings, and inputs. It uses New York timestamps, three `line` objects, and three `label` objects; old objects are deleted or reused so only the latest trading day remains visible.

**Tech Stack:** TradingView Pine Script v6, TradingView drawing objects (`line`, `label`), Markdown README.

---

## File Structure

- Create `indicators/spx-open-orb.pine`: the complete indicator.
- Create `README.md`: install instructions and manual validation checklist.

No packages, build tooling, external test harness, alerts, strategies, or extra helper files.

## References

- Approved design spec: `docs/superpowers/specs/2026-07-02-spx-open-orb-indicator-design.md`
- TradingView Pine Script v6 user manual: https://www.tradingview.com/pine-script-docs/
- TradingView lines and boxes: https://www.tradingview.com/pine-script-docs/visuals/lines-and-boxes/
- TradingView text and shapes: https://www.tradingview.com/pine-script-docs/visuals/text-and-shapes/
- TradingView timeframes: https://www.tradingview.com/pine-script-docs/concepts/timeframes/
- TradingView time: https://www.tradingview.com/pine-script-docs/concepts/time/
- TradingView inputs: https://www.tradingview.com/pine-script-docs/concepts/inputs/

### Task 1: Create the Pine Indicator

**Files:**
- Create: `indicators/spx-open-orb.pine`

- [ ] **Step 1: Run the pre-implementation file check**

Run:

```bash
test -f indicators/spx-open-orb.pine
```

Expected: command fails because the indicator file does not exist yet.

- [ ] **Step 2: Create the indicator file**

Create `indicators/spx-open-orb.pine` with this content:

```pine
//@version=6
indicator("SPX Open + ORB", "SPX Open ORB", overlay = true, max_lines_count = 10, max_labels_count = 10)

const string NY_TZ = "America/New_York"
const int SESSION_OPEN_HOUR = 9
const int SESSION_OPEN_MINUTE = 30
const int SESSION_CLOSE_HOUR = 16
const int SESSION_CLOSE_MINUTE = 0

int orbMinutesInput = input.int(15, "ORB Duration Minutes", minval = 1)
color openColorInput = input.color(color.yellow, "Open Line Color")
color orbHighColorInput = input.color(color.lime, "ORB High Color")
color orbLowColorInput = input.color(color.red, "ORB Low Color")
int lineWidthInput = input.int(2, "Line Width", minval = 1, maxval = 5)

priceText(string levelName, float price) =>
    levelName + " " + str.tostring(price, format.mintick)

upsertLine(line lineId, int startTime, int endTime, float price, color lineColor, int lineWidth) =>
    line result = lineId
    if na(result)
        result := line.new(startTime, price, endTime, price, xloc = xloc.bar_time, extend = extend.none, color = lineColor, width = lineWidth)
    else
        line.set_xy1(result, startTime, price)
        line.set_xy2(result, endTime, price)
        line.set_color(result, lineColor)
        line.set_width(result, lineWidth)
    result

upsertLabel(label labelId, int labelTime, float price, string labelText, color labelColor) =>
    label result = labelId
    if na(result)
        result := label.new(labelTime, price, labelText, xloc = xloc.bar_time, style = label.style_none, textcolor = labelColor, size = size.small)
    else
        label.set_x(result, labelTime)
        label.set_y(result, price)
        label.set_text(result, labelText)
        label.set_textcolor(result, labelColor)
    result

int nyYear = year(time, NY_TZ)
int nyMonth = month(time, NY_TZ)
int nyDay = dayofmonth(time, NY_TZ)
int tradingDay = nyYear * 10000 + nyMonth * 100 + nyDay

int sessionOpenTime = timestamp(NY_TZ, nyYear, nyMonth, nyDay, SESSION_OPEN_HOUR, SESSION_OPEN_MINUTE, 0)
int orbEndTime = sessionOpenTime + orbMinutesInput * 60 * 1000
int sessionCloseTime = timestamp(NY_TZ, nyYear, nyMonth, nyDay, SESSION_CLOSE_HOUR, SESSION_CLOSE_MINUTE, 0)

float chartSeconds = timeframe.in_seconds()
bool validChart = timeframe.isintraday and chartSeconds <= orbMinutesInput * 60
bool inSession = validChart and time >= sessionOpenTime and time < sessionCloseTime
bool inOrbWindow = inSession and time >= sessionOpenTime and time < orbEndTime
bool reachedSession = validChart and time >= sessionOpenTime

var int activeTradingDay = na
var float sessionOpenPrice = na
var float orbHigh = na
var float orbLow = na
var int openStartTime = na

var line openLine = na
var line orbHighLine = na
var line orbLowLine = na

var label openLabel = na
var label orbHighLabel = na
var label orbLowLabel = na

bool resetForInvalidChart = not validChart and (not na(openLine) or not na(orbHighLine) or not na(orbLowLine) or not na(openLabel) or not na(orbHighLabel) or not na(orbLowLabel) or not na(activeTradingDay))
bool resetForNewDay = validChart and (na(activeTradingDay) or tradingDay != activeTradingDay)

if resetForInvalidChart or resetForNewDay
    if not na(openLine)
        line.delete(openLine)
    if not na(orbHighLine)
        line.delete(orbHighLine)
    if not na(orbLowLine)
        line.delete(orbLowLine)
    if not na(openLabel)
        label.delete(openLabel)
    if not na(orbHighLabel)
        label.delete(orbHighLabel)
    if not na(orbLowLabel)
        label.delete(orbLowLabel)

    openLine := na
    orbHighLine := na
    orbLowLine := na
    openLabel := na
    orbHighLabel := na
    orbLowLabel := na
    sessionOpenPrice := na
    orbHigh := na
    orbLow := na
    openStartTime := na
    activeTradingDay := validChart ? tradingDay : na

if inSession and na(sessionOpenPrice)
    sessionOpenPrice := open
    openStartTime := time

if inOrbWindow
    orbHigh := na(orbHigh) ? high : math.max(orbHigh, high)
    orbLow := na(orbLow) ? low : math.min(orbLow, low)

if reachedSession
    int labelTime = time < sessionCloseTime ? time : sessionCloseTime

    if not na(sessionOpenPrice)
        openLine := upsertLine(openLine, openStartTime, sessionCloseTime, sessionOpenPrice, openColorInput, lineWidthInput)
        openLabel := upsertLabel(openLabel, labelTime, sessionOpenPrice, priceText("Open", sessionOpenPrice), openColorInput)

    if not na(orbHigh) and not na(orbLow)
        orbHighLine := upsertLine(orbHighLine, sessionOpenTime, sessionCloseTime, orbHigh, orbHighColorInput, lineWidthInput)
        orbLowLine := upsertLine(orbLowLine, sessionOpenTime, sessionCloseTime, orbLow, orbLowColorInput, lineWidthInput)
        orbHighLabel := upsertLabel(orbHighLabel, labelTime, orbHigh, priceText("ORB High", orbHigh), orbHighColorInput)
        orbLowLabel := upsertLabel(orbLowLabel, labelTime, orbLow, priceText("ORB Low", orbLow), orbLowColorInput)
```

- [ ] **Step 3: Run the file-exists check**

Run:

```bash
test -f indicators/spx-open-orb.pine
```

Expected: command succeeds.

- [ ] **Step 4: Compile in TradingView**

Open TradingView Pine Editor, paste `indicators/spx-open-orb.pine`, and click **Add to chart**.

Expected: the script compiles without errors and appears as `SPX Open ORB` on the chart.

- [ ] **Step 5: Commit**

```bash
git add indicators/spx-open-orb.pine
git commit -m "feat: add SPX open ORB indicator"
```

### Task 2: Add User Documentation

**Files:**
- Create: `README.md`

- [ ] **Step 1: Run the pre-documentation file check**

Run:

```bash
test -f README.md
```

Expected: command fails because the README does not exist yet.

- [ ] **Step 2: Create the README**

Create `README.md` with this content:

```markdown
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

The script hides itself on daily, weekly, and monthly charts. It also hides when the chart timeframe is greater than the selected ORB duration. For example, a 15-minute ORB works on seconds, 1-minute, 5-minute, and 15-minute charts, but hides on a 30-minute chart.

## Manual Validation

Use these checks in TradingView:

- On SPX 1-minute, 5-minute, and 15-minute charts, the open line appears after 9:30 AM ET.
- During the ORB window, ORB high and ORB low update live.
- After the ORB window ends, ORB high and ORB low stop changing.
- Lines project to 4:00 PM ET.
- Labels move with the latest regular-session bar and stop at 4:00 PM ET.
- Seconds charts show the same behavior.
- A 30-minute chart hides when ORB duration is 15 minutes.
- Daily charts hide.
- Loading a new trading day clears prior levels.

## Scope

This indicator intentionally does not include alerts, breakout markers, ORB boxes, strategy entries, backtesting, symbol locking, or historical-session plotting.
```

- [ ] **Step 3: Run the README check**

Run:

```bash
test -f README.md
```

Expected: command succeeds.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add indicator usage notes"
```

### Task 3: Final Validation

**Files:**
- Read: `indicators/spx-open-orb.pine`
- Read: `README.md`
- Read: `docs/superpowers/specs/2026-07-02-spx-open-orb-indicator-design.md`

- [ ] **Step 1: Verify expected files exist**

Run:

```bash
test -f indicators/spx-open-orb.pine && test -f README.md && test -f docs/superpowers/specs/2026-07-02-spx-open-orb-indicator-design.md
```

Expected: command succeeds.

- [ ] **Step 2: Verify no scoped-out features were added**

Run:

```bash
rg -n "alert|strategy\\(|box\\.new|plotshape|plotarrow|historical|backtest|entry" indicators README.md
```

Expected: no matches in `indicators/spx-open-orb.pine`; README matches are acceptable only inside the Scope section.

- [ ] **Step 3: Verify the approved session constants are present**

Run:

```bash
rg -n "America/New_York|SESSION_OPEN_HOUR = 9|SESSION_OPEN_MINUTE = 30|SESSION_CLOSE_HOUR = 16|ORB Duration Minutes" indicators/spx-open-orb.pine
```

Expected: matches for all required session and input values.

- [ ] **Step 4: Run TradingView manual checks**

In TradingView, validate:

```text
SPX 1m: open line appears at 9:30 ET; ORB high/low update live; ORB locks after 15 minutes.
SPX 5m: same behavior with fewer bars.
SPX 15m: ORB captures the opening 15-minute bar and locks after it.
SPX seconds chart: same behavior on available seconds timeframe.
SPX 30m with 15-minute ORB: indicator hides.
SPX daily: indicator hides.
After 4:00 PM ET: labels stay at the session close and do not move farther right.
Next loaded trading day: prior levels are gone; only the latest trading day is visible.
```

Expected: every check passes.

- [ ] **Step 5: Commit any validation-only wording fixes**

If README wording is adjusted during validation, commit only that documentation change:

```bash
git add README.md
git commit -m "docs: clarify validation notes"
```

Expected: skip this commit when no README wording changes are made.
