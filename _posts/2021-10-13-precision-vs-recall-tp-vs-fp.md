---
title: TP vs FP / Precision vs Recall  정리
layout: post
date: '2021-10-26 22:45:08'
author: Edward Park
categories:
- DataScience
tags: []
cover: "/assets/instacode.png"
---

## 개요
분류모델에서 성능을 평가하는 지표로 AUC, F1-score 등이 사용된다. 그리고 Precision, Recall 등은 이를 계산할때 활용된다.<br>
하지만 이름만 들어서는 어떻게 계산되는지 와닿지않아 매번 사용할때마다 헷갈려서 이렇게 정리를 해두고자 한다.

## TP, FP, TN, FN
**T, F**<br>
T : 예측과 실제 결과가 일치<br>
F : 예측과 실제 결과가 불일치<br>
**P, N**<br>
P : **예측**을 참으로 하는 경우<br>
N : **예측**을 거짓으로 하는 경우

- TP(True Positive) : 예측을 참으로 했고, 실제도 참인 경우<br>
- FP(False Positive) : 예측을 참으로 했지만 실제가 거짓인 경우

## Precision vs Recall
- Precision = $$\frac{TP}{TP+FP}$$, 참으로 **예측**한 것중 실제 참인것의 비율
- Recall = $$\frac{TP}{TP+FN}$$, **실제** 참인 것중 예측을 참으로 한 것의 비율(FN은 예측은 거짓, 실제는 참인 것)

F1-score는 Precision과 Recall의 조화평균이며 다음과 같이 나타낼 수 있다.<br>
$$F1-score=\frac{2}{\frac{1}{Precision} + \frac{1}{Recall}}=\frac{2\cdot Precision\cdot Recall}{Precision + Recall}$$
