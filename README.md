### Project Title

체스게임 승자예측 프로젝트

### Overview

- 기간  |  2021. 02 ~ 2021. 02
- 담당 파트 |  개인프로젝트
- 플랫폼 |  Python, Tensorflow, Colab notebook

### Background 

1. 체스(IBM사의 Deep Blue/1997), 바둑(Google사의 AlphaGo/2016) 등의 마인드 스포츠는 인공지능이 인간을 능가할 수 있는가에 대한 척도가 되었음
2. 체스의 많은 전략과 수순 역시도 분석 가능한 "경우의 수"라는 관점 하에 데이터 분석 및 랭글링을 실시하여, 높은 정확도 구현뿐만 아니라 설명가능한 AI(Explainable AI)를 구현해보고 싶었음

### Goal

1. 80% 이상 정확도를 지닌 체스 게임 승자 예측 모델 구현
2. ELI5 / PDP PLOT / SHAP 등의 XAI Library를 활용하여 예측 결과 해석

### Chess Rules

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/084342d0-eb02-4833-a6af-31dabc7b4c1d" alt="text" width="number" />
</p>

1. 가로축은 A-H, 세로축은 1-8 좌표로 기물의 진행 순서를 파악할 수 있음
2. Opening 분류 Ⅰ
    - Game이 뒤에 붙는 경우는 그 오프닝이 매우 오래전부터 사용
    - Defence : 수비적인 오프닝이 아닌 그 오프닝을 흑이 결정하는 전략
    - Attack :공격적인 오프닝을 뜻할 때도 있지만 백이 그 오프닝을 결정하는 전략
    - Gambit : 기물을 희생하며 시작하는 오프닝
3. Opening 분류 Ⅱ
    - 오픈게임(Open Game) : e4 e5로 시작하는 오프닝
    - 세미오픈게임(Semi-Open Game) : .e4에 대해 e5가 아닌 다른 수로 대응하는 오프닝
    - 클로즈게임(Closed Game) : d4 d5로 시작하는 오프닝
    - 세미클로즈게임(Semi-Closed Game) : d4에 대해 d5가아닌 다른수로 대응하는 오프닝
    - 플랭크오프닝(Flank Opening) : 첫번째 수에서 e4나 d4를 두지 않는 오프닝
      
### Dataset

```

1. kaggle : https://www.kaggle.com/datasnaek/chess
2. id : 플레이어 아이디
3. rated : 게임 방식(Rating/Casual) - Casual 게임은 rating에 영향을 받지 않음
4. created_at : 게임 시작시간
5. last_move_at : 게임 종료시간
6. turns : 기물을 주고 받은 횟수
7. victory_status : 승리 방식(체크메이트, 시간초과, 기권, 무승부)
8. winner : 우승자
9. increment_code : 게임 시간 설정
10. white_id : 백 아이디
11. white_rating : 백 rating 점수
12. black_id : 흑 아이디
13. black_rating : 흑 rating 점수
14. moves : 기물의 이동 기록
15. opening_eco : 오프닝 코드 넘버
16. opening_name : 오프닝 이름
17. opening_ply : 오프닝 이동 횟수```
```

### Theories

1.  최종 모델의 결과는 기준 모델의 정확도보다 약 30% 이상 높을 것이다.
2. 기물의 이동기록(moves)에서 Feature Engineering 을 통해 만들어진 Feature는 승자 분류에 높은 연관성이 있을 것이다
   
### Feature Engineering

1. rating_gap : white_rating(백 rating 점수) - black_rating(흑 rating 점수)
2. new features from moves
    - moves의 초반 기물 움직임에 따라 'Opening 분류 Ⅱ' 방식으로 변환
    - 'x'(기물을 잡은 기호)의 횟수에 따라 black_kill, white_kill 생성 
    - '+'(메이트 기호)의 횟수에 따라 black_mate, white_mate 생성
  
      
### EDA

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/2f3cb885-9fad-4f2b-b89e-89139ff8ed00" alt="text" width="number" />
</p>

1. 다중 클래스 분류 문제
2. 흑백의 승자 분포는 비교적 균일하므로 Accuray를 1차적 평가지표로 설정
3. ROC Curve와 AUC Score를 2차적 평가지표로 설정
4. 기준 모델은 클래스 기준 범주 비율(백 / 약 49%)

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/04b09ad3-09b4-4bd4-80a5-e8063f22f1ea" alt="text" width="number" />
</p>

- Target과의 상관관계가 높은 Feature는 rating_gap, white(black)_mate, white(black)_kill 순이었음 → Feature Engineering으로 새롭된 만들어진 feature가 모델의 예측 성능 향상에 큰 영향을 끼칠 것으로 예상
  
### Modeling

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/52f8cf64-b177-4772-9973-639d54cfae2f" alt="text" width="number" />
</p>

1. RandomizedCV를 통해 최적화를 실시
2. 검증 정확도가 제일 높은 모델은 XGB Boost 모델이었으며 5-Fold CV를 통해 일반화 검증 완료(평균 Score 0.85)

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/b851a0a1-0461-4529-8d88-afbc224e7787" alt="text" width="number" />
</p>

### XAI(eXplainable AI)

1. Permutation Importance

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/1f938ff5-aa2d-4d2a-9abf-80925a096bac" alt="text" width="number" />
</p>

2. PDP Plot

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/22deff41-3674-48bd-8b2b-9a330519f88f" alt="text" width="number" />
</p>


3. PDP Interact Plot

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/80272f41-c623-41a5-97c9-4b843e2217d5" alt="text" width="number" />
</p>

4. SHAP(승자를 백으로 예측한 확률 95%)

<p align="center">
  <img src="https://github.com/mugan1/music_transcription/assets/71809159/0775858f-443d-49c8-801c-3403c029993c" alt="text" width="number" />
</p>

### Conclusion

1. 총평
    - 모델 중 XGB Boost 모델은 약 85% 정도의 정확도를 보였으며, 예측의 결정적 요인은 Feature Engineering을 통해 추출된 Feature임
    - XAI Library를 통해 일관성 있는 결과 분석이 가능하였음
   
3. 소감 및 기대효과
    - 충분한 양질의 데이터가 주어진다는 가정 하에, 향후 승부를 가르는 방식의 마인드 스포츠 결과 예측의 정확도가 높아질 것이라 기대함
