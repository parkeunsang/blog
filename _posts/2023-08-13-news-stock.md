---
title: 뉴스기사와 주가 관계 분석 - (1) 데이터 수집(네이버 증권 뉴스)
layout: post
date: '2023-08-13 22:01:40'
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
주가는 여러가지 factor(요인)들에 의해 움직입니다. 이러한 factor에는 금리, 환율 등 거시적 지표, 최근 주가의 횡보(OHLC: Open, High, Low, Change), 개별 종목의 재무관련 데이터와 같은 **수치화 가능한 데이터**가 있고, 종목 관련 기사와 같이 **수치화가 어려운(정성적) 데이터**가 있습니다리고 투자자들의 심리적 요인이나 알려지지 않은 호재/악재 등 외부자의 입장에서 **수집, 분석하기 어려운 데이터**들도 있을 것입니다.

저같은 경우 약 1년 전, 암호화폐, 국내 주식을 대상으로 수치화된 데이터를 이용한 분석을 여러 방면으로 진행했었고, 뚜렷한 성과를 내지 못했습니다. 그래서 나중에 뉴스기사와 같은 **정성적인 데이터를 활용해봐야겠다**는 생각을 하며 잠깐 분석을 멈추었습니다.
그리고 최근 NLP쪽 공부를 할 기회가 생기기도 했고, 시간적 여유도 생겨서 뉴스기사와 KOSPI 주가간의 관계를 분석하는 과정을 가볍게 블로그에 기록하고자 합니다.  (1) 데이터 수집 - (2) 데이터 전처리 - (3) EDA 및 clustring기반 분석 - (4) LSTM, BERT 모델 기반 예측 - (5) 뉴스 기사와 개별 종목 OHLC 데이터를 접목한 모델링 순서로 5개 정도의 글을 작성할 예정입니다.

