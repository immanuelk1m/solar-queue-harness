# PIVOT 모드 (정체 카운터 ≥ 8이거나 점진 후보가 없을 때, 실행당 최대 1건)

PIVOT은 챔피언 레시피의 정보 표현, 예측 대상, 문제 정의 자체를 바꾼다. 결과는 INCREMENT와 같은 계약으로 판정한다. 리더보드 전체 구간 3-seed lb-eval이고, 대조는 챔피언이다.

## 피봇으로 인정하지 않는 것 (기존 축의 변형)
- chronology split·regularization, worst-era/minimax/REx 류
- 기존 111개 특징의 단순 selection·pruning·weighting
- XGBoost/NN backbone 교체, loss 함수 변경만 하는 것
- residual head, post-hoc correction, alpha/blending 변경, ensemble 가중·routing
- 기존 analog/KNN의 K·거리·lookback 변경
- 기존 특징 위에 attention/graph/gate만 추가하는 것
- 실패 실험의 threshold만 바꾼 재시도, 같은 정보를 다른 모델로 다시 학습하는 것

## 피봇 층 (후보는 서로 다른 층에서 고른다)
- **정보 표현**: summary 특징이 버린 공간·시간 구조를 되살린다(예: SUVI/EUV 영상의 CH 경계·면적 시계열, 자기장 지도).
- **예측 대상**: 속도 대신 도착 시각, 지속 시간, 진폭, 이벤트 형태 같은 중간 물리량을 예측한다. 그 예측을 속도 궤적으로 바꾸는 단계까지 포함한다.
- **문제 분해**: HSS/CIR, CME, quiet를 서로 다른 생성 메커니즘으로 보고 따로 모델링한다. 분류는 인과적 입력만으로 한다.
- **물리 구조**: source → 전파 → 도착 → 속도 궤적을 명시적으로 모델링한다(PFSS/WSA/HUXt, drag-based).
- **새 정보 축**: 기존 source의 아직 안 쓴 raw·고차원 표현, 또는 필요하면 공식 창까지 얻을 수 있는 새 관측 source.
- **학습 단위**: origin×lead 회귀 대신 이벤트 궤적·episode·field 단위로 학습한다.

## PIVOT 행 작성 규칙
- `[단일 변경]` 대신 "무엇을 버리고 무엇을 새로 보는지"를 쓴다. 단, 그 밖의 부분은 챔피언 레시피를 그대로 쓴다고 명시한다.
- **정보 존재 여부 점검을 실험 안에 포함한다.** 예: 도착 시각 피봇이라면 현재 특징으로 예측한 도착 시각(B)과 새 표현으로 예측한 도착 시각(C)의 MAE·상관·recall을 먼저 보고한다. 이 점검은 진단 보고용이다. 결과가 나빠도 전체 구간 lb-eval까지 진행한다(LB10 완수 규칙). "C가 B보다 못하면 중단" 같은 게이트는 쓰지 않는다.
- 성공하면 공식 MSE로 어떻게 이어지는지(어느 셀·리드의 오차가 얼마나 줄어야 하는지)를 수치로 쓴다.
- 실패하면 닫히는 연구 축을 명시한다.
- 외부 논문의 개선률을 우리 MSE 기대치로 옮기지 않는다. 내부의 오차 질량, 오라클 headroom, 예측 가능성 격차로 근거를 댄다.
