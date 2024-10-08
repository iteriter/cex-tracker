## installation
```docker build & docker compose up -d```

## usage
after starting the docker containers, api service should be available at
```http://127.0.0.1:3000```

below are the supported endpoints for the api service

```http://127.0.0.1:3000/crypto/exchanges/``` -- list all available exchanges. note: 'average' is a synthetic exchange aggregating prices from all exchanges
```http://127.0.0.1:3000/crypto/price/``` -- list prices for all pairs, on all exchanges, including the 'average' exchange
```http://127.0.0.1:3000/crypto/price?exchange=kraken``` -- list prices for all pairs on the given exchange, in this case 'kraken'
```http://127.0.0.1:3000/crypto/price?pair=BTCUSDT``` -- list prices for the given pair on all exchanges, including the 'average' exchange
```http://127.0.0.1:3000/crypto/price?pair=BTCUSDT&exchange=kraken``` -- get the price for the specified pair on the given exchange

all pair names are normalised across exchanges into the format of [BASE][QUOTE], e.g. BTCUSDT, ETHUSDT, etc.

## architecture
The Django app and the Exchanges parser are running as separate instances,
communicating via the redis app. As pairs data becomes available on the websockets,
it is pushed to the redis, from where the Django app can always read the latest data
upon the user request.


## potential issues
- adding new tickers to streams might take a while if there's not much data in the stream
  in practice this should be negligible as the websocket might have anywhere above 100 streams
  producing events every second


## considerations
tickers are updated less frequently than the live trading data.
in order to receive the most up to date best ask/bid, we might want to use order Book data or/and track completed Trade events


## potential improvements
- throttling with an exponential backoff should probably be added to individual connections in order to avoid
exceeding the exchange's rate limits
- new websocket connections might need to be added from time to time
to handle new pairs becoming live on exchange
- pairs that went off from trading should be removed from the list of tracked pairs