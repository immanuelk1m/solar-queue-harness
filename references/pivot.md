# PIVOT 모드 (현재 연구 basin이 포화되었거나 중요한 미해결 가설을 기존 INCREMENT로 판별하기 어려울 때, 실행당 최대 1건)

PIVOT은 챔피언 레시피의 정보 표현, 예측 대상, 문제 정의 자체를 바꿀 수 있다. 그러나 PIVOT이라는 이유만으로 큐에 진입하지 않는다. **모든 예측 PIVOT도 SKILL.md의 A-GATE A1–A5를 통과해야 한다.** 결과는 INCREMENT와 같은 계약으로 판정한다. roll3 3폴드×3시드(대조는 챔피언 roll3 control)와 도착 선검정이고, 통과하면 공식 창 lb-eval로 1회 확인한다.

## 피봇으로 인정하지 않는 것 (기존 축의 변형)
- chronology split·regularization, worst-era/minimax/REx 류
- 기존 111개 특징의 단순 selection·pruning·weighting
- XGBoost/NN backbone 교체, loss 함수 변경만 하는 것(PIVOT 범위 한정 — 아키텍처 전체를 바꾸는 것은 아래 ARCH로 낸다)
- residual head, post-hoc correction, alpha/blending 변경, ensemble 가중·routing
- 기존 analog/KNN의 K·거리·lookback 변경
- 기존 특징 위에 attention/graph/gate만 추가하는 것
- 실패 실험의 threshold만 바꾼 재시도, 같은 정보를 다른 모델로 다시 학습하는 것
- 챔피언 레시피의 휴리스틱 값(lag, strip 수, 리드 경계, 멤버·가중, analog K·lookback, origin grid)을 바꾸는 것. 이것은 피봇도 INCREMENT도 아니고 금지다(`heuristics.md`).
- 같은 causal 입력을 유지한 trajectory diffusion / regime decoder / functional PCA / level-shape decomposition / target transform.
- arrival-time latent를 만들지만 실제 새 arrival/propagation 관측이 없는 모델.
- 현재 L1 상태를 더 잘 분류하는 것만으로 25–72h를 해결하려는 모델.
- source eruption 상태를 더 잘 표현하지만 source→Earth propagation bridge가 없는 모델.

위 후보는 연구 진단용 B급일 수 있지만 `태양풍 큐`의 예측 실험으로 쓰지 않는다.

## 피봇 층 (후보는 서로 다른 층에서 고른다)
- **정보 표현**: summary 특징이 버린 공간·시간 구조를 되살린다. 임계값·strip·스칼라 요약 없이 학습형 인코더로 직접 넣는다(예: SUVI/EUV 영상 시퀀스·자기장 지도·ENLIL field를 CNN/ViT/state-space 인코더로 표현). 사람이 정한 CH 경계·면적·strip 요약은 `heuristics.md` 금지 대상이다.
- **예측 대상**: 속도 대신 도착 시각, 지속 시간, 진폭, 이벤트 형태 같은 중간 물리량을 예측한다. 그 예측을 속도 궤적으로 바꾸는 단계까지 포함한다.
- **문제 분해**: HSS/CIR, CME, quiet를 서로 다른 생성 메커니즘으로 보고 따로 모델링한다. 분류는 인과적 입력만으로 한다.
- **물리 구조**: source → 전파 → 도착 → 속도 궤적을 명시적으로 모델링한다(PFSS/WSA/HUXt, drag-based).
- **새 정보 축**: 기존 source의 아직 안 쓴 raw·고차원 표현, 또는 필요하면 공식 창까지 얻을 수 있는 새 관측 source.
- **학습 단위**: origin×lead 회귀 대신 이벤트 궤적·episode·field 단위로 학습한다.

## 구조 전환 우선순위

오라클 headroom은 `어디에 문제가 있는가`를 정하는 근거다. **같은 입력을 다른 구조로 예측하는 것 자체는 해결책으로 보지 않는다.** PIVOT은 아래 순서로 우선한다.

1. **실제 heliospheric propagation observation/state**: CCOR/HI/IPS/ENLIL field/MHD state처럼 Earth 도착 전 전파 상태를 직접 관측하거나 계산한다.
2. **causal arrival-time ensemble/posterior**: 다중 operational forecast, ensemble HUXt/ENLIL 등 도착 불확실성을 직접 제공한다.
3. **pre-arrival shock/HSS precursor**: radio, energetic particle, GCR precursor처럼 L1 도착 전에 arrival phase에 정보를 준다.
4. **physics propagation operator/field**: source/boundary를 lead-aligned propagation state로 변환한다.

