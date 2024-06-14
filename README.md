# 부산 평균기온 예측

## :alarm_clock: 개발 기간: 2024년 3월 27일 ~ 2024년 4월 12일
## 개발환경:
|IDE|프로그래밍<br/>언어|
|------|---|
|![VISUAL STUDIO CODE](https://img.shields.io/badge/Visual_Studio_Code-0078D4?style=for-the-badge&logo=visual%20studio%20code&logoColor=white)|![PYTHON](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)|

## :people_holding_hands: 멤버구성 및 역할
[조장]

천진원(AUTO-ARIMA(S-ARIMA, ARIMA) 모델링)

[조원]

김현주(CNN-LSTM 모델링), 심재은(DNN, RNN, LSTM 모델링), 이승후

* EDA, Preprocessing은 조원 모두가 함께 진행

## :bulb: 기획 선정 및 배경
현재 지구온난화는 지구온난화를 넘어 지구열대화라는 새로운 용어로 사용해야한다고 논의되는 만큼 그 심각성이 매우 커진 상황입니다.

다가오는 기후변화를 예측하지 못한다면 우리 인류는 지구온난화의 각종 위협에 제대로 대처하지 못하고 결국 인류 멸망이라는 최악의 시나리오에 직면하게 될 것입니다.

그러므로 미래의 기후를 예측하고 다가오는 기후변화에 대응하는 것은 매우 중요한 과정입니다. 따라서 본 프로젝트를 통해 각종 기후지표를 통한 다변량 시계열 예측을 통해 미래 기온을 예측하는 모델을 구축하여 기후변화에 대응하고자 합니다.

## :bar_chart: 사용한 데이터
[기상청] 기상자료개발포털 데이터(1993년 1월 1일 ~ 2023년 12월 31일)

* 기후 평년값 기준이 약 30년이므로 해당 기간의 데이터를 가져옴


## :robot: 모델 개발
> ### EDA
1. 컬럼 정보 확인

|컬럼명|
|------|
|날짜|
|지점|
|평균기온(℃)|
|최저기온(℃)|
|최고기온(℃)|
|평균습도(%rh)|
|강수량(mm)|
|일조합(hr)|
|일조율(%)|
|일사합(MJ/m2)|
|평균풍속(m/s)|

* 최초 데이터는 최고기온(℃)까지로, 평균습도 ~ 평균풍속까지는 모델 예측에 필요한 기후 변수를 추가한 것임

2. 피쳐 간 상관관계 분석<br/>
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/104814c8-4c75-4eae-aebb-c574e07d000e)

- 평균기온은 최저, 최고기온과 0.99로 매우 높은 상관관계가 있고, 일사합, 일조율과는 낮은 상관관계가 있으며 강수량, 평균풍속과는 매우 낮은 상관관계가 있음

- 독립변수 간에는 최저, 최고기온, 평균습도 간에 높은 상관관계가 있고, 일조합, 일조율, 일사합 간에 높은 상관관계가 있음

3. 결측치 확인<br/>
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/16b04e44-307c-43a5-86aa-b55ebdc65a8f)

* 강수량(mm): 7554, 일조합(hr): 14, 일사합(MJ/m2): 95, 평균풍속(m/s): 5의 결측치 탐지

4. 이상치 확인<br/>
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/64c2279a-e2d9-460a-97dc-a0cfa95945ea)

* 강수량에서 많은 이상치 탐지, 그러나 한국 기후 특성상 여름철 장마로 인해 발생한 것으로, 이는 계절 특성에 의한 것이므로 이상치라고 판단하지 않았음

5. 날짜별 시각화<br/>
- 날짜별 평균기온
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/015e792a-6e2f-436a-9917-0e9d88669964)

- 날짜별 평균습도
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/60f3974e-0e8c-433a-bed0-da7e2d12bc27)

- 날짜별 평균강수량
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/aef82e70-fcf9-4e07-aac0-7f7d52194339)

- 날짜별 평균풍속
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/37c595e0-1b76-4221-a677-b128c10d0fd2)

> ### Preprocessing
1. 불필요한 컬럼 제거(지점, 일조합(hr), 일조율(%))

- 일조합, 일조율, 일사합은 모두 비슷한 변수로, 서로 높은 상관관계를 지니고 있음. 이것은 다중공선성 문제를 발생시킬 수 있으므로 Target과 상관관계가 가장 높은 변수인 일사합을 제외한 나머지 변수들은 제거

- 지점은 부산 지점에 한정되어 있어 의미 있는 데이터가 아니므로 제거

2. 컬럼명 변경

- 컬럼에 ()가 들어있는 부분이 많으므로 변경
  
```['날짜', '평균기온(℃)', '최저기온(℃)', '최고기온(℃)', '평균습도(%rh)' ,'강수량(mm)', '일사합(MJ/m2)', '평균풍속(m/s)']  > ['날짜', '평균기온', '최저기온', '최고기온', '평균습도', '강수량', '일사합', '평균풍속']```

3. 결측치 처리

- 최저기온의 데이터 결측치의 경우, 실제 데이터를 확인해 본 결과 전날과 거의 동일한 기온을 가지고 있어 ffill을 통해 전날의 데이터로 결측치를 채움

- raw data에서 강수량이 1mm보다 적게 온 경우에 0으로 표기를 했고, 강수량이 0인 경우 NaN값으로 표기했음. 따라서 1mm보다 작게 온 경우에는 0.01로 값을 대체하고 NaN값을 0으로 대체하였음.

