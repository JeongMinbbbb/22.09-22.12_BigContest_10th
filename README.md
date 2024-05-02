# 제10회 빅콘테스트 데이터 분석 공모전(2022.09-2022.12)
> 데이터 분석 분야 퓨처스리그 참여

> 과학기술정보통신부와 한국지능정보사회진흥원이 주최


## 주제 : 앱 사용성 데이터를 통한 대출신청 예측분석


### **사용 데이터**
- 2022년 3월 ~ 6월    
1. user_spec.csv (user 신용정보)
    - 신청서 번호, 유저 번호, 신청 정보, 유저 정보 등 17개의 Column
    - 분석에 활용할 데이터      
2. loan_result.csv (사용자가 신청한 대출별 금융사별 승인결과)
    - 신청서 번호, 금융사 번호, 상품 번호, 한도, 금리, 신청 여부 **(target)** 등 7개의 Column
    - 예측해야하는 target 변수를 포함한 데이터      
3. log_data.csv (finda App 로그 정보 )
    - 유저 번호, 행동 명, 행동 일시 등 4개의 Column
    - 유실된 자료가 있다고 주최측에서 공지한 데이터 **(데이터 결함을 고려하여 부분 활용)**
    <p align="middle">
      <img src="https://github.com/JeongMinbbbb/22.09-22.12_BigContest_10th/assets/130365764/bcc063d3-1e35-426d-98ca-c02387b5c488" alt="image">
    </p>

## **분석 목적**
1. 고객 세그멘트별 홈화면 맞춤 메시지 기획
2. 특정 기간 내에 대출을 신청할 고객 예측
3. 앱 활성화를 위한 인사이트 도출

     
## **분석 과정**
1. **전처리**
    - 결측인 데이터들은 각각의 성격에 맞게 범주화, 삭제, 예측을 적용함
  
      1. 나이, 성별 결측은 예측할 수 없고, 삭제하기엔 데이터의 손실이 많아 **범주화**  
  
      2. 유저 정보에 결측이 있었지만, 해당 유저는 대출을 할 수 없는 청소년 혹은 본 분석 목적에 해당하지 않는 유저이기에 **삭제**   
  
      3. 신용 점수라는 변수는 연속형 변수라는 특징 and 대출 정보와 관련 높은 변수라고 판단 => 따라서 MissForest 모델을 사용한 **대체** 

2. **고객 특성 분류**
    - 행동 로그 및 인적 사항 데이터 전처리
    - 복잡한 이상치 정의를 위한 실험 진행
    - 2단계 군집분석으로 고객의 특성 분류
        - (1차) 짧은 기간 내에 조건을 자주 변경하는 고객들을 정보 오입력 그룹으로 정의하고 사전 분류
        - (2차) 정보 오입력 그룹 외 고객들을 대상으로 재차 고객 특성 분류
        - K-Means Clustering 방식을 사용
            - K값을 임의로 선택하여 군집화 가능
            - 새로운 사용자에 대한 군집 분류시 적절함
            - 대용량 데이터이기 때문에, 계산 시간 단축
    - 고객 특성별 메시지 기획
        - 대출 활성화 고객 그룹 : 변경된 대환 대출 금리 정보를 제공하는 사후 관리 서비스를 안내하여 충성 고객으로의 전환을 유도
        - 정보 오입력 고객 그룹 : 정확한 대출 조건 조회를 위해 자주 변경된 기존 정보들을 다시 확인하도록 유도 
        - 신규 고객 : 다양한 서비스 종류를 간략히 소개하고 목적에 맞는 튜토리얼을 선택하도록 유도
        - 잠재적 휴면 고객 : 팝업창을 통해 개인의 이용 현황을 전달하고, 업데이트된 신규 서비스를 소개  

3. **대출 신청 상품을 예측하는 모델 구축**
    - 데이터 불균형 문제 완화를 위해 대출 신청 프로세스의 특징을 반영한 모델링 진행
        - 대출을 실행한 고객의 신청서를 분류한 뒤, 신청서 내 추천된 각 대출 상품에 대한 신청여부를 예측
    - 데이터 불균형 문제 완화를 위해 Recall을 중요하게 판단
        - 대출 실행 고객의 신청서를 분류할 때, Recall에 더 높은 가중치를 부여한 평가지표 사용
        - 1차 모델링에서 FALSE로 판단한 신청서는 최종 의사결정에서 제외되므로, TRUE를 선정하는 기준을 더 유하게 적용할 필요가 있음
        - 따라서, F1-Score가 아닌 F1.5-Score를 기준으로 모델의 Threshold를 조정함으로써 2차 모델링 과정에서 3가지 모델을 앙상블
    - 사용한 모델
        - (1차) Catboost : 모델 학습에 사용한 데이터(user_spec)에 범주형 변수가 많았기 때문
        - (2차) XGBoost, Catboost, LGBM : 예측 성능이 준수하며 해석이 용이함, 앙상블을 적용하기 위해 Boosting 기반 모델 3가지를 사용
    - **모델링 개요**
        - 1차 모델링을 통해 필요 없는 신청서를 제거했기 때문에, 데이터 불균형을 줄일 수 있었음 **[1:19 ==> 1:9]**
      ![빅콘_모델구조_크기조정](https://user-images.githubusercontent.com/90736934/209518599-7b2d945f-8f89-4280-949a-77901a465170.png)

 
## **분석 결과 및 성과**
1. 모델의 최종 성능 F1-Score : 약 0.43
2. 성과 : 320개 팀 중 상위 4% 달성, 데이터 손실률 40% 축소


### 프로젝트 의의 및 성장한 점
1. 창의적인 접근 방식을 시도하여 문제 해결
2. 도메인의 특성 이해를 중요시하는 습관 형성
3. 행동 로그와 고객 데이터의 범용성을 이해

  
### 최종 탈락에 대한 피드백(팀원과 리뷰)
- 피처 엔지니어링 과정에서 과적합에 대한 검증이 부족
- 베이지안 최적화 알고리즘을 사용해 하이퍼파라미터 튜닝을 하고 과적합을 방지하려 했음
- 그러나 이 과정에서 K-fold cross validation 처리를 놓쳤고, 발표 당시 모델의 과적합에 대한 피드백을 받음
- 이로 인해 모델링을 2단계로 나눈 것에 대한 장점까지 퇴색됨. 불균형 해소와 추가적인 해석만으로는 방법론 도입의 근거가 부족

***
### Team Project
#### 팀 구성: 배정민, 이승준(팀장), 이한재, 조용민
#### 소속: 동국대학교 통계학과
***