## 뉴스 데이터 수집
전체 뉴스데이터를 수집하기에는 리소스도 부족하고, 불필요한 기사도 많습니다. 그래서 [네이버 증권](https://finance.naver.com/item/news.naver?code=005930)에 각 종목별 연관된 뉴스를 제공하는 란이 있는데, 이부분의 데이터를 수집하였습니다.

### Module
- FinanceDataReader의 경우 **0.9.50** 이상의 버전 사용 권장드립니다.
- working dir에서 data_news라는 폴더안에 각 종목 코드별 csv파일을 생성해 저장합니다(data_news/005930.csv, data_news/005490.csv, ...)
- 기존에 저장되어있는 데이터가 있는 경우 **해당시점 이후의 데이터만 수집**합니다
- Request 요청에 문제가 있는 경우 요청 Term을 점점 늘려서(최대 1000초까지) 요청을 시도합니다. 다만 현재 시점 기준(23.08.13) Client쪽이 아닌 Server쪽 이슈로 Request 요청에 문제가 생겼던 경우는 없었습니다.

```Python
import requests
from bs4 import BeautifulSoup as bs
import pandas as pd
from tqdm import tqdm
import time
import FinanceDataReader as fdr
import os

class Scrap:
    def __init__(self, code):
        self.code = code
        self.headers = {'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.79 Safari/537.36'}
        self.check_exist()
        self.recent_request = []  # page_num이 증가해도, 값이 안바뀌는 지점이 있음
        
    def check_exist(self):
        file_name = f'news_{code}.csv'
        if file_name in os.listdir('data_news'):
            try:
                self.df = pd.read_csv(f'data_news/{file_name}')
                self.saved_date_max = self.df['time'].max()  # str. ex) '2023.04.17 17:56'
            except pd.errors.EmptyDataError:
                self.df = pd.DataFrame()
                self.saved_date_max = '2000.01.01 00.00'
        else:
            self.df = pd.DataFrame()
            self.saved_date_max = '2000.01.01 00.00'
            
    def get(self, url):
        wait = 30
        while wait < 1000:
            try:
                res = requests.get(url, headers=self.headers)
                return res
            except:
                print(f"===== {url} Connection Error {wait}")
                time.sleep(wait)
                wait *= 2
        raise Exception("Max Connection Request")
        
    def get_data_from_row(self, row):
        data = {}
        title = row.find('a', {'class': 'tit'})
        news_comp = row.find('td', {'class': 'info'})
        te = row.find('td', {'class': 'date'})
        if title:
            data['title'] = title.text.strip()
        if news_comp:
            data['news_comp'] = news_comp.text.strip()
        if te:
            data['time'] = te.text.strip()
        return data

    def get_last_page_num(self):
        url = f'https://finance.naver.com/item/news_news.naver?code={self.code}&page=10000'
        res = self.get(url)
        soup = bs(res.text)
        table = soup.find('table', {'class':'Nnavi'})
        if table:
            last_page_num = soup.find('table', {'class':'Nnavi'}).find_all('a')[-1].text
            last_page_num = int(last_page_num.replace(',', ''))
            return last_page_num
        else:
            return -1
    
    def get_news(self, page_num):
        url = f'https://finance.naver.com/item/news_news.naver?code={self.code}&page={page_num}'
        iter_stop = False  # 마지막 페이지거나, 기존 저장된 데이터와 중복되는 경우 iter_stop = True
        res = self.get(url)
        soup = bs(res.text)
        table = soup.find('div', {'class': 'tb_cont'}).find('table').find('tbody')
        rows = table.find_all('tr')
        if rows is None:  # 정상적인 response를 받지 못한 경우
            raise Exception('Invalid Response')
        if len(rows) == 0:  # Last Page
            return [], True
        elif len(rows) == 1:
            if rows[0].find(attrs={'class':'title'}) is None:
                return [], True
        news_data = []
        for row in rows:
            sub_table = row.find('tbody')
            if sub_table:
                sub_rows = sub_table.find_all('tr')
                for sub_row in sub_rows:
                    data = self.get_data_from_row(sub_row)
                    if data:
                        news_data.append(data)
                    else:
                        print('===Invalid Row', self.code, page_num)
            else:
                data = self.get_data_from_row(row)
                if data:
                    news_data.append(data)
                else:
                    print('===Invalid Row', self.code, page_num)
        if not news_data or news_data[-1]['time'] < self.saved_date_max:
            iter_stop = True
        if news_data == self.recent_request:
            iter_stop = True
        self.recent_request = news_data
        return news_data, iter_stop
```
### 데이터 수집
- 현재 상장되어있는**kospi 종목을 대상**으로 수집을 진행합니다.
- 종목의 기사가 0건이라도 파일은 저장됩니다(row=0)


```Python
df_kospi = fdr.StockListing('KOSPI')
df_kospi = df_kospi[df_kospi['Market'] == 'KOSPI'].reset_index(drop=True)
# df_kospi_comp = df_kospi[~df_kospi['Sector'].isna()].reset_index(drop=True)
kospi_symbols = df_kospi['Code'].values
kospi_symbols = [x for x in kospi_symbols if x.endswith('0')]
for code in tqdm(kospi_symbols):
    scraper = Scrap(code)
    is_last = False
    news_data = []
    page_num = 1
    while not is_last:
        data, is_last = scraper.get_news(page_num)
        news_data.extend(data)
        page_num += 1
        time.sleep(0.5)
        if page_num % 100 == 99:  # 너무 많이 돌면 tracking
            print(code, page_num)
    df = pd.concat([pd.DataFrame(news_data), scraper.df])
    df = df.drop_duplicates()
    if len(df):
        df = df.sort_values('time', ascending=False)
    df.to_csv(f'data_news/news_{code}.csv', index=False, encoding='utf-8-sig')
```

### 수집 결과
22.04.16 ~ 23.08.13 데이터는 [Kaggle](https://www.kaggle.com/datasets/park123/korean-news-title-datafrom-naver-stock)에 업로드를 해두었습니다. 혹시 위 코드를 사용해 수집을 하시는 경우 <font color="red">중복된 데이터 수집을 방지하기 위해 제가 수집한 데이터를 다운로드하신 후 코드 실행</font> 부탁드립니다.
코드 관련 문의사항이 있으신 경우 `edwardfdsp@gmail.com` 으로 메일 부탁드립니다. 감사합니다.
