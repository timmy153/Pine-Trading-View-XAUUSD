//@version=5
strategy("EMA 9/21 Strategy 15m Clean", shorttitle="EMA921 15m Clean", overlay=true, initial_capital=10000, default_qty_type=strategy.percent_of_equity, default_qty_value=100, commission_type=strategy.commission.percent, commission_value=0.0)
c15FastLen = input(9, "Fast EMA Length")
c15SlowLen = input(21, "Slow EMA Length")
c15AllowLong = input(true, "Allow Long")
c15AllowShort = input(true, "Allow Short")
c15UseTrendFilter = input(true, "Use EMA 200 Trend Filter")
c15TrendLen = input(200, "Trend EMA Length")
c15UseHtfFilter = input(true, "Use 1H EMA 200 Filter")
c15HtfTf = input("60", "Higher Timeframe")
c15HtfLen = input(200, "HTF EMA Length")
c15UseCciFilter = input(true, "Use CCI Filter")
c15CciLen = input(20, "CCI Length")
c15CciLongMin = input(50.0, "Long CCI Minimum")
c15CciShortMax = input(-50.0, "Short CCI Maximum")
c15UseAdxFilter = input(true, "Use ADX Filter")
c15AdxLen = input(14, "ADX Length")
c15AdxSmooth = input(14, "ADX Smoothing")
c15MinAdx = input(18.0, "Minimum ADX")
c15UseVolFilter = input(true, "Use Volume Filter")
c15VolLen = input(20, "Volume SMA Length")
c15VolMult = input(1.0, "Volume Multiplier")
c15CooldownBars = input(5, "Cooldown Bars")
c15StopLossPct = input(1.2, "Stop Loss %")
c15TakeProfitPct = input(3.0, "Take Profit %")
c15UsdStopMove = input(10.0, "SL Alert USD Move")
c15ShowLabels = input(true, "Show Buy/Sell Labels")
c15FastEma = ta.ema(close, c15FastLen)
c15SlowEma = ta.ema(close, c15SlowLen)
c15TrendEma = ta.ema(close, c15TrendLen)
c15Cci = ta.cci(hlc3, c15CciLen)
c15HtfClose = request.security(syminfo.tickerid, c15HtfTf, close, barmerge.gaps_off, barmerge.lookahead_off)
c15HtfEma = request.security(syminfo.tickerid, c15HtfTf, ta.ema(close, c15HtfLen), barmerge.gaps_off, barmerge.lookahead_off)
[c15PlusDi, c15MinusDi, c15Adx] = ta.dmi(c15AdxLen, c15AdxSmooth)
c15VolAvg = ta.sma(volume, c15VolLen)
var int c15LastEntryBar = na
c15CooldownOk = na(c15LastEntryBar) or bar_index - c15LastEntryBar >= c15CooldownBars
c15LongTrendOk = not c15UseTrendFilter or close > c15TrendEma
c15ShortTrendOk = not c15UseTrendFilter or close < c15TrendEma
c15LongHtfOk = not c15UseHtfFilter or c15HtfClose > c15HtfEma
c15ShortHtfOk = not c15UseHtfFilter or c15HtfClose < c15HtfEma
c15LongCciOk = not c15UseCciFilter or c15Cci > c15CciLongMin
c15ShortCciOk = not c15UseCciFilter or c15Cci < c15CciShortMax
c15AdxOk = not c15UseAdxFilter or c15Adx >= c15MinAdx
c15VolOk = not c15UseVolFilter or volume > c15VolAvg * c15VolMult
c15BuyEma200Alert = ta.crossover(c15FastEma, c15SlowEma) and close > c15TrendEma
c15SellEma200Alert = ta.crossunder(c15FastEma, c15SlowEma) and close < c15TrendEma
c15LongSignal = ta.crossover(c15FastEma, c15SlowEma) and c15AllowLong and c15LongTrendOk and c15LongHtfOk and c15LongCciOk and c15AdxOk and c15VolOk and c15CooldownOk and strategy.position_size == 0
c15ShortSignal = ta.crossunder(c15FastEma, c15SlowEma) and c15AllowShort and c15ShortTrendOk and c15ShortHtfOk and c15ShortCciOk and c15AdxOk and c15VolOk and c15CooldownOk and strategy.position_size == 0
c15LongStop = strategy.position_avg_price * (1 - c15StopLossPct / 100)
c15LongTp = strategy.position_avg_price * (1 + c15TakeProfitPct / 100)
c15ShortStop = strategy.position_avg_price * (1 + c15StopLossPct / 100)
c15ShortTp = strategy.position_avg_price * (1 - c15TakeProfitPct / 100)
c15LongUsdStop = strategy.position_avg_price - c15UsdStopMove
c15ShortUsdStop = strategy.position_avg_price + c15UsdStopMove
c15LongSlAlert = strategy.position_size > 0 and (close <= c15LongStop or close <= c15LongUsdStop)
c15ShortSlAlert = strategy.position_size < 0 and (close >= c15ShortStop or close >= c15ShortUsdStop)
c15LongSlLabel = c15LongSlAlert and not c15LongSlAlert[1]
c15ShortSlLabel = c15ShortSlAlert and not c15ShortSlAlert[1]
if c15LongSignal
    strategy.entry("Long", strategy.long, alert_message="EMA921 15m: Long entry")
    c15LastEntryBar := bar_index
