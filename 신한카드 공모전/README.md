## 신한카드 데이터 해커톤



### 공모전에 대해

신한카드에서 하루동안 진행하는 데이터 해커톤입니다. 주제는 카드 이용 고객의 **가구 구성원 수 예측** 이었습니다.



### 공모전 순위

장려상(3위)을 수상했습니다.



### 공모전을 하게된 동기

다양한 데이터를 접해보고 싶었습니다. 특히, 카드사 데이터는 개인정보 이슈로 인해서 접하기가 어렵고 공모전에서 제공하더라도 개인별로 제공되는 것이 아닌, 그룹별로 제공되는 경우가 많아 의미있는 분석을 하기가 쉽지 않습니다. 이번 해커톤에서는 혹시 유의미한 카드 데이터가 주어지는지 기대가 되어서 신청하게 되었습니다.



### 사용 기술

* Python



### 공모전에 대한 자세한 기술

이번 공모전은 보안상의 이슈로 ppt 반출이 불가능했기 때문에 따로 결과 ppt는 없습니다. 따라서 여기서 자세하게 그 내용을 기술하도록 하겠습니다.



#### 1) 주제

카드 이용 고객의 가구 구성원 수를 예측하는 것입니다. 은행 입장에서 가구 구성원 수를 파악하는 것은 아주 중요합니다. 독신인지, 기혼인데 자녀는 있는지, 자녀는 없는지 등에 따라서 소비 형태가 달라지고 마케팅이 달라지기 때문입니다. 이러한 배경 때문에 공모전이 주최된 것으로 보였습니다.

타겟으로 주어진 가구 구성원 수는 1, 2, 3, 4입니다. 즉 네개의 범주를 가지는 multi-classification이었습니다.

#### 2) 데이터

고객의 카드 이용 내역이 주어졌습니다. 하지만 경시적 자료는 아닌, 일정 기간에 고객의 카드 이용 내역을 모두 더한 데이터였습니다. 하지만 그룹별로 합산된 것이 아닌, 고객 개개인별로 합산되었기 때문에 각 고객의 특성을 파악할 수 있었습니다.

#### 3) 변수

고객의 결제 카테고리는 매우 다양했습니다. 약 20종류에서 30종류가 존재했던 것으로 기억하는데, 한 고객이 모든 종류의 소비를 행하는 것이 아니기 때문에 데이터가 전체적으로 sparse했습니다 (0의 값이 다수 존재했습니다)

#### 4) EDA

가장 먼저 기본적인 인적 정보에 대한 EDA를 수행하였습니다. 나이대, 거주지역 별로 가구 구성원 수의 분포가 상이하였습니다. 이는 어찌보면 당연한 결과인데, 사회적으로 볼 때에도 나이가 많을 수록 기혼일 가능성이 높고 학원가가 밀집해있는 목동, 강남 등에는 4인가구 이상이, 대학가 주변인 서대문구, 대학로 등은 1인가구가 많기 때문입니다. 이를 EDA로 직접 확인한 후에, 모형에 반드시 반영해야겠다고 생각했는데, 이들은 모두 범주형 변수이므로 어떻게 변수로 넣어야할지 고민이 되었습니다. 특히 거주 지역을 구 단위로 설정할 경우에 범주 개수가 매우 많아져, dummy variable을 만든다면 tree 기반 모형에서는 그 결과가 매우 좋지 않을 수 있기 때문입니다. 이러한 이유로 저는 범주형 변수에 대한 인코딩이 따로 필요 없는 LightGBM과 정수형 인코딩을 통한 XGBoost을 시도하기로 정하였습니다.

가구 구성원 별로 차이가 존재하는 변수는 상이했습니다. 예를 들어서, 편의점 결제 내역은 1인 가구 여부에 따라 크게 달라졌지만 유아 관련 결제 내역은 1인 가구 보다는 3인 가구에서 그 차이가 더 크게 나타났습니다. 또한 학원 관련 결제 내역은 4인 가구에서 가장 많은 것으로 파악됐습니다.

이렇게, 각 클래스를 이진 분류할 때, 사용할만한 변수는 여럿 존재하였지만 다수의 클래스를 한꺼번에 분류하는 multi-classification으로 이 변수들을 사용하는 것이 모델에 주요하게 작용할지 의문이 들었습니다. 따라서 저는 해당 EDA를 바탕으로 oneVsRest 기법의 분류 모형을 사용하기로 결정했습니다.

또한 새로운 변수로는 대출 상품의 종류 등을 통해서 담보 상태를 파악하고자 하였습니다.

#### 5) 모델링 및 결과 해석

앞서 EDA에서 계획한대로 oneVsRest 기법을 이용하고, 추가적으로 multi-classification을 사용하였습니다. 분류 모형으로는 LightGBM, XGBoost을 시도하였습니다.

파라미터 튜닝은 GridSearch가 아닌, hyperopt을 이용하여 베이지안 최적화를 통해 진행하였습니다.

저는 다른 팀들과는 다르게, SHAP을 이용해서 ML 모형의 결과를 해석하였습니다. SHAP은 모형이 왜 그렇게 결과를 냈는지 알려주는, 일종의 해석 기법입니다. 더이상 ML은 Black Box가 아니며 이를 해석하는 시도가 다양하게 이루어졌고 저는 이번 대회를 통해서 이에 대한 경험을 가져가고자 시도하였습니다.

SHAP을 통해서 1인, 2인, 3인, 4인 가구를 분류하는데 주요하게 영향을 미친 변수를 각각 리포트하여 신한카드 관계자들에게 인사이트를 제공하였고 비즈니스 활용 방안을 어필했습니다. 바로 이 점이 수상에 주요하게 영향을 준 듯 했습니다.

#### 6) 결과

정확도 0.44, F1 Macro Score 0.47의 처참한 결과를 받았습니다. 불행 중 다행으로, 다른 팀들도 모두 이와 비슷한 점수를 리포트했습니다. 시간이 너무 부족하기도 했고, 4인 가구를 민감하게 판별하지 못하는 점이 전체적인 패인이었습니다.

#### 

### 얻은 것

* 카드 데이터에 대한 경험

  카드 데이터를 예전부터 다뤄보고 싶었는데, 이번 해커톤을 통해서 접하게 되어서 나름 흥미로웠습니다. 한가지 아쉬웠던 점은, 카드 데이터가 경시적으로 주어지지 않고 시간에 대해서 통합되었기 때문에 시간에 따른 소비 패턴 분석 등은 시도할 수 없었다는 점입니다.

* SHAP 시도

  머신러닝 모델을 처음으로 해석해보았는데, 관계자분들의 반응이 생각보다 좋았습니다. 이를 보면서 관계자 분들도 단순히 모델링만을 돌리는 것을 원치 않고, 모델링 결과를 이용하여 인사이트 도출, 비즈니스 방안 등에 더 관심이 있는 것으로 보였습니다. 이러한 경험을 토대로 저는 공모전 이후에 머신러닝 모델을 해석해보는 공부를 할 예정입니다. LIME, SHAP에 대해서 페이퍼 리딩을 해보고 어떤 식으로 해석을 해야 하는지, 시각화를 해야하는지 등에 대해서 고민할 예정입니다.

