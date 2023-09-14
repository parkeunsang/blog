---
title: 뉴스기사와 주가 관계 분석 - (2) 데이터 전처리
layout: post
date: '2023-09-14 00:30:00'
author: Edward Park
categories:
- Stock
tags:
- Stock
- Python
- Kospi
cover: "/assets/instacode.png"
---

## Intro
[이전 포스트(데이터 수집 편)](https://parkeunsang.github.io/blog/stock/2023/08/13/news-stock.html)에서 네이버 증권의 뉴스 기사를 수집하는 방법을 소개했습니다. 이번 포스트 에서는 수집한 데이터를 데이터 분석, 모델링을 하기 위한 전처리 과정을 진행하겠습니다. 참고로 2023.08.12 까지의 데이터는 [kaggle](https://www.kaggle.com/datasets/park123/korean-news-title-datafrom-naver-stock)에 업로드 해두었습니다.

## 주가 데이터 저장
일자별 뉴스 제목 데이터와 주가의 관계를 분석하기 위해 주가 데이터를 FinanceDataReader Library를 활용해 수집하고, 저장합니다. FinanceDataReader의 version은 가급적이면 최신 버전(0.9.50 이상)사용을 권장드립니다(`fdr.__version__` command로 version 확인 가능).
```Python
import pandas as pd
import FinanceDataReader as fdr
import os
from tqdm import tqdm
import pickle
from datetime import datetime
import numpy as np

# write
def write_stock_data(end):
    end_fdr = datetime.strptime(end, '%y%m%d').strftime('%Y-%m-%d')
    codes = [x.split('_')[-1].split('.')[0] for x in os.listdir('data_news')]
    data = {}
    for code in tqdm(codes):
        df = fdr.DataReader(code, '2022-01-01', end_fdr)
        data[code] = df

    with open(f'data_stock/kospi_{end}.pkl', 'wb') as f:
        pickle.dump(data, f)
				
write_stock_data('230914')  # 수집하고자 하는 시점의 end date
```
작업하고 있는 폴더에 `data_news` 라는 폴더를 생성하고, 위 코드를 실행시키면 해당 폴더에 `news_005930.csv, news_357120...` 형태의 데이터가 저장됩니다.

## 데이터 전처리
뉴스 제목과 날짜, 그리고 다음날 시초가에 매수해 n일 후 매도했을때의 수익률을 병합하는 과정을 진행합니다.
```Python
def get_change_value(df, endure=0):
    """
    Args:
        df: target data frame
        endure: 얼마 뒤에 팔 것인지. 0이면 당일 시초가에 사서 당일 종가에 매도
    Return:
        result: pd.DataFrame
    """
    col_name = f'Change_{endure}'
    df = df.copy()
    df[col_name] = df['Close'].shift(-endure) / df['Open'] - 1
    df.loc[df['Open'] == 0, col_name] = -999
    return df
		
def get_y(df, date, col_name):
    df_target = df[df.index > date]
    if len(df_target):
        y = df_target[col_name].values[0]
    else:
        y = np.nan
    return y
# 위에서 저장한 data를 load
codes = [x.split('_')[-1].split('.')[0] for x in os.listdir('data_news')]
with open('data_stock/kospi_230914.pkl', 'rb') as f:
    data_stock = pickle.load(f)
endure = [0, 5, 20]  # 며칠있다 매도할지
for key, df in tqdm(data_stock.items()):
    for e in endure:
        df = get_change_value(df, e)
    df['Code'] = key
    data_stock[key] = df
		
records = []
for code in tqdm(codes):
    try: 
        df_news = pd.read_csv(f'data_news/news_{code}.csv')
    except pd.errors.EmptyDataError:  # empty news
        continue
    df_stock = data_stock[code]
    comp_name = df_kospi.loc[df_kospi['Code'] == code, 'Name'].values[0]
    df_news['name'] = comp_name
    df_news['code'] = code
    for e in endure:
        df_news[f'y_{e}'] = df_news.apply(lambda x:get_y(df_stock, x['time'], f'Change_{e}'), axis=1)
    df_news_records = df_news.to_dict(orient='records')
    records.extend(df_news_records)
		
df = pd.DataFrame(records)
df.to_csv('dataset_230914.csv', index=False, encoding='utf-8-sig')  # 최종 형태로 저장
df.sample(5)
```
위의 코드를 실행시키면 df에 아래와 같은 데이터가 저장됩니다. 
<img src="/blog/post_images/stock/stock_news_1.png" title="real_estate">
- y_0 column은 기사 배포 후 다음날 시초가에 매수하고, 당일 종가에 매도했을 때의 수익률
- y_5 column은 기사 배포 후 다음날 시초가에 매수하고, 5영업일 후 매도했을 때의 수익률
입니다.<br>

다음 포스트에서는 위 데이터를 활용한 EDA 및 모델링을 진행할 예정입니다. 감사합니다.
