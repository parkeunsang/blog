---
title: "[Python] 바이낸스 API를 이용한 선물거래(매매)"
layout: post
date: '2022-05-10 20:06:11'
author: Edward Park
categories:
- Stock
tags:
- Bitcoin
cover: "/assets/instacode.png"
---

## 요약
[지난 포스트](https://parkeunsang.github.io/blog/stock/2021/06/11/python-api.html) 에서 **바이낸스 API**를 활용해 OHLC 데이터 수집을 해보았다. <br>
이번 포스트에서는 API를 이용해 **현재 보유종목 확인, 선물 거래**를 하는 코드를 구현해보고자 한다. <br>
CCTX나 binance와 같은 python library를 통해 위의 기능을 좀 더 쉽게 구현할 수도 있으나, 직접 API를 이용하는게 **보안과 기능의 확장성** 측면에서 좋다고 판단해 위의 library들은 사용하지 않았다. <br>


## 1. API Key발급
가격조회가 아닌 개인 계정을 이용해 매수, 매도를 하는 것이므로 먼저 바이낸스 계정의 API키를 발급해야 한다.
1. binance 사이트에서 profile 아이콘을 누르고 API Management에 들어간다.
<img src="/blog/post_images/binance/api_1.png" title="api">
2. Create API 왼쪽의 input box에 API 이름을 입력하고, Create API를 눌러 API를 생성한다.
3. 이때 생성된 <font color="red">API Key와 Secret Key</font>는 선물 거래에 사용되니 기록해두어야하며(특히 Secret Key는 한번 생성하면 나중에 다시 확인할 수 없다), 유출되지 않게 유의해야한다.
4. Edit 버튼을 눌러 **Enable Future** 를 체크하고, Save버튼을 눌러 저장한다. 
<img src="/blog/post_images/binance/api_2.png" title="api">

## 2. 계좌 잔고, 주문내역, 상태 조회
API Key의 경우 보안을 위해 환경변수로 저장하는게 낫다. [환경 변수를 이용한 API Key관리](https://parkeunsang.github.io/blog/python/2021/07/06/api-key.html) 참고<br>
이하 코드에 등장하는 API 요청은  [바이낸스 API 공식 문서](https://binance-docs.github.io/apidocs/futures/en/#public-endpoints-info) 를 참고했다.
```Python
import requests as rq
import hmac
import hashlib

# 위에서 확인한 api key, secret key
API_KEY = 'your api key'
SECRET_KEY = 'your secret key'
URL = 'https://fapi.binance.com'
headers = {
    'X-MBX-APIKEY': API_KEY
}

def get_user_data(url_sub):
    now = rq.get('https://api.binance.com/api/v3/time').json()['serverTime']  # 현재 시점
    message = f'timestamp={now}'
    signature = hmac.new(key=SECRET_KEY.encode('utf-8'), msg=message.encode('utf-8'),
                 digestmod=hashlib.sha256).hexdigest()
    url = f'{URL}{url_sub}?{message}&signature={signature}'
    result = rq.get(url, headers = headers)
    return result.json()

balance = get_user_data('/fapi/v2/balance')  # 내 계좌 잔고
order_log = get_user_data('/fapi/v1/allOrders')  # 내 주문 내역
account = get_user_data('/fapi/v2/account')  # 내 계좌 정보
print(balance)
```
- GET요청으로 현재 내 계좌 정보를 조회할 수 있다.
- 일반적인 api get 요청과 다르게 **현재 시점**과 **signature**를 url에 담아서 요청해야한다. signature는 서명에 해당하는 내용으로, secret key의 해쉬값을 의미한다.

## 3. 선물 매수/매도
매수, 매도의 경우 계좌의 데이터를 변경하는 것이기 때문에 **POST**요청을 사용한다.
```Python
def order(symbol, side, quantity)-> int:
    now = rq.get('https://api.binance.com/api/v3/time').json()['serverTime']  # 현재 시점
    if quantity >= 10:
        quantity = str(int(quantity))
    else:
        quantity = str(quantity)
        quantity = quantity[:5]
        if quantity.endswith('.'):
            quantity = quantity[:-1]
    if side == 1:
        side = 'BUY'
    else:
        side = 'SELL'
    message = f'symbol={symbol}&side={side}&type=MARKET&quantity={quantity}&timestamp={now}'
    signature = hmac.new(key=SECRET_KEY.encode('utf-8'), msg=message.encode('utf-8'),
                 digestmod=hashlib.sha256).hexdigest()
    url = f'{URL}/fapi/v1/order?{message}&signature={signature}'
    result = rq.post(url, headers = headers)
    return result.status_code
		
symbol = 'IOTAUSDT'
side = 'SELL'  # or BUY
quantity = 20
status_code = order(symbol, side, quantity)
print(status_code)  # 200이 프린트되면 정상적으로 주문된것
```
- 위의 코드는 **IOTAUSDT**를 <font color="red">시장가(Market)</font>로 20개 Sell(Short)하는 코드이다.
- 만약 Leveraeg가 20배이고, IOTAUSDT가 현재 1$ 이면 내가 투자한 금액은 1$가 된다.
- 총 **매수/매도하는 금액이 너무 작거나 수량의 소수점 아래 수가 너무 많으면** 정상적으로 주문이 처리되지않아 중간에 이를 조정하는 부분을 넣었다.
<img src="/blog/post_images/binance/api_3.png" title="api">

## 마무리
필자의 경우 OHLC데이터만 사용해 RNN기반 가격예측 모델을 만드는것을 시도했으나, 수수료를 이길(시장가 매매의 경우 한번 사고팔때 약 0.1%의 수수료 발생) 전략을 구현하지는 못했다. 이 과정에서 트레이딩 경험이 적어 데이터를 보는 인사이트가 매우 부족하다고 느꼈다. <br>
그래서 **트레이딩에 도메인 Knowledge**가 있으신 분들과 함께 협업하면 시너지가 잘 나올 것 같아 이러한 알고리즘 매매에 관심있으신 분들은  `edwardfdsp@gmail.com`로 메일 주시면 감사하겠습니다.