if c15ShortSignal
    strategy.entry("Short", strategy.short, alert_message="EMA921 15m: Short entry")
    c15LastEntryBar := bar_index
if strategy.position_size > 0
    strategy.exit("Long Exit", from_entry="Long", stop=c15LongStop, limit=c15LongTp, alert_message="EMA921 15m: Long exit 1.2% SL or 3% TP")
if strategy.position_size < 0
    strategy.exit("Short Exit", from_entry="Short", stop=c15ShortStop, limit=c15ShortTp, alert_message="EMA921 15m: Short exit 1.2% SL or 3% TP")
plot(c15FastEma, "EMA 9", color=color.new(color.aqua, 0), linewidth=2)
plot(c15SlowEma, "EMA 21", color=color.new(color.orange, 0), linewidth=2)
plot(c15TrendEma, "EMA 200", color=color.new(color.gray, 40), linewidth=2)
plot(strategy.position_size > 0 ? c15LongStop : na, "Long Stop", color=color.new(color.red, 0), style=plot.style_linebr)
plot(strategy.position_size > 0 ? c15LongTp : na, "Long TP", color=color.new(color.lime, 0), style=plot.style_linebr)
plot(strategy.position_size < 0 ? c15ShortStop : na, "Short Stop", color=color.new(color.red, 0), style=plot.style_linebr)
plot(strategy.position_size < 0 ? c15ShortTp : na, "Short TP", color=color.new(color.lime, 0), style=plot.style_linebr)
plotshape(c15ShowLabels and c15LongSignal, title="BUY Label", text="BUY", style=shape.labelup, location=location.belowbar, color=color.new(color.green, 0), textcolor=color.white, size=size.small)
plotshape(c15ShowLabels and c15ShortSignal, title="SELL Label", text="SELL", style=shape.labeldown, location=location.abovebar, color=color.new(color.red, 0), textcolor=color.white, size=size.small)
plotshape(c15ShowLabels and c15BuyEma200Alert, title="BUY EMA200 Label", text="BUY\nEMA200", style=shape.labelup, location=location.belowbar, color=color.new(color.lime, 0), textcolor=color.black, size=size.tiny)
plotshape(c15ShowLabels and c15SellEma200Alert, title="SELL EMA200 Label", text="SELL\nEMA200", style=shape.labeldown, location=location.abovebar, color=color.new(color.maroon, 0), textcolor=color.white, size=size.tiny)
plotshape(c15ShowLabels and c15LongSlLabel, title="Long SL Label", text="SL\nLong", style=shape.labeldown, location=location.abovebar, color=color.new(color.red, 0), textcolor=color.white, size=size.small)
plotshape(c15ShowLabels and c15ShortSlLabel, title="Short SL Label", text="SL\nShort", style=shape.labelup, location=location.belowbar, color=color.new(color.red, 0), textcolor=color.white, size=size.small)
alertcondition(c15LongSignal, title="Long Entry Alert", message="EMA921 15m: Long entry")
alertcondition(c15ShortSignal, title="Short Entry Alert", message="EMA921 15m: Short entry")
alertcondition(c15BuyEma200Alert, title="BUY EMA 9/21 Above EMA 200", message="BUY Alert: EMA 9 crossed above EMA 21 and price is above EMA 200")
alertcondition(c15SellEma200Alert, title="SELL EMA 9/21 Below EMA 200", message="SELL Alert: EMA 9 crossed below EMA 21 and price is below EMA 200")
alertcondition(strategy.position_size > 0 and close <= c15LongStop, title="Long Stop Loss Alert", message="EMA921 15m: Long stop loss reached")
alertcondition(strategy.position_size > 0 and close >= c15LongTp, title="Long Take Profit Alert", message="EMA921 15m: Long take profit reached")
alertcondition(strategy.position_size < 0 and close >= c15ShortStop, title="Short Stop Loss Alert", message="EMA921 15m: Short stop loss reached")
alertcondition(strategy.position_size < 0 and close <= c15ShortTp, title="Short Take Profit Alert", message="EMA921 15m: Short take profit reached")
alertcondition(c15LongSlAlert, title="Long SL Alert 1.2% or 10 USD", message="EMA921 15m: Long SL alert, price moved down 1.2% or 10 USD from entry")
alertcondition(c15ShortSlAlert, title="Short SL Alert 1.2% or 10 USD", message="EMA921 15m: Short SL alert, price moved up 1.2% or 10 USD from entry")
