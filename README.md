# EonTrading-Schema


- Propose schema for storing time series into mongodb

    1. Document Structure (Bucket) (Not recommended)

        - A typical document in a time series collection groups data by stock and interval (e.g., 1 minute, 1 hour).

        - ```{
            "_id": ObjectId("..."),
            "stock_symbol": "AAPL",
            "exchange": "NASDAQ",
            "interval": "1m",  // minute-level bars
            "data": [
                {
                "timestamp": ISODate("2024-06-20T14:31:00Z"),
                "open": 190.00,
                "high": 191.00,
                "low": 189.50,
                "close": 190.75,
                "volume": 1200
                },
                {
                "timestamp": ISODate("2024-06-20T14:32:00Z"),
                "open": 190.75,
                "high": 191.25,
                "low": 190.50,
                "close": 191.00,
                "volume": 900
                }
                // ...more bars
            ]
            }
        ```
        - | Disadvantage | Explanation |
            |-----------------------------|-------------|
            | Updates Are Expensive       | Modifying an individual data point within a bucket requires rewriting the whole document or complex update logic. If you frequently update/correct historical data, this is a pain. |
            | Document Size Limit         | MongoDB documents have a size limit (typically 16 MB). If a bucket (array) grows too large, you hit this limit, requiring carefully managed bucket sizes. |
            | Complexity in Bucketing     | You must decide how to split buckets (by time, number of entries, etc.), which can add maintenance overhead or lead to uneven data distribution. |
            | Query and Aggregation Complexity | Filtering for a specific timestamp or value inside buckets can be less direct (requires array element matching), potentially less efficient than querying top-level fields. |
            | Insert Order Matters        | If you insert out-of-order data or late-arriving timestamps, you need logic to find and update the correct bucket, which can get complicated. |
            | Not Ideal for Real-Time Updates | If you need ultra-low latency inserts (such as for live feeds), bucketed writes may be slower than single-document inserts. |
            | Difficult Deletion of Individual Points | Removing just one data point from a bucket involves rewriting the entire document. |




    2. One Document Per Datapoint (Recommended)

    - ```
    {
    "_id": ObjectId("..."),
    "stock_symbol": "AAPL",
    "exchange": "NASDAQ",
    "timestamp": ISODate("2024-06-20T14:31:00Z"),
    "open": 190.00,
    "high": 191.00,
    "low": 189.50,
    "close": 190.75,
    "volume": 1200
    }
        ```


- date field
    - Use ISODate for timestamp fields to leverage MongoDB's date querying capabilities.
    - Example: `"timestamp": ISODate("2024-06-20T14:31:00Z")`


