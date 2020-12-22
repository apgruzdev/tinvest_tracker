```python
from constants import TINKOFF_INVESTM_TOKEN, START_DATE
import tinvest
from tinvest import schemas
```

```python
from datetime import datetime, timedelta
import pandas as pd
```

```python
client = tinvest.SyncClient(TINKOFF_INVESTM_TOKEN)
client.get_accounts().payload.accounts  # use TinkoffIis
```

```python
# usd_course
# currencies = client.get_market_currencies()
# instruments = currencies.payload.instruments
# instrument = instruments[0]
# instrument
```

```python
def get_usd_day_price(client, use_day) -> float:
    usd_candle = client.get_market_candles(figi='BBG0013HGFT4', 
                                           from_=use_day-timedelta(days=2), to=use_day,
                                           interval=schemas.CandleResolution.day).payload.candles
    usd_course = float(usd_candle[0].c)
    return usd_course
```

```python
usd_course = get_usd_day_price(client, use_day=datetime.now())
usd_course
```

```python
portfolio = client.get_portfolio(broker_account_id='2002930614')
positions = portfolio.payload.positions
positions[0]
```

```python
operations = client.get_operations(broker_account_id='2002930614', from_=START_DATE, to=datetime.now())
operations = operations.payload.operations
operations[0]
```

```python
portfolio_sum = 0.
for position in positions:
    current_ticker_cost = float(position.balance * position.average_position_price.value + position.expected_yield.value)
    if position.average_position_price.currency.name == "usd":
        current_ticker_cost *= usd_course
    portfolio_sum += current_ticker_cost
portfolio_sum
```

```python
sum_pay_in = 0.
for operation in operations:
    if operation.operation_type.value == "PayIn":
        sum_pay_in += float(operation.payment)
sum_pay_in
```

```python
rur_profit = round((portfolio_sum - sum_pay_in) / sum_pay_in * 100, 2)
f'Profit in RUR: {rur_profit}% [{int((portfolio_sum - sum_pay_in))} RUR]' 
```

----

```python
sum_pay_in_usd = 0.
for operation in operations:
    if operation.operation_type.value == "PayIn":
        usd_course = get_usd_day_price(client, use_day=operation.date)
        sum_pay_in_usd += float(operation.payment) / usd_course
sum_pay_in_usd
```

```python
usd_course = get_usd_day_price(client, use_day=datetime.now())
usd_profit = round((portfolio_sum / usd_course - sum_pay_in_usd) / sum_pay_in_usd * 100, 2)
f'Profit in USD: {usd_profit}% [{int((portfolio_sum / usd_course - sum_pay_in_usd))} USD]' 
```

```python
invest_vs_usd = round(rur_profit - (sum_pay_in_usd * usd_course - sum_pay_in) / sum_pay_in * 100, 2)
f'+/- investing vs. USD: {invest_vs_usd}%'
```

----

```python
days = (datetime.now() - min([operation.date for operation in operations if operation.operation_type.value == "PayIn"]).replace(tzinfo=None)).days
```

```python
years = days / 365.
```

```python
f'Profit in RUR: {round(rur_profit / years, 2)} %/Y' 
```

```python
f'Profit in USD: {round(usd_profit / years, 2)} %/Y' 
```
