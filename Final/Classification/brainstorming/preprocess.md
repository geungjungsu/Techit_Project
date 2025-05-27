```
- 변수
    - Features(X)
        - ID : 순번
        - Customer ID: 각 고객의 고유 식별번호
        - Surname: 고객의 성
        - Credit Score: 고객의 신용점수
        - Geography: 고객이 거주하는 국가
        - Gender: 고객의 성별
        - Age: 고객의 나이
        - Tenure: 고객이 은행을 이용한 기간
        - Balance: 고객의 계좌 잔액
        - NumOfProducts: 고객이 이용하는 은행 상품의 수(ex. 예금,적금)
        - HasCrCard: 신용카드 보유 여부
        - IsActiveMember: 활성 회원 여부
        - EstimatedSalary: 고객의 예상 연봉
    - Target(Y)
        - Exited: 고객 이탈 여부
```

1. 변수 제거 후보
```
- Id
- Surname
- CustomerId
  - 고유값이라면 유지
  - 고유값 기능 못한다면 제거
```
2. 시각화 해석
   
![image](https://github.com/user-attachments/assets/51fabf79-f86d-4c35-b3fb-073ef23ef924)

```
# CreditScore : 정규분포, 400 이하 부터는 값이 적어보임
-> Low / Normal / High로 나눌 수 있을까?

# Geography : France > Spain > Germany(거의 6:2:2)

# Gender : Male > Female (거의 6:4 의 분포)

# Age : 정규분포에 가까움, 60대 이상부터는 값이 현저히 적어보임
-> 데이터가 적은 연령들을 사용하기 위해 연령대라는 파생변수를 만드는게 좋을까?
-> 10대, 20대, 30대, 40대, (50대 이상?) 혹은 (50대, 60대 이상)
-> 10대는 금융 상품을 구매할 수 없는 환경이 대부분이라서 20대에 포함시키는 것이 좋아보임
-> 30대가 대부분 -> 금융 상품 적극적으로 이용하기에 적합한 나이(경제활동 적극적으로 시작하는 나이)

# Tenure : 0과 10을 제외하고는 거의 유사한 분포

# Balance : 0이 분포의 대부분을 차지한다
-> 0의 데이터가 크기 때문에 0과 나머지를 1로 파생변수 생성 가능성?
-> 혹은 0을 없앤다면? -> 무리가 있어보임. 0도 의미가 있는 데이터 -> 잔고가 없는 계좌는 이탈 확률 올린다?

# NumOfProducts : 1과 2가 대부분의 분포를 차지하고 3,4는 극소수
-> 1과 2 그리고 나머지를 하나(3,4)로 만들어 버린다면?

# HasCrCard : 보유한 사람(1)이 분포의 대부분을 찾디한다

# IsActiveMember : 활성 고객과 비활성 고객의 분포가 거의 유사

# EstimatedSalary : 거의 정규분포

# Exited : 이탈하지 않은 고객이 이탈 고객의 약 3.5배

```




2. 변수 전처리
```
- 2_1.타입 변경
  - 인코딩
      - Object : Geography(3), Gender(2)
          - int
          - category
  - float -> int
      - Age
          - ex) 36.44, 32.34(소수점 존재)
          - 소수점의 의미 불명
      - Balance, EstimatedSalary
          - 소수점 의미 명확(타입 변경 X)
      - HasCrCard, IsActiveMember
          - 범주형 변수
          - int
          - category
          
- 2_2.스케일링(이상치 기준으로 스케일링 적용)

<
  - StandardScaler : Mean = 0, std = 1
      - 데이터 정규분포에 가까운 경우
      - 변수 크기 차이가 크고, 이상치 적을 때
  
  - RobustScaler : Median, IQR 사용
      - 이상치 영향 최소화
      - 중앙값 기준으로 스케일링 하므로 분포가 비대칭적인 경우 효율
      - 데이터가 정규분포를 따르는 경우, StandarScaler만큼 효과적이지 않을 수 있다
  
  - MinMaxScaler : Min = 0 , Max = 1 사이 스케일링
      - 데이터가 특정 범위내로 제한되어야 할 때
      - 값의 분포가 균등하지 않을때
      - 딥러닝 모델에서 주로 사용
      - 해석이 쉽다
      - 값의 분포를 유지하며 스케일링
      - 이상치에 민감
>
  - StandardScaler
      - Balance
  - RobustScaler
      - Age
      - CreditScore
  - MinMaxScaler
      - Tenure
      - EstimatedSalary

- 2_3.로그변환
  - 시각화 후 그래프 대체로 정규분포
  - 따라서 로그변환 X

- 2_4.중복치
    - 중복되면 안되는 변수
        - 1. id : 제거해도 좋아보임
        - 2. CustomerId : 고유값으로써 기능 상실
            -> 중복치 141813개 : 제거하는 것이 좋아보임
        - 3. EstimatedSalary : 예상 연봉이 소수점까지 아예 똑같을 수가 있나?
            -> 중복치 109736개 : 하지만 해당 변수의 중복치 발생 가능성 여부를 내가 판단하는 것은 좋지 않아 보이므로 유지
        - 4. 값이 모두 똑같은 중복치가 나올 수 있나?
            -> 0개
            -> train(id 제거) 중복치 : 0
            -> train(CustomerId 제거) 중복치 : 0
            -> train(id, CustomerId 제거) 중복치 : 54개 -> 발생할 수 없지않을까? 분석에 도움이 될지 의문

        - 5. 잔고, 예상연봉, 성(이름), 성, 국가, CustomerId가 동일할 수가 있나??
            -> CustomerId까지 동일한거면 진짜 동일한 사람일거라는 타당한 의문
            -> 12186개
            -> 제거하는게 맞아 보이지만. 사실 CustomerId만 빼놓고 본다면 내가 판단하기에는 무리가 있다.
            -> 멘토님, 강사님께 질문 후 내린 결론 : CustomerId가 데이터로써 오류로 판단.
            -> 12186을 중복치로 보고 제거하기에는 따라서 무리가 있다.


- 2_5.파생변수

    - 우리가 생각했을 때 이탈에 영향을 많이 끼칠 것 같은 것
        - 정수 : CreditScore(신용점수), Age(나이), Tenure(기간), EstimatedSalary(연봉)
        - 소희 : NumOfProducts(상품수), HasCrCard(카드보유여부),  CreditScore(신용점수), Age(나이), Tenure(기간)
    
            - 30대의 수가 많다, 나이가 높을수록 연봉이 올라가고 잔고가 많아질텐데 그럼 영향이 당연히 있지않을까
            - 이 은행이 이미 젋은 사람을 대상으로 한 상품, 서비스를 많이 보유하고 있지 않을까?
            - 그러면 자연적으로 나이가 상관이 없지 않을까?
            - Age가 상관이 없다면 이런 생각까지 갈 수 있을 것 같다.
    
    - 우리가 생각했을 때 영향을 많이 끼칠 것 같지않은것
        - 정수, 소희 : id, CustomerId, Surname, Balance(잔고), Geography(국가), HasCrCard(카드보유여부), IsActiveMember(활성 회원 여부 - 휴면계정)
            -> Geography : 프랑스의 수가 훨씬 많은게 분석 대상 은행이 프랑스 소유가 아닌가?
            -> 확인 결과 해당 데이터는 프랑스 은행에서부터 생긴 데이터
            -> 그럼 아무래도, 프랑스보다는 독일과 스페인의 이탈율이 더 높지않으려나?
            -> 그렇다면 이탈 결과에 좋은 영향을 줄 수도 있을 가능성이 있을수도 있다.
    
    
    - 영향 많이 끼칠 것 같은것에서 조합 만들기
    - 신용점수(CreditScore), 나이(Age), 기간(Tenure)
    - 연봉(EstimatedSalary), 상품수(NumOfProducts), 카드보유여부(HasCrCard)
        - 나이대 별 신용점수(CreditScore By Age)
        - 연령 별 기간 : Tenure By Age
        - 참여도 : NumOfProducts(1,2,3,4) + HasCrCard(0,1) + IsActveMember(0,1) -> 1, 2, 3, 4, 5, 6 : 낮을수록 적은 참여도
        - EstimatedSalary 별 (CreditScore * HasCrCard)
    
    - 영향 많이 끼칠 것 같은 것 + 끼칠 것 같지않은 것
    - Id, CustomerId, Surname
    - 정수 : Geography,HasCrCard, IsActiveMember
    - 소희님 : Balance
        - 잔고 별 신용점수 : CreditScore By Balance | Balance_group
        - 연령 별 잔고 : Balance By Age
        - 잔고 별 상품 수 : NumOfProducts By Balance | Balance_group

```






