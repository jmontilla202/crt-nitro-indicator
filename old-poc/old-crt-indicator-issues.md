| Issue | Consequence |
| --- | --- |
| Developing HTF values are requested without a confirmed-bar offset | CRT conditions and bias can change before the higher-timeframe candle closes |
| New C2 detection uses a false-to-true transition | Consecutive valid C2 candles can be missed |
| Expiration counts chart bars, not the selected setup timeframe | “C3 only” means different things on different chart timeframes |
| Break, pullback, and re-break can all pass on one candle | The script assumes a price sequence that OHLC data cannot establish |
| TP is checked before SL, including the entry candle | Ambiguous candles can be reported as wins |
| Chop restrictions appear in the table but do not consistently gate entries | The displayed verdict and actual signal logic can disagree |
| Stop placement is calculated backward from the desired reward/risk | The stop can lack structural meaning; invalid target geometry is not rejected |