- 평균풍속의 경우, 각 날마다 풍속이 매번 바뀌는 경우가 많아 다른 기준점을 만드는 것보다 평균풍속의 평균으로 결측치는 해결하는 것이 맞다고 판단하여 평균으로 결측치를 해결

- 습도와 일사합은 사계절이 있는 한국 날씨의 특성상 전체 평균으로 할 경우 예측 결과에 문제가 될 수 있다고 판단하여 전체 연도 결측 달의 평균으로 대체

4. 파생변수 생성

- VIF 다중공선성 진단 결과 최저기온, 최고기온, 평균습도가 높은 다중공선성이 있음.<br/>
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/b7737b6f-ecc5-4544-9acd-d8ab152f1765)

- 따라서 다중공선성 해결을 위해 이 세 가지의 변수를 사용하여 극값평균_습도라는 파생변수를 생성했음
  - 최저, 최고기온을 평균 내고 습도를 가중치로 곱한 값으로, 습도를 가중치로 곱한 이유는 한국의 사계절 특성상 습도가 사계절에 따라 큰 차이를 보이므로 유의미한 결과를 가져올 수 있다고 판단함
  - 극값평균과 평균기온과 다른 점은 극값평균의 경우 그날의 최저, 최고기온을 평균한 것이고, 평균기온은 일 8회 정시 관측값을 사용함

5. 데이터 정규화
- RobustScaler를 사용했음(강수량에서 많은 이상치가 나왔으므로 이상치에 상대적으로 민감하지 않은 RobustScaler 사용)

6. data split + Sliding window 적용

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/4bfca7b8-6330-4243-9fbc-b94655912aa1)

> ### Modeling
1. ARIMA

- ARIMA는 단일 변수인 평균기온만 사용하기 위해 평균기온만 존재하는 시계열 데이터를 새롭게 만들었음<br/>
![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/bef52929-d5a0-45c1-bb6f-1f49cc11d67e)

- kpss와 adf를 이용하여 최적의 d값을 계산했음(Optimized 'd' = 0)

- 계절성을 반영한 Auto_ARIMA와 계절성을 반영하지 않음 Auto_ARIMA 두 모델을 생성

- 트렌드만 고려한 예측, 한 지점에 대한 예측 후 모델을 업데이트 하는 방식 두 가지로 예측 진행
  
  - Auto_ARIMA_Seasonal
 
 - 트렌드만 고려한 예측

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/fcd004fe-ac30-4628-a624-5f75fe876997)

- 한 지점에 대한 예측 후 모델 업데이트<br/>![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/73dc9155-8d44-4b3f-9917-c72e1ad1fc89)

* MAE: 1.46, MSE: 3.98, RMSE: 1

  - Auto_ARIMA_Non_Seasonal

- 트렌드만 고려한 예측

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/da54e7e6-a830-43e9-9296-523a249fc444)

- 한 지점에 대한 예측 후 모델 업데이트<br/>![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/3c676e2f-9819-418d-9daf-39e5c3bfbe62)

* MAE: 1.47, MSE: 3.91, RMSE 1

2. DNN, RNN, LSTM, CNN-LSTM 모델 성능 비교

* NN 모델은 다변량 데이터로 진행했음

|모델명|MAE|MSE|RMSE|
|-----|----|---|----|
|DNN|0.2443|0.1230|0.3507|
|RNN|0.2046|0.0758|0.2754|
|LSTM|0.1908|0.0603|0.2457|
|CNN-LSTM|0.2015|0.0678|0.2602|

* LSTM 모델을 선정하여 최종 예측 진행(RNN = 장기의존성 문제, CNN-LSTM = 과적합 문제)

> ### Forecast

* Input data = 사용한 데이터 제일 마지막 90일(2023년 10월 3일 ~ 2023년 12월 31일)

* Output data = 90일의 미래 평균기온(2024년 1월 1일 ~ 2024년 3월 30일)

* 결과

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/2d2850c5-8ca8-4871-9552-38abfb518d5b)

> ### LSTM 성능 개선

1. 스케일링을 적용하지 않았음

- 스케일링을 사용하지 않으면 얼마 정도의 성능이 나올지에 대한 실험으로 진행했음. 데이터의 특성이 뚜렷한, 즉 사계절 기후의 특성이 존재하는 한국 계절 상 데이터의 특징을 평탄화하는 스케일링을 진행하게 되면 그 특성을 제대로 반영하지 못해 모델 예측 성능에 영향을 미쳤다고 판단함. 측정 결과 성능이 좋아졌음

2. test dataset에 적용되어 있던 sliding window 제거
- 데이터를 split 하는 함수에서 train과 test dataset 모두 sliding window가 적용되어 있었음. 따라서 train, test를 미리 분리하고 train에만 sliding window dataset을 적용함

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/11665fab-8318-4603-bad1-c28e2c22c82a)

3. random seed 설정
- 기존에 random seed가 설정되어 있지 않아 재현율에 문제가 발생하였음. 따라서 random seed를 고정하여 재현율을 높임

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/b8bdc57a-7ba9-4c2a-91ba-de2cf0ff2917)

4. epoch, batch_size를 바꿔가며 실험 진행
- 여러 실험 결과 기존 epoch 20, batch_size 32에서 좋은 성능을 보임(기존에도 같은 epoch, batch_size 사용)

#### 결과

![image](https://github.com/Jinwonie/temp_forecast/assets/155731578/8993c1de-b616-4217-8228-08fec5429aec)

* MAE: 3.4642, MSE: 18.9176 RMSE: 4.3464
