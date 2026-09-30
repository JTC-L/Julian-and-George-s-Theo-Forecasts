# Order book replay and theo model

Interactive replay of order book snapshots and trades. One tab per timestamp shows the order book on the left and the trading history on the right, with a theo, a prediction check and a simple trading simulation underneath.

- `index.html` is dataset 1 (`vwap_workshop_books.csv`, `vwap_workshop_trades.csv`).
- `set2.html` is dataset 2 (`vwap_workshop_books_set2.csv`, `vwap_workshop_trades_set2.csv`).

Each page is a single self-contained HTML file with its data embedded. Open it in a browser, no install needed.

## The model as currently implemented

**Level weight.** size x 2^(-% distance from mid) x e^(-(k x age)^2), with k = 1/(40 x sqrt 2), so the time weight's inflection point is at 40 s. Age is the `age_s` value in the book file.

**Book theo.** Every price is multiplied by the total weight of the opposite side. With bids 8.30, 8.35, 8.40 and asks 8.45, 8.50, 8.55, each ask price is multiplied by the sum of the three bid weights, and each bid price by the sum of the three ask weights. The theo is the sum of those products divided by the sum of the multipliers used.

**Last trade in the theo (switchable).** The most recent trade printed by the snapshot is counted as one more order on the aggressor's side (a buy with the bids, a sell with the asks). It is treated as 0 s old, and its price is midway between the traded price and the next level in the direction they traded, read from that snapshot's book (next ask above for a buy, next bid below for a sell; one 0.05 tick further if the visible book has no such level). It is not shown in the order book ladder.

**Prediction check.** Theo at Tn-k against the mid at Tn, for k = 1, 2, 3, compared with the plain mid change (mid at Tn-k minus mid at Tn). Mean absolute error, RMSE and how often the theo was closer are shown.

**Trigger and PnL.** At each snapshot, buy ask levels priced below the theo and sell into bid levels priced above it, from the best price outward.
- At most 100 shares per snapshot.
- When entering or adding to a position, at most the smaller of 100 shares and 20% of the shares showing on the side traded.
- Position limit of +/-300. At the limit, if the theo is between the best bid and ask, trade up to 100 shares at the touch to reduce.
- PnL is cash + position x mark, marked to the mid and to the theo.

## Caveats

- The trade files give only the interval between two snapshots, so trades are spaced evenly inside their interval in sequence order. That spacing affects where trades sit on the chart.
- Each snapshot is treated as fully tradable at displayed sizes, but resting orders persist between snapshots, so the same liquidity can be counted more than once in the simulation.
- No fees and no exit rule beyond the limits above.
- Pages load the IBM Plex Sans font from Google Fonts and fall back to system fonts if it is unavailable.
