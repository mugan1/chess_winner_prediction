# Chess Game Winner Prediction

## 주제

---

체스 게임 데이터를 활용한 승자 예측(분류)

---

## 기획 배경

---

- 체스(IBM사의 Deep Blue/1997), 바둑(Google사의 AlphaGo/2016) 등의 마인드 스포츠는 인공지능이 인간을 능가할 수 있는가에 대한 척도가 되었음
- 체스의 많은 전략과 수순 역시도 분석 가능한 "경우의 수"라는 관점 하에 데이터 분석 및 랭글링을 실시하여, 높은 정확도 구현뿐만 아니라 설명가능한 AI(Explainable AI)를 구현해보고 싶었음

---

## 목표

---

- 80% 이상 정확도를 지닌 체스 게임 승자 예측 모델 구현
- ELI5 / PDP PLOT / SHAP 등의 Library를 활용하여 예측 결과 해석

---

## 체스 게임 설명

---

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled.png)

- 가로축은 A-H, 세로축은 1-8 좌표로 기물의 진행 순서를 파악할 수 있음
- Opening 분류 1
    1. Game이 뒤에 붙는 경우는 그 오프닝이 매우 오래전부터 사용
    2. Defence : 수비적인 오프닝이 아닌 그 오프닝을 흑이 결정하는 전략
    3. Attack :공격적인 오프닝을 뜻할 때도 있지만 백이 그 오프닝을 결정하는 전략
    4. Gambit : 기물을 희생하며 시작하는 오프닝
- Opening 분류 2
    1. 오픈게임(Open Game) : e4 e5로 시작하는 오프닝
    2. 세미오픈게임(Semi-Open Game) : .e4에 대해 e5가 아닌 다른 수로 대응하는 오프닝
    3. 클로즈게임(Closed Game) : d4 d5로 시작하는 오프닝
    4. 세미클로즈게임(Semi-Closed Game) : d4에 대해 d5가아닌 다른수로 대응하는 오프닝
    5. 플랭크오프닝(Flank Opening) : 첫번째 수에서 e4나 d4를 두지 않는 오프닝

---

## 데이터 소개(kaggle - [https://www.kaggle.com/datasnaek/chess](https://www.kaggle.com/datasnaek/chess))

---

- id : 플레이어 아이디
- rated : 게임 방식(Rating/Casual) - Casual 게임은 rating에 영향을 받지 않음
- created_at : 게임 시작시간
- last_move_at : 게임 종료시간
- turns : 기물을 주고 받은 횟수
- victory_status : 승리 방식(체크메이트, 시간초과, 기권, 무승부)
- winner : 우승자
- increment_code : 게임 시간 룰
- white_id : 백 아이디
- white_rating : 백 rating 점수
- black_id : 흑 아이디
- black_rating : 흑 rating 점수
- moves : 기물의 이동 기록
- opening_eco : 오프닝 코드 넘버
- opening_name : 오프닝 이름
- opening_ply : 오프닝 이동 횟수

---

## 해결 과제

---

- 다중 클래스 분류 문제
- 흑백의 승자 분포는 비교적 균일하므로 Accuray를 1차적 평가지표로 설정
- ROC Curve와 AUC Score를 2차적 평가지표로 설정
- 기준 모델은 클래스 기준 범주 비율(백 / 약 49%)

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%201.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%201.png)

---

## 가설 설정

---

- 최종 모델의 결과는 기준 모델의 정확도보다 약 30% 이상 높을 것이다.
- 기물의 이동기록(moves)에서 Feature Engineering 을 통해 만들어진 Feature는 승자 분류에 높은 연관성이 있을 것이다.

---

## Feature Engineering

---

- rating_gap : white_rating(백 rating 점수) - black_rating(흑 rating 점수)
- new features from moves
    1. moves의 초반 기물 움직임에 따라 'Opening 분류 2'
    2. 'x'(기물을 잡은 기호)의 횟수에 따라 black_kill, white_kill 생성
    3. '+'(메이트 기호)의 횟수에 따라 black_mate, white_mate 생성

---

## 데이터 분석

---

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%202.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%202.png)

- Feature와 Target(winner)과의 상관관계
- Target과의 상관관계가 높은 Feature는 rating_gap, white(black)_mate, white(black)_kill 순이었음 → Feature Engineering으로 새롭된 만들어진 feature가 모델의 예측 성능 향상에 큰 영향을 끼칠 것으로 예상

---

## Modeling

---

[모델 검증 성능 비교](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/%E1%84%86%E1%85%A9%E1%84%83%E1%85%A6%E1%86%AF%20%E1%84%80%E1%85%A5%E1%86%B7%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%89%E1%85%A5%E1%86%BC%E1%84%82%E1%85%B3%E1%86%BC%20%E1%84%87%E1%85%B5%E1%84%80%E1%85%AD%20f4bd7e0cb81b4339a255e04b23758e62.csv)

- 모델은 RandomizedCV를 통해 최적화를 실시한 결과
- 검증 정확도가 제일 높은 모델은 XGB Boost 모델이었으며 5-Fold CV를 통해 일반화 검증 완료(평균 Score 0.85)
- XGB Boost 모델 Test Accuracy 0.84, Test AUC Score 0.89

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%203.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%203.png)

---

## XAI(eXplainable AI)

---

- Permutation Importance

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe.png)

- PDP Plot

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe%201.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe%201.png)

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(1).png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(1).png)

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(2).png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(2).png)

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(3).png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(3).png)

- PDP Interact Plot

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(4).png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/_(4).png)

- SHAP(승자를 백으로 예측한 확률 95%)

![Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%204.png](Chess%20Game%20Winner%20Prediction%203a491d4b9d4a4ff5ab0696d71a4923fe/Untitled%204.png)

- 분석 결과
    1. Permutation Importance는 모든 Feature가 Target에 어떻게 작용하는지 분석할 수 있는 도구이며, PDP는 개별 Feature가 Target에 미치는 작용을 분석하는 도구   
    2. SHAP은 각 Feature가 예측 결과에 얼마나 공헌했는지 알 수 있는 도구
    3. 공통적으로 가설대로 rating_gap(점수 차)와 moves feature에서 추출한 기물을 잡은 횟수(kill), 메이트 횟수(mate) 등이 예측의 주요 요인이 되었음
    4. 가설에 설정되어 있지 않은 turns(기물을 주고 받은 횟수)의 경우, 횟수가 많아질수록 무승부의 확률이 높아지기 때문에 무승부를 예측하는 데에 작용하는 주요인으로 해석됨

    ---

    ## 결론 및 기대효과

    ---

- 모델 중 XGB Boost 모델은 약 85% 정도의 정확도를 보였으며, 예측의 결정적 요인은 Feature Engineering을 통해 추출된 Feature임
- XAI Library를 통해 일관성 있는 결과 분석이 가능하였음
- 충분한 양질의 데이터가 주어진다는 가정 하에, 향후 승부를 가르는 방식의 마인드 스포츠 결과 예측의 정확도가 높아질 것이라 기대함