# AI/ML/DL

### 들어가며...

사내에서 교수 초청으로 듣게 된 AI/ML/DL 교육..

찬찬히 시간을 가지고 공부했으면 그래도 좀 이해했을텐데, 짧은 시간 내에 너무 방대한 내용을 머리 속에 넣으려니 도저히 정리가 안된다.

복습하고 혼자 정리해보는 시간을 가지려 했는데, 그러기엔 정말 너무너무 양이 많아서...

이해한 정도만 간단하게 리뷰하고, 자세한 내용은 잘 정리된 글들을 조각모음 해두고 필요할 때 내용을 꺼내 보아야겠다.

# AI

![img_01](https://miro.medium.com/max/1400/1*dPigpqVMcTZPgn-Wt5GCLQ.png)

AI/ML/DL 의 영역이 분리되어 있음을 이해하자!

- AI: 인간의 학습능력, 추론능력, 지각능력, 언어 이해 능력 등을 컴퓨터에 구현한 기술. (기계학습을 제외한, symbolic AI)
- ML: 과거의 경험을 바탕으로 추론 및 결정하는 방법을 머신에게 알려주고, 학습한 내용을 바탕으로 과거에 축적된 데이터를 분석하여 머신이 데이터를 평가, 판단하도록 하는 기술
- DL: 머신러닝의 일종으로, 인간의 뇌 구조와 같은 인공신경망 네트워크로 이루어지는 기계학습. 축적된 데이터 뿐 아니라 마치 사람처럼 스스로 학습할 수 있도록 하는 기술

## EDA (Exploratory Data Analysis, 탐색적 데이터 분석)

- 데이터 분석 과정에 대한 개념으로, 데이터를 분석하고 결과를 내는 과정에 있어 지속적으로, 다양한 각도에서 데이터에 대한 탐색과 이해를 기본으로 가지고, 자료를 직관적으로 바라보는 과정을 의미한다.
- 데이터 자체에 대한 해석이 잘못되면 모든 데이터 분석의 내용(프레임, 시각화 그래프 등)이 잘못되는 것이기 때문에, 데이터가 표현하는 현상을 더 잘 이해하고, 잠재적인 문제를 해결하기 위해서 EDA는 매우 중요한 개념이다.
- (충분하고 철저한 분석/설계 위에서 개발 해야 하는 것과 같은 이치!)

### EDA 에 필요한 기술

1. raw data 의 description, dictionary 를 통해 각 데이터의 컬럼과 로우의 의미를 이해하는 기술
2. 결측치(Missing value) 처리 및 데이터 필터링 기술
3. 직관적인 그래프 시각화 기술

**[...and more, 자세히 보기](https://jalynne-kim.medium.com/데이터분석-기초-eda의-개념과-데이터분석-잘-하는-법-a3cac2cc5ebc)**

# ML

처리하는 데이터: 정형 데이터 (데이터베이스, 레코드 파일 등)

학습 방법: 데이터를 기반으로 한 학습

알고리즘: 랜덤 포레스트 등

## 머신러닝의 종류

머신러닝을 할 때, '무엇을 분석하고자 하는가' 는 매우 중요한 문제이다.

데이터의 특성에 맞게 학습의 종류 (지도학습/비지도학습) 를 결정하고 시작하여야 한다.

### 1. 지도학습: 라벨(=y, 종속변수)이 있음

- 말 그대로 '가르쳐준다' 는 뜻. 데이터를 분석/예측함에 있어 정보(종속변수) 가 있는 경우
- 종속변수가 범주형 자료형이면 분류를, 연속형 자료형이면 예측을 할 수 있다.
- (예측) 회귀 분석: MAE Metric, RMSE Metric 사용. **잔차를 줄이는 것이 좋은 회귀식!**
- (분류) KNN(K-nearest neighbors algorithm), SVM, 의사결정 나무

### 2. 비지도학습: 라벨이 없음

- 데이터만 있고, 종속변수는 없는 경우
- 데이터의 분포를 보고 인사이트를 도출한다 (데이터 사이의 비슷한 특성(feature) 를 가진 것들로 그룹핑)
- 차원 축소(PCA)
- 클러스터링

### 3. 강화학습

> ## 지도학습 - 회귀 모델

### 회귀 모델

- Ridge: L1 Norm (=Gaussian)
- Lasso: L2 Norm (=Laplace)
- Elastic Net: mixed

### 성능 평가 지표: RMSE, MAE 모델

회귀 평가 지표는 실제 값과 회귀 예측값의 차이를 기반으로 한다.

값이 작을수록 예측값과 실제값의 차이가 없다는 뜻이기 때문에, MAE, MSE, RMSE, MSLE, RMSLE는 값이 작을수록 회귀 성능이 좋은 것이다. 반대로, R²(R Square) 는 값이 클수록 성능이 좋다.

- MAE (Mean Absolue Error)
  - 실제 값과 예측 값의 차이를 절댓값으로 변환해 평균한 것
- MSE (Mean Squared Error)
  - 실제 값과 예측 값의 차이를 제곱해 평균한 것
- RMSE (Root Mean Squared Error)
  - MSE 값은 오류의 제곱을 구하므로 실제 오류 평균보다 더 커지는 특성이 있어 MSE에 루트를 씌운 RMSE 값을 쓴다.
- MSLE (Mean Squared Log Error)
  - MSE에 로그를 적용.
- RMSLE (Root Mean Squared Log Error)
  - RMSE에 로그를 적용.
- R² (R Sqaure)
  - 분산 기반으로 예측 성능을 평가. R² = 예측값 Variance / 실제값 Variance (1에 가까울수록 정확)

**[...and more, 자세히 보기](https://bkshin.tistory.com/entry/머신러닝-17-회귀-평가-지표)**

> ## 지도학습 - 분류 모델

### 분류모델

- KNN(K-nearest neighbors algorithm)
- SVM(Support Vector Model)
- 의사결정 나무

### 성능 평가 지표

- Confusion Matrix (오차행렬)
  - 트레이닝을 통한 예측 성능을 측정하기 위해, 예측값과 실체값을 비교하기 위한 표.
  - True/False, Positive/Negative의 2\*2 행렬로 나타낸다. 예를 들어, TP/TN 은 실제 값을 맞게 예측한 부분이며, FP/FN 은 실제값과 다르게 예측한 부분을 의미한다.
- Accuracy (정확도)
  - 모델이 바르게 분류한 부분의 비율 (Confusion Matrix 의 TP, TN)
- Precision (정밀도)
  - 모델이 Positive 라 분류한 것 중 실제값이 Positive 인 비율 (Confusion Matrix 의 열 방향에서 TP)
- Recall (재현도)
  - 실제값이 Positive 인 것 중 모델이 Positive 라 분류한 비율 (Confusion Matrix 의 행 방향에서 TP)
- F1 score
  - Precision 과 Recall 의 조화평균
  - 데이터가 불균형/편향되어 있을 시 Accuracy 의 값이 정합성을 보장하지 못하므로, imbalanced data 에 F1 score를 지표로 사용한다.

**[...and more, 자세히 보기](https://leedakyeong.tistory.com/entry/분류-모델-성능-평가-지표-Confusion-Matrix란-정확도Accuracy-정밀도Precision-재현도Recall-F1-Score)**

### 의사결정나무

- 불순도: 한 노드의 모든 샘플이 같은 클래스에 속해있다 = 불순도가 낮다 (gini=0) => 불순도가 낮을수록 좋다
- 엔트로피: = (정보량 \* 확률) = 기대값. 엔트로피는 불확실성의 척도. (엔트로피가 높다 = 정보량이 많고, 확률이 낮다)
- 엔트로피가 높다 = 불순도가 높다 (min-max: 0-1)

  - cross entropy(정보량 \* 추정확률); deep learning - classification 에서 척도로 cross entropy 를 사용.

- 의사결정나무는 데이터 전처리가 거의 필요하지 않다 **중요장점!**
- 가지치기: 과적합 방지, 적절한 적합성을 보장하기 위해 실시.

- Model Ensemble(여러개의 모델을 사용하는 모델 앙상블)
  - Bagging: Random foreset (무작위 서브셋)을 구성하여 서브 샘플에 각각 학습시키고 그 모델을 만들어둔다.
  - Boosting: 모델의 약한 부분(=잔차)을 찾아내어, loss function 을 사용하여 가중치를 변화 -> 개선 모델을 만든다.
    - AdaBoost: 가중치를 이용. Stump(약한 분류기) 를 반복하여 만듬. stump 마다의 amount of say 를 합하여 최종 분류.
    - GradientBoost: 잔차 전체에 대한 모델을 만들어 기존 모델에 add
    - XGBoost: GradientBoost 를 병렬학습.

> ## 비지도학습 - 차원축소 (PCA)

차원축소는 많은 feature 로 구성된 다차원 데이터 세트의 차원을 축소하여 새F로운 차원의 데이터 세트를 생성하는 것. (고차원 -> 저차원)

일반적으로 차원이 증가할수록 (=feature 가 많아질수록) 예측 신뢰도가 떨어지고, 과적합(overfitting)이 발생하고, 개별 feature 간 상관관계가 높을 가능성이 있다.

이러한 데이터 분석의 위험성을 줄이고, 아래와 같은 장점을 가지기에 차원축소를 진행한다.

1. 시각화를 통한 데이터 패턴 인지
2. 노이즈 제거
3. 메모리 절약
4. 퍼포먼스 향상

### PCA (Principal Component Analysis; 주성분 분석)

차원 축소 방법 중 하나.

여러 개의 feature 중 가장 중요한 feature 몇 개만을 선택(주성분)하여 데이터를 분석하는 기법.

**[...and more, 자세히 보기](https://bkshin.tistory.com/entry/머신러닝-9-PCA-Principal-Components-Analysis?category=1057680)**

> ## 비지도학습 - 클러스터링(군집분석)

클러스터링이란, 어떤 데이터들이 주어졌을 때, 이들을 클러스터로 그룹핑 시켜주는 것을 의미한다.

거리 기반 클러스터링; 거리가 가까운 데이터들이 묶여 하나의 클러스터를 이루도록 함.

- 유클리디안 distance(L2 Norm); 최단 직선거리
- 맨하탄 distance(L1 Norm); 가로거리 + 세로거리
- cosine 유사도; 삼각함수 사용

### 성능척도

좋은 클러스터링? 클러스터 중심점과의 거리가 짧을수록 좋다.

- 실루엣
- Clustering Quality

### K-means Clustering

K-means clustering 에서,

1. K = 클러스터의 개수.
2. means = 클러스터의 중심 (centroid)

즉, K-means Clustering 이란 K 개의 centroid 를 기반으로 K개의 클러스터를 만드는 것을 의미하며, 이를 통해 유사한 데이터 포인트끼리 그룹핑하여 패턴을 찾아내기 위해 사용한다.

- 단점:
  1. 최적해를 보장하지 않음 (휴리스틱 알고리즘)
  2. 비선형 경계에 적용하기 어려움

**[...and more, 자세히 보기](https://bkshin.tistory.com/entry/머신러닝-7-K-평균-군집화-K-means-Clustering?category=1057680)**

### DBSCAN (DENSITY-BASED SPATIAL CLUSTERING OF APPLICATIONS WITH NOISE )

- outlayer 를 찾기에 최적.
- minpoints: corepoint <-> reachable point <-> noise
- ε (앱실론): 거리제한threshold (바운더리 원의 반지름)

# DL

처리하는 데이터: 비정형 데이터 (이미지, 영상, 음성, 텍스트 등)

학습 방법: 알고리즘의 가중치 분석, 가상 공간 이해를 위한 이론 도구 등을 제공

알고리즘: 의사결정나무, 선형회귀모형 등

cf) Cross Entropy

## 딥러닝 개념

### 1. 신경망 (인공신경망)

퍼셉트론(Perceptron): 딥러닝을 구성하는 기본 구조.

인간 뇌의 신경망 구조를 본뜬 구조로, [입력노드] 에서 여러 개의 입력 데이터를 받아 [은닉노드] 에서 계산한 다음, [출력노드] 로 하나의 출력값을 내보낸다.

1개의 출력값을 가지면 단층 퍼셉트론, 2개 이상일 경우 다층 퍼셉트론이라 부르며, 2개 이상의 은닉노드 layer 를 가질 때부터 '딥러닝' 이라고 부른다.

- Activation Functions
  - ReLu
  - Sigmoid
  - Hyperblic tansent

### 2. 순전파와 역전파

딥러닝에서는 데이터 학습 과정이 2가지 방향성을 지니는데,

**[입력층] -> [은닉층] -> [출력층]** 의 정방향으로 이어지는 것이 순전파, 역방향으로 이루어지는 학습법을 역전파라고 한다.

출력층을 거져 나온 출력값(예측값) 과 실제 값의 차이(=손실)존재할 수밖에 없다. 이 손실을 최소화하는 학습이 필요하고, [출력층] -> [은닉층] -> [입력층] 의 역방향으로 계산하여 손실을 최소화할 수 있도록 **가중치를 업데이트** 하는데, 이러한 학습법을 역전파라고 한다.

cf) 최적화 방법: Gradient Descent

### 3. CNN/RNN/GAN

3-1. CNN (Convolutional Neural Network; 합성곱 신경망)

- 원본 데이터에서 필터가 순차적으로 합성곱을 진행하며 출력값을 계산
- 작은 필터로 부분적으로 학습하고, 이런 작은 부분들을 종합하여 좀 더 큰 패턴을 학습

3-2. RNN (Recurrent Neural Network; 순환 신경망)

- 시계열 데이터처럼 시간에 따라 변화하는 데이터를 학습시키기 위한 딥러닝
- 신경망에서 학습의 계산 결과 이력을 기억하고, 그것을 다음 학습 시에 입력하는 재귀형 학습을 수행
- 은닉상태(Hidden State)를 파라미터로 가짐

3-3. GAN (Generative Adversarial Network)

- Auto encoder 의 개념을 역발상으로 활용한 모델. decoder 에서 이미지를 재현하는 기능을 가져와 generator (이미지 생성자) 의 역할을 만들고, 생성된 이미지를 descriminator(감별자) 라는 신경망에서 원본과 비교하여 '진/위' 여부를 감별. 진짜와 가짜를 감별할 수 없을 때까지 학습을 시키면 두 신경망의 성능이 평형상태에 이르며 학습이 완료된다.

- _cf) Auto encoder_: 원본 데이터의 수많은 feature 중, 가장 중요한 특성을 찾아내기 위한 모델.
  encoder 부분에서 원본 이미지(myself)를 압축하여 특성을 추출하고, decoder 부분에서 원본 크기만큼 이미지로 재현한 다음, 재현된 이미지를 원본과 비교하면서 학습하여 가장 중요한 특성을 code layer 층에서 추출하는 기법.

**[and more, 자세히 보기](https://jalynne-kim.medium.com/딥러닝-핵심-개념-핵심-용어-쉽게-알아보자-가능한-7196a39df5a0?p=7196a39df5a0)**

## 딥러닝 가중치 초기화

- 가중치의 초기값(Parameter Initialization; starting point) 설정이 중요!
  - 사비에르 초기화(=글로로트 초기화): 이전 은닉층의 노드의 개수가 n이고 현재 은닉층의 노드의 개수가 m일 때 2/√(n+m)를 표준편차로 하는 정규분포로 가중치 초기화
  - He 초기화(=카이밍 초기화): **ReLU함수를 활성화 함수로 사용할 때 추천**되는 초기화 방법. 이전 은닉층의 노드의 개수가 n, √(2/n) 를 표준편차로 하는 정규분포로 가중치 초기화 => 사비에르보다 활성값이 고르게 분포!

##

Batch Normalization

최적화 알고리즘

- Momentum

전이학습

## 자연어 처리

**[and more, 자세히 보기](https://wikidocs.net/109251)**

## 용어들

### 차원의 저주

- 데이터 학습을 위해 차원이 증가하면서(=변수의 수 증가), 개별 차원 내 학습할 데이터의 수가 차원의 수보다 적어져 모델의 성능이 저하되는 현상.
- 무조건 변수의 수가 증가한다고 해서 차원의 저주가 생기는 것이 아니라, 관측치 수보다 변수의 수가 많아지면 발생한다.
- 차원을 줄이거나, 더 많은 데이터를 획득하여 문제를 해결할 수 있다.

### 자유도

**[and more, 자세히 보기](https://blog.minitab.com/ko/statistics-and-quality-data-analysis/what-are-degrees-of-freedom-in-statistics)**
