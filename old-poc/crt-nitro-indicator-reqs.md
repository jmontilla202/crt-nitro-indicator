CRT Nitro Indicator Requirements

- log chop level at 5m, 15m, 30m, and 60m. Determine if chop is improving, worsening or keeping constant. Log at minute intervals that make sense, adjusted, use sensible default. Indicator should only work on these timeframes. Other timeframes should display 'Use a 15m, 30m or 60m timeframe'
- Only displayed data on for current timeframe on chart 
- Log / display shorts and longs setup count seperately, each get 2 trades max
- Only count trades towards daily max if chop level is 'B' or better.
- Plot trade entry, take profit and stop-loss levels on chart if chop level is 'B' or better 
- Make these changes incrementaly so as to split updates up into management chunks units
- If it makes sense, start from scratch, add 
- Use semantic versioning of patch levels and minor versions. Make current version pre-release, unstable.
- Add version to title, include version as comment in code as well
- Lets get these items ironed out first unless you see something that does not align with trading index futures such as ES, NQ, YM or RTY on prop accounts of popular prop firms such as Lucid and Take Profit Trader. Eval specs: Size = 50k, PT = 3k, DD = 2k, EOD, DLL, No subscription / monthly fee.