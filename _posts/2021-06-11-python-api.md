---
title: "[Python] 바이낸스 API를 이용해 분단위 비트코인 가격데이터 수집"
layout: post
date: '2021-06-11 18:55:22'
author: Edward Park
categories:
- Stock
tags:
- Bitcoin
cover: "/assets/instacode.png"
---

## Abstract
주식의 경우 **일 단위** 가격데이터(Open, High, Low, Close, Volume 등)는 한국거래소 등에서 쉽게 구할 수 있다. 하지만 시간, 분 단위 데이터는 증권사 API를 통해서 요청받아야하며, 이마저도 복잡한 프로세스를 거쳐야한다. <br>
하지만 비트코인데이터의 경우 일단위 뿐만 아니라 시간, 분 단위 데이터도 업비트, Binance등의 사이트에서 따로 인증키 없이 API를 통해 쉽게 데이터를 받을 수 있다. 그래서 이번 포스트에서는 **Binance**에서 제공하는 API를 통해 여러 암호화폐 데이터를 수집하고, 엑셀 파일로 저장하는 법을 알아보자.

## Binance site
[Binance API docs](https://binance-docs.github.io/apidocs/spot/en/)<br>
위 사이트에서 Binance에서 제공하는 API를 확인할 수 있다. 엄청나게 다양한 정보를 제공하지만, 나름 정리가 잘 되어있지만, 개인적으로는 암호화폐에 대한 도메인 지식도 부족하고 영어도 능숙하지 못한 편이라 원하는 정보를 찾는데 꽤나 많은 시간을 썼었다.
<img src="/blog/post_images/binance/binance_api_1.png" title="binance_api">
스크롤을 내리다보면 `Klines/Candlestick Data`라는 항목을 찾을 수 있다. 그리고 GET 요청을 통해 해당 url에 적절한 parameter를 보내 요청하면 아래와 같은 Response를 얻을 수 있다.
<img src="/blog/post_images/binance/binance_api_2.png" title="binance_api">
여기서 `asset volume`, `taker` 등 생소한 용어들이 등장하는데, asset volume은 거래량의 단위를 다르게한것이고, taker은 시장가로 거래한 사람들(?) 정도로 생각하면 될 것 같다(정확하지는 않음).<br>

## Code
### Import libraries
```Python
import requests
from datetime import datetime
import time
import pandas as pd
from tqdm import tqdm
```
### 심볼(암호화폐 코드) 가져오기
```Python
result = requests.get('https://api.binance.com/api/v3/ticker/price')
js = result.json()
symbols = [x['symbol'] for x in js]
symbols_usdt = [x for x in symbols if 'USDT' in x]  # 끝이 USDT로 끝나는 심볼들, ['BTCUSDT', 'ETHUSDT', ...]
```
암호화폐 가격을 요청하기위해서는 필수로 해당 암호화폐의 Symbol을 알아야한다. 예를들어 **비트코인을 달러로 거래한 데이터**에 해당되는 심볼은 `BTCUSDT` 이다. <br>
보통 바이낸스 거래소에서는 달러단위로 거래를 많이하므로 USDT로 끝나는 심볼들(2021.06.10 기준 310개)을 사용해 데이터를 수집한다.
### 날짜(구간)와 심볼을 입력하면 DataFrame으로 반환
```Python
COLUMNS = ['Open_time', 'Open', 'High', 'Low', 'Close', 'Volume', 'Close_time', 'quote_av', 'trades', 
                   'tb_base_av', 'tb_quote_av', 'ignore']
URL = 'https://api.binance.com/api/v3/klines'
def get_data(start_date, end_date, symbol):
    data = []
    
    start = int(time.mktime(datetime.strptime(start_date + ' 00:00', '%Y-%m-%d %H:%M').timetuple())) * 1000
    end = int(time.mktime(datetime.strptime(end_date +' 23:59', '%Y-%m-%d %H:%M').timetuple())) * 1000
    params = {
        'symbol': symbol,
        'interval': '1m',
        'limit': 1000,
        'startTime': start,
        'endTime': end
    }
    
    while start < end:
        print(datetime.fromtimestamp(start // 1000))
        params['startTime'] = start
        result = requests.get(URL, params = params)
        js = result.json()
        if not js:
            break
        data.extend(js)  # result에 저장
        start = js[-1][0] + 60000  # 다음 step으로
    # 전처리
    if not data:  # 해당 기간에 데이터가 없는 경우
        print('해당 기간에 일치하는 데이터가 없습니다.')
        return -1
    df = pd.DataFrame(data)
    df.columns = COLUMNS
    df['Open_time'] = df.apply(lambda x:datetime.fromtimestamp(x['Open_time'] // 1000), axis=1)
    df = df.drop(columns = ['Close_time', 'ignore'])
    df['Symbol'] = symbol
    df.loc[:, 'Open':'tb_quote_av'] = df.loc[:, 'Open':'tb_quote_av'].astype(float)  # string to float
    df['trades'] = df['trades'].astype(int)
    return df

start_date = '2021-06-01'
end_date = '2021-12-31'
symbol = symbols_usdt[0]
get_data(start_date, end_date, symbol)
```
<img src="/blog/post_images/binance/data_result.png" title="binance_api">
위의 결과처럼 start_date, end_date, symbol을 넣으면 해당되는 데이터를 뽑아준다. 이때 틱 단위를 바꾸고 싶으면 **params에서 inverval을 3m, 1h**등으로 수정해 사용할 수 있다.

## 여러 암호화폐 데이터 수집 및 저장
```Python
import time
years = list(range(2017, 2022))  # 바이낸스에서는 2017년 8월 이후의 데이터부터 제공
for symbol in symbols_usdt[:10]:
    for year in years:
        start_date = f'{year}-01-01'
        end_date = f'{year}-12-31'
        df = get_data(start_date, end_date, symbol)
        df.to_csv(f'E:/projects/binance/data/{symbol[:3].lower()}_{year}.csv', index=False)  # csv파일로 저장하는 부분
				time.sleep(1)  # 과다한 요청으로 API사용이 제한되는것을 막기 위해
```
위의 코드를 실행하면 비트코인, 이더리움 등 10가지 암호화폐의 2017년부터 현재까지 데이터를 csv파일로 저장할 수 있다.<br>
다음 포스트에는 간단하게 암호화폐 데이터에 대한 EDA를 해볼 예정이며 기타 코드에대한 문의사항이나 매매전략에대한 아이디어가 있으신 분들은 `edwardfdsp@gmail.com`로 메일 부탁드립니다.
