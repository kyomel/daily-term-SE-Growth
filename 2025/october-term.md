day - 1

## Delta Encoding

### Definition:
Delta Encoding is a data compression technique that stores only the differences (deltas) between consecutive values instead of storing the absolute values. This is highly effective when data changes gradually or has patterns, as differences are typically smaller numbers that require less storage space.

**Key Properties:**
- Space efficient: Differences are usually smaller than original values
- Pattern exploitation: Works best with sequential or time-series data
- Simple concept: Store first value + sequence of changes
- Reversible: Can reconstruct original data perfectly

**How It Works:**
- Store the first value as-is (base value)
- For each subsequent value, store the difference from the previous value
- To decode: start with base value and apply deltas sequentially

### Example:
Stock Prices
```
Original prices: [$100.50, $100.75, $100.60, $101.10, $100.90]
Delta encoded:   [$100.50, +$0.25, -$0.15, +$0.50, -$0.20]

Benefits:
- Smaller numbers to store/transmit
- Easier to spot trends (mostly small changes)
- Better compression when combined with other techniques
```

---
