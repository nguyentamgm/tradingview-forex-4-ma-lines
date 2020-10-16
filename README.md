# The base lines for trend analysis in Pine Script
Tradingview only allows 3 indicators for free users. So I made this one for myself to add 4 moving average lines into the chart and it is still counted as 1 indicator only, means I can add 2 more ;)

![Forex base lines screenshot](https://www.tradingview.com/x/9gT0BGfp/)

```javascript
//@version=4
study("Forex base lines", overlay=true)

bb_length = input(title="BB Length", type=input.integer, defval=20, minval=1, maxval=100)
bb_mult = input(title="BB Mult", type=input.float, defval=2, minval=0.01, maxval=20)
bb_source = close
bb_dev = bb_mult * stdev(bb_source, bb_length)
bb_middle = sma(bb_source, bb_length)
bb_upper = bb_middle + bb_dev
bb_lower = bb_middle - bb_dev
p_middle = plot(bb_middle, title="BB Middle", color=#B22222)
p_upper = plot(bb_upper, title="BB Upper", color=#008080, linewidth=1, transp=50)
p_lower = plot(bb_lower, title="BB Lower", color=#008080, linewidth=1, transp=50)

// EMA - fast, slow
ema_xfast = input(title="EMA X-Fast", type=input.integer, defval=6, minval=1, maxval=1000)
ema_fast = input(title="EMA Fast", type=input.integer, defval=18, minval=1, maxval=1000)
ema_slow = input(title="EMA Slow", type=input.integer, defval=50, minval=1, maxval=1000)
sma_xslow = input(title="SMA X-Slow", type=input.integer, defval=200, minval=1, maxval=1000)

ema_xfast_val = ema(close, ema_xfast)
ema_fast_val = ema(close, ema_fast)
ema_slow_val = ema(close, ema_slow)
sma_xslow_val = sma(close, sma_xslow)

p_xfast = plot(ema_xfast_val, title="EMA X-Fast", color=color.green, linewidth=1, transp=50)
p_fast = plot(ema_fast_val, title="EMA Fast", color=color.green, linewidth=3, transp=50)
p_slow = plot(ema_slow_val, title="EMA Slow", color=color.blue, linewidth=4, transp=50)
p_xslow = plot(sma_xslow_val, title="SMA X-Slow", color=color.gray, linewidth=4, transp=50)

// Trend rider strategy
if (ema_fast_val > ema_slow_val and ema_slow_val > sma_xslow_val)
    // If candle[2] is RED
    if(open[1] > close[1] and open[1] >= ema_fast_val)
        // If candle[1] is GREEN
        if(close > open)
            body = close - open
            nose = high - close
            weak = open - low

            // Bullish pinbar or bullish engulfing
            if (nose < body and weak > 2 * body) or (close > open[1])
                label.new(bar_index, low, color=color.green, text="TR", style=label.style_labelup, size=size.tiny)
        
if (ema_fast_val < ema_slow_val and ema_slow_val <= sma_xslow_val)
    // If candle[2] is GREEN
    if(open[1] < close[1] and close[1] < ema_fast_val)
        // If candle[1] is RED
        if(close < open)
            body = open - close
            nose = close - low
            weak = high - open

            // Bearish pinbar or bearish engulfing
            if (nose < body and weak > 2 * body) or (close < open[1])
                label.new(bar_index, high, color=color.red, text="TR", style=label.style_labeldown, size=size.tiny)

// Fill BB ranges
// fill(p_upper, p_lower, color=bb_middle > ema_fast_val ? #008B8B : #8B0000)

// Pinbar arrows
weak_top = high - max(open, close)
weak_bot = min(open, close) - low
body_weigh = close - open
pinbar_down = body_weigh <= 0 and weak_top > 2*weak_bot and weak_top > 3*abs(body_weigh) and high >= highest(high, 10)
pinbar_up = body_weigh >= 0 and weak_bot > 2*weak_top and weak_bot > 3*abs(body_weigh) and low <= lowest(low, 10)
plotshape(pinbar_down, title="Pinbar Down", style=shape.arrowdown, color=color.red)
plotshape(pinbar_up, title="Pinbar Up", style=shape.arrowup, color=color.green, location=location.belowbar)

// Parabolic SAR
psar_start = input(title="PSAR Start", type=input.float, step=0.001, defval=0.02)
psar_increment = input(title="PSAR Increment", type=input.float, step=0.001, defval=0.02)
psar_maximum = input(title="PSAR Maximum", type=input.float, step=0.01, defval=0.2)
psar_value = sar(psar_start, psar_increment, psar_maximum)
psar_dir = psar_value < close ? 1 : -1

psar_color = psar_dir == 1 ? #3388bb : #fdcc02
plot(psar_value, title="PSAR", style=plot.style_circles, linewidth=2, color=psar_color, transp=0)
```