72h level decomposition, trajectory diffusion, event-mixture, latent regime, functional basis처럼 출력 표현이나 학습 단위만 바꾸는 후보는 새 propagation/arrival 정보가 함께 없으면 B급으로 기각한다.

오라클 headroom에 임의의 회수율을 곱해 사전 MSE 개선률을 만들거나 후보 통과 조건으로 사용하지 않는다.

각 PIVOT은 반드시 다음을 적는다.
- 현재 구분되지 않는 두 개 이상의 연구가설.
- 이 구조가 두 가설을 어떻게 구분하는지.
- 성공 시 다음 연구 행동.
- 실패 시 닫히거나 우선순위가 낮아지는 축.
- 관련 태양풍 물리 메커니즘.
- 해당 메커니즘을 표현하는 수학적 상태/추정량.

## 아키텍처 피봇 (ARCH 계보)
E0472 계열(per-lead XGBoost mixture + residual analog) 자체를 버리고 처음부터 다른 구조로 예측한다. 한 행으로 판정하지 않고 계보(최대 6단계)로 키운다(`diagnostics.md` 계보 규칙, `row-template.md` ARCH 양식). 판정은 챔피언 고정이다(v7): ARCH 출력을 챔피언 궤적 위 학습형 잔차·게이트로 결합해 roll3로 판정하고, 단독 점수는 계보 기록으로만 남긴다. 신경망은 OMNI2(SW-062) 사전학습 후 ACE 기간 미세조정한다. **ARCH도 A-GATE를 통과해야 하며 architecture-only 계보는 새로 열지 않는다.**

후보 계열 예시(각각 오늘의 병목과 연결해 쓴다):
- **사건 단위 도착 시각 분포 + 궤적 합성**: 고속 셀·shock 창의 timing 오차(오라클 −21%)를 직접 겨냥. 사건 검출(CME·CIR) → p(t_arr) · 진폭 예측 → 72h 궤적의 기대값 합성.
- **물리 전파 + 학습 hybrid**: GONG/PFSS·WSA 배경풍 → HUXt 또는 drag-based 전파 → L1 궤적, 그 위에 학습 보정. 72h level(오라클 −50%)과 HSS 도착을 구조적으로 표현. q77(단독 hydro/HUXt)이 나빴던 것은 학습 결합이 없었기 때문인지 계보 안에서 검증.
- **원시 시퀀스·영상 시퀀스 모델**: ACE 원시 시계열 + SUVI/EUV 영상 시퀀스를 직접 입력으로 72h 궤적을 동시 출력(seq2seq). 요약 특징이 버린 시간 구조를 복원. 이전 신경망 시도(TabPFN·xLSTM·NBEATSx·Crossformer)는 짧은 학습·단발이었으므로 같은 계열이라도 계보 예산으로는 미검증이다.
- **확률적 상태공간**: 배경풍 level·HSS 상태·CME 상태를 잠재 상태로 두고 관측(L1·원격)으로 필터링해 72h 예측 분포의 평균을 출력.

## PIVOT 행 작성 규칙
- `[단일 변경]` 대신 "무엇을 버리고 무엇을 새로 보는지"를 쓴다. 단, 그 밖의 부분은 챔피언 레시피를 그대로 쓴다고 명시한다.
- **정보 존재 여부 점검을 실험 안에 포함한다.** 예: 도착 시각 피봇이라면 현재 특징으로 예측한 도착 시각(B)과 새 표현으로 예측한 도착 시각(C)의 MAE·상관·recall을 먼저 보고한다. 이 점검은 진단 보고용이다. 결과가 나빠도 roll3 판정까지 진행한다(완수 규칙). "C가 B보다 못하면 중단" 같은 게이트는 쓰지 않는다.
- 성공하면 공식 MSE의 어느 셀·리드·이벤트 성분에 영향을 줄 것으로 예상하는지 방향을 적는다.
- 개선률을 사전에 임의로 정량화할 필요는 없다.
- 실패하면 닫히는 연구 축을 명시한다.
- 외부 논문의 개선률을 우리 MSE 기대치로 옮기지 않는다. 내부의 오차 질량, 오라클 headroom, 예측 가능성 격차로 근거를 댄다.


## A급 PIVOT이 없을 때

A등급 PIVOT이 당장 실행 불가능하면 B급 PIVOT을 대신 넣지 않는다. 목표 A급 PIVOT의 blocker를 제거하는 ENABLE-DATA / ENABLE-PHYSICS / INFO-TEST 해결 작업을 낸다(시트 데이터만 사용). 해결 작업은 완료되면 어떤 A급 예측 실험이 실행되는지 사전에 이름을 적어야 한다.
