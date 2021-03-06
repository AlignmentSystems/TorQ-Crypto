# Data Capture

### Feed Handlers

The feed handler processes collect real time cryptocurrency data though 
RESTful APIs. We have added 5 feeds which collect data from the following 
exchanges:

-    [OKEX](https://www.okex.com/docs/en/) 
-    [DigiFinex](https://docs.digifinex.com/en-ww/v3/) 
-    [Huobi](https://huobiapi.github.io/docs/spot/v1/en/#introduction) 
-    [ZB](https://www.zb.com/api) 
-    [Bluehelix/HBTC](https://github.com/bhexopen/BHEX-OpenApi) 

Each feed collects level 2 order book data for its subscribed symbols at a set 
frequency and limit which is discussed [here](https://aquaqanalytics.github.io/TorQ-Crypto/configuration/). 
After converting the JSON response to a KDB table the following standardisation 
occurs before the data is sent to the ticker plant:

-    Conversion of times to KDB timestamps 
-    Quotes arranged in order of best to worst 
-    Duplicated data will not be sent (i.e quotes that have not changed from last publish)

This diagram summarises the data capture:

![Sym Config](graphics/dataflow.PNG)

### Tables

Each feed publishes data to three tables in the RDB - exchange, exchange_top
and a table specific to its own feed. The exchange table contains a superset
of L2 data collected from all exchanges with exchange_top containing only top-of-book 
data. It is these exchange table which are used in the inbuilt functions to compare
quotes across exchanges and over time. 

    meta exchange
    c           | t f a
    ------------| -----
    time        | p
    sym         | s   g
    exchangeTime| p
    exchange    | s
    bid         | F
    bidSize     | F
    ask         | F
    askSize     | F

    meta exchange_top
    c           | t f a
    ------------| -----
    time        | p
    sym         | s   g
    exchangeTime| p
    exchange    | s
    bid         | f
    bidSize     | f
    ask         | f
    askSize     | f

##### Additional Information:

The HTTP requests that the feed processes send do not have a time out and it is 
possible for these requests to fail on the exchange side for myriad of reasons. In 
such cases a small gap may be seen in the data, typically this will not be more 
than a few minutes.
