---
name: solar-queue-harness
description: 태양풍 72시간 속도 예측 연구(sunrun)의 다음 A등급 예측 실험 또는 그 실험을 가능하게 하는 해결 작업을 설계해 Google Sheet `태양풍 큐`에 추가하는 하네스. 시트 `태양풍 연구`/`태양풍 큐`와 GitHub 모델 스크립트를 읽어 현재 SOTA·작업 챔피언·병목·미해결 연구가설을 동적으로 확정한다. 25–72h propagation/arrival 정보를 새로 제공하는 예측 실험만 큐에 허용하고, 실행 가능한 A등급 후보가 없으면 data/physics/info blocker를 제거하는 해결 작업을 최소 1건 만든다. 태양풍 큐 추가, 다음 실험 제안, 연구 방향 진단을 요청받으면 사용한다.
---

# solar-queue-harness

**버전: v6.3 (2026-10-15).** 실행 결과의 첫 줄에 `solar-queue-harness v6.3`를 출력한다. 원본은 https://github.com/immanuelk1m/solar-queue-harness (main)이다. 업로드된 사본과 내용이 다르면 GitHub 원본을 따른다.

로컬 워커는 이 스킬이 `태양풍 큐`에 쓴 행 하나를 받는다. 예측 실험 행은 사양 그대로 구현해 리더보드 전체 구간에서 평가한다. ENABLE-DATA / ENABLE-PHYSICS / INFO-TEST / DATA-DISCOVERY 해결 작업 행은 사양에 적힌 산출물·완료 조건을 수행하며 SOTA/lb-eval 결과로 세지 않는다. 행의 `사양·지시` 칸이 곧 워커의 작업 지시다. 그래서 행 품질이 실험 품질을 결정한다.

**목표.** 고정된 SOTA E id, MSE 값, 목표 개선률을 스킬에 하드코딩하지 않는다.
매 실행마다 `태양풍 연구` 최신 원본 전체를 읽고 현재 SOTA와 현재 평가 계약에서의 작업 챔피언을 다시 확정한다.

이 스킬의 목적은 다음 두 가지다.
1. 현재 SOTA보다 재현 가능하게 더 나은 예측기를 찾는다.
2. 현재 핵심 병목을 실제로 해결하는 다음 행동을 항상 최소 1건 만든다. 특히 25–72h event phase / heliospheric propagation / arrival uncertainty를 더 식별할 수 있는 새 인과 정보를 우선한다.

**큐 품질 원칙.**
- 예측 실험은 아래 `A-GATE`를 모두 통과한 A등급만 `태양풍 큐`에 쓴다.
- 같은 causal 입력을 유지한 채 loss, target transform, output representation, latent-state parameterization, backbone, weighting, sampling, pruning, post-processing만 바꾸는 후보는 B/C등급으로 분류하고 예측 실험 큐에 쓰지 않는다.
- 단, 문제를 발견하고도 0건으로 끝내지 않는다. A등급 예측 실험이 당장 실행 불가능하면 그 A등급 실험의 blocker를 제거하는 `ENABLE-DATA`, `ENABLE-PHYSICS`, `INFO-TEST`, `DATA-DISCOVERY` 중 최소 1건을 큐에 쓴다.
- 정보이득은 A등급 후보끼리의 우선순위를 정하는 기준이다. 정보이득만으로 B/C등급 예측 실험을 큐에 승격시키지 않는다.

## A-GATE — 예측 실험 큐 진입 hard gate

예측 후보는 A1–A5를 모두 만족해야 한다.

### A1. 25–72h 병목 직접성
- 25–72h event arrival phase, HSS/CIR/CME propagation, shock arrival, 또는 미래 background-wind state 중 하나를 직접 더 식별해야 한다.
- 현재 L1 상태를 더 잘 분류하는 것만으로는 통과하지 않는다.

### A2. 새로운 인과 정보
다음 중 하나 이상이어야 한다.
- 새 원격/상류 관측: coronagraph, heliospheric imager, IPS, radio shock, energetic-particle/GCR precursor 등.
- 실제 propagation state/field: WSA/HUXt/ENLIL/MHD/SFT 등에서 lead에 따라 전파되는 물리 상태.
- 독립적인 arrival posterior: 다중 operational forecast, ensemble propagation, forecast revision trajectory 등.
- Earth 도착 전에 관측되는 precursor로서 문헌 또는 내부 검정에서 유의미한 lead-time 근거가 있는 상태.

아래 변경만으로는 새 정보로 인정하지 않는다.
- loss 또는 target transform
- trajectory basis/decomposition
- latent-state / diffusion / generative head / regime decoder
- model backbone
- sampling/origin weighting, feature weighting, pruning/selection
- post-processing, residual correction, ensemble/stacking만 변경

### A3. 조건부 추가정보
후보는 반드시 다음 질문에 한 문장으로 답해야 한다.
`기존 챔피언의 ACE + CH/SUVI + DONKI/ENLIL + recurrence를 이미 안다고 할 때도 무엇이 추가로 남는가?`
답이 "더 좋은 representation일 수 있다" 수준이면 탈락한다.

### A4. propagation bridge
사양에 반드시 아래 물리 경로를 구체적으로 쓴다.
`관측/상태 -> heliospheric propagation -> arrival phase -> L1 25–72h speed`
중간 propagation 상태 또는 pre-arrival precursor가 없는 `solar source -> ML -> future speed` 직결 가설은 원칙적으로 탈락한다.

### A5. 시간 선행성
새 정보가 event가 L1에 도착한 뒤에만 드러나는 정보라면 탈락한다.
25–72h 예측에 의미 있는 선행성이 있음을 문헌, provider timing, physical propagation equation, 또는 pre-cutoff information test로 근거화해야 한다.

### 등급
- **A**: A1–A5 모두 통과 → 예측 실험 큐 가능.
- **B**: 병목 관련 구조/표현 실험이지만 새로운 propagation/arrival 정보가 없음 → 예측 실험 큐 금지.
- **C**: weighting/HPO/target-transform/post-hoc/late L1-state/단순 결합 → 예측 실험 큐 금지.

B/C 후보는 마지막 출력의 `기각 후보`에만 기록한다.

## 0건 방지 — 해결 작업 fallback

A등급 예측 실험이 0건이면 B/C 실험을 억지로 넣지 말고 다음 순서로 최소 1건의 해결 작업을 만든다.

1. **ENABLE-DATA**: 이름이 정해진 A등급 후보를 막는 데이터 수집·causal availability·coverage·manifest blocker를 제거한다.
2. **ENABLE-PHYSICS**: 이름이 정해진 A등급 후보에 필요한 propagation model/field/operator의 최소 실행 기반을 구축한다.
3. **INFO-TEST**: 유망한 새 source/state가 기존 챔피언 조건부 residual에 추가 정보를 갖는지 pre-cutoff에서 직접 검정한다.
4. **DATA-DISCOVERY**: 위 셋도 만들 수 없을 때만, 현재 병목을 직접 관측할 새로운 upstream source를 찾고 causal feasibility를 판정한다.

해결 작업은 반드시 `[목표 A급 실험]`, `[현재 blocker]`, `[완료 조건]`, `[성공 시 다음 행동]`, `[실패 시 다음 행동]`을 가져야 한다. 막연한 문헌조사나 "데이터를 더 찾아본다"는 행은 금지한다.

**기준은 매 실행 동적으로 확정한다.**
- **현재 SOTA**: `태양풍 연구`의 리더보드 전체 구간 결과 중 오라클·hindsight·계약 위반·무효 run을 제외하고, 서로 비교 가능한 공식 평가 계약에서 가장 좋은 유효 seed-mean MSE를 가진 run으로 확정한다. 시트에 SOTA/채택 상태가 명시되어 있으면 함께 교차검증한다.
- 평가 계약이 서로 달라 직접 비교할 수 없는 run은 하나의 SOTA 순위에 억지로 섞지 않는다. 필요한 경우 `현재 공식 SOTA`와 `현재 인과 계약 작업 챔피언`을 별도로 적는다.
- **작업 챔피언**: 현재 신규 실험이 따라야 하는 평가·인과 계약 안에서 재현이 확인된 가장 좋은 run이다. 3-seed로만 기준을 넘은 run은 **잠정 챔피언**, REPLICATE(seed 3·4·5 재현) 6-seed 짝비교를 통과한 run은 **확정 챔피언**이다. 점진 체인의 [기준 레시피]는 확정 챔피언이다.
- E id와 MSE는 예시나 기본값으로 하드코딩하지 않는다. 반드시 시트 최신값에서 읽는다.
- 후보를 만들 때 예상 MSE 개선률의 최소 임계값을 두지 않는다. 예상 성능은 참고 정보일 뿐 큐 진입 조건이나 순위의 필수 항목이 아니다.
- 이하 '챔피언'은 모두 작업 챔피언을 뜻한다.

입력:
- 시트: https://docs.google.com/spreadsheets/d/1eqmgRn3qJH_44Bkv14piwzhADU2fWZL8-p-d6EzF-nw (탭 `태양풍 연구`, `태양풍 큐`)
- 데이터셋 manifest: https://docs.google.com/spreadsheets/d/1KWgNQPlajHWw7FLUMz3lr4rO2z7sxuxXGstD0-YjAw4 (SolarWind DB, 탭 `데이터셋 manifest`; 미러 `references/datasets.md`)
- 모델 스크립트: https://github.com/TLAB-HGU/sunrun-leaderboard/tree/main/scripts/seongeun

반드시 함께 읽을 파일: `references/contract.md`(평가 계약), `references/evidence.md`(확정 사실·닫힌 축), `references/row-template.md`(큐 행 양식), `references/literature.md`(검색·수학 대안 절차), `references/diagnostics.md`(1.5단계 연구 건강 진단 계산법), `references/heuristics.md`(손으로 정한 휴리스틱 금지 규칙), `references/datasets.md`(사용 중·수집 중·평가된 데이터셋 manifest). PIVOT 모드일 때만 `references/pivot.md`도 읽는다.

## 절차 (순서대로, 각 단계 출력을 사용자에게 보인다)

### 1. 상태 확정 — 표로 출력
시트 두 탭의 **최신 원본**을 처음부터 끝까지 읽는다. 일부 샘플만 읽고 판단하지 않는다. 표에 넣을 값:
- **현재 SOTA**: 위 동적 판정 규칙으로 E id, seed별 MSE, seed-mean, 평가 계약, 고속 셀, F1을 확정한다.
- **현재 작업 챔피언**: 신규 실험이 따르는 현재 인과·평가 계약 안에서 별도로 확정한다.
- SOTA와 작업 챔피언이 다르면 왜 다른지 한 줄로 설명한다.
- 이전 스킬 문서나 `evidence.md`에 기록된 과거 SOTA 값을 현재값으로 가정하지 않는다.
- **작업 챔피언 판정 규칙.** `태양풍 연구` 행 중 다음을 모두 만족하는 행에서 seed-mean MSE가 가장 낮은 것이다.
  1. lb-eval(리더보드 전체 구간) 3-seed 결과가 있다.
  2. 오라클이 아니다. E0474/E0481과 "ORACLE"·"hindsight" 표기 행은 제외한다.
  3. ACE obs+2h 규칙을 지켰다. 2026-10-12 이후 등록이거나 "ACE+2h", "causal replay" 표기가 있다. E0472 자체와 그 이전 run은 제외한다.
  4. 직전 작업 챔피언보다 평균이 **1.5% 이상** 낮고 모든 seed가 낮다. 1.5% 미만의 차이는 노이즈 안의 동률이므로 챔피언을 바꾸지 않는다.
  - 이 조건을 만족하는 run은 **잠정 챔피언**이다. 행 `REP|` 줄의 verdict=확정이 있으면 **확정 챔피언**이다. 확정 챔피언이 없으면 현재 계약 안의 기준 replay(현재 계약으로 다시 돌린 직전 최고 레시피)를 시트에서 찾아 확정 챔피언으로 적고, 그 근거를 쓴다. 표에 잠정·확정을 모두 적는다. 작업 챔피언의 E id, seed MSE, 리드대역(1–6/7–24/25–48/49–72h) MSE, 고속 셀 4개(llh/lhh/hlh/hhh), F1, 코드 경로를 적는다.
- 현재 SOTA 대비 작업 챔피언의 %도 함께 적는다(평가 계약이 같을 때만).
- **최근 lb-eval 결과 목록.** 최근 15건을 표로 적는다. 열은 E id, 큐 ID, 기준, 변경, seed-mean, 챔피언 대비 %다.
- **정체 카운터.** 마지막 챔피언 갱신 이후 lb-eval까지 끝난 행의 수다.
- **스택 후보.** 그 행의 기준 레시피 대비 평균 **−0.8% 이상** 낮고 **3개 seed 모두** 기준 레시피의 같은 seed보다 낮은 행이다. seed 방향이 모두 같으면 노이즈일 가능성이 낮다. 정보원이 서로 다른 후보끼리는 결합 시 가산될 수 있다. 각 후보의 E id, 기준, 감소량, 개선된 셀을 적는다.
- **결합 결과.** 이미 돌린 COMBINE 행(`solar-comb-*`)의 결과와 가산성(실제 감소 ÷ 단독 감소 합)을 적는다. 같은 재료 조합은 다시 제안하지 않는다.
- **큐 현황.** 대기 중인 행의 메커니즘 목록이다. 중복 검사에 쓴다.
- **ARCH 계보 표.** `references/evidence.md`의 "과거 아키텍처 계보"(N1–N5, 모두 종료)를 먼저 적고, 그다음 `solar-arch-*` 계보를 적는다. 계보마다 한 줄: 계보 ID, 단계 수/6, 계보 최고 MSE(E id), 직전 단계 대비 %, 확정 챔피언 대비 %, 상태(활성/종료/승격 대기/챔피언). 계보는 `solar-arch-<계보>-s<N>` 행과 결과 줄의 `lineage=`로 찾는다. 상태는 `references/diagnostics.md`의 계보 규칙으로 정한다.

수치는 시트에 적힌 값만 쓴다. 없는 값은 "시트에 없음"으로 적는다. 추정치를 사실처럼 쓰지 않는다.

### 1.5 연구 건강 진단 — 필수, 표로 출력
연구가 어디서 헛돌고 있는지를 **사용자가 알아채기 전에** 하네스가 찾아 큐에 반영하는 단계다. 시트 `태양풍 연구`의 lb-eval 행을 **코드(파이썬)로 파싱해** 계산한다. 결과 칸 첫 줄의 `LB|`·`REP|` 줄을 우선 쓰고, 없으면 자유 텍스트에서 seed MSE를 뽑는다. 기준 레시피(base)가 같은 행끼리만 비교한다. 출력 표의 열은 신호 / 수치 / 판정(발동·미발동) / 큐 조치다. 계산 방법은 `references/diagnostics.md`를 따른다.

| 신호 | 계산 | 발동 조건 | 큐 조치 |
|---|---|---|---|
| H1 선택 효과 | 같은 base 위 최근 N개 lb-eval의 평균 차이 d. σ = sd(d). 귀무 기대 최고치 = −σ·z_N (z: N=10→1.54, 20→1.87, 30→2.04, 50→2.25) | 관측 최고 개선이 귀무 기대 최고치보다 작다(덜 낮다) → `NULL-LIKE` | 잠정 챔피언·스택 후보는 재현 전까지 체인에 쓰지 않는다. 같은 유형의 단일 변경은 이번 실행 최대 1건. 구조 전환 행(PIVOT)을 정체 카운터와 무관하게 최소 1건 |
| H2 미재현 | 잠정 챔피언과 스택 후보 중 `solar-rep-*` 결과도, 대기 중인 REPLICATE 행도 없는 것 | 1건 이상 | `REPLICATE` 행을 맨 먼저 추가한다. 후속 연구 방향에 미치는 영향이 큰 후보를 우선하고, 동률이면 관측된 개선량이 큰 순으로 한다(실행당 최대 2건). 6-seed 짝 판정: 평균 d < 0이고 t ≤ −2, 또는 6개 모두 음수일 때만 확정 |
| H3 결합 비가산 | 각 COMBINE의 가산성 = 실제 감소 ÷ 단독 감소 합 | < 0.3 | 그 재료들을 "노이즈 가능"으로 표시한다. 같은 재료 재결합 금지. 재료가 속한 유형을 H5 계산에서 "확정 0"으로 센다 |
| H4 정체 | 확정 챔피언 갱신 이후 lb-eval 수 | ≥ 8 | PIVOT 허용 |
| H5 유형 포화 | 최근 15개 행을 intervention 유형으로 분류: 특징 블록 추가 / 후처리·잔차 / 하이퍼파라미터 / 예측 대상 변경 / 구조·학습 단위 / 새 정보원 | 한 유형이 ≥ 60%이고 그중 확정 개선 0건 | 그 유형을 이번 실행에서 금지한다. 다른 층에서만 후보를 낸다 |
| H6 오차 정렬 | 최근 개선 행(잠정 포함)이 줄인 셀·리드와, 확정 챔피언의 오차 질량(고속 셀 4개·shock 창) 비교 | 개선이 저속 셀에서만 나고 고속 셀은 악화 | 이번 실행의 모든 신규 행은 고속 셀·shock 창 오차를 직접 겨냥하고, 그 성분에 대한 가설을 판별하도록 설계한다 |
| H7 반복 고장 | 같은 메커니즘의 인프라 실패 ≥ 2회, 또는 처음부터 새로 짠 한 행짜리 모델이 +20% 이상 악화 | 해당 | INCREMENT·PIVOT에서는 그 메커니즘을 base 위 단일 변경 형태로만 다시 제안한다. 한 행짜리 독립 모델 재구현은 금지한다. ARCH 계보는 이 금지에서 제외한다. 단발 실패는 예산 없이 한 번에 만든 결과이기 때문이다 |
| H8 아키텍처/계보 고착 | 최근 15행을 base 계보별로 묶었을 때 가장 많이 쓰인 한 계보의 비율(계보는 시트의 parent/base 관계와 구조 계열로 계산), 활성 ARCH 계보 수 | 비율 ≥ 80%이고 H1 또는 H5가 발동했고 활성 계보 < 2 | architecture-only ARCH를 자동 생성하지 않는다. A-GATE를 통과하는 새 propagation/arrival 정보가 결합된 ARCH만 허용한다. 없으면 0건 방지 fallback으로 해결 작업을 낸다. |

진단 표 아래에 **"이번 실행이 바꾸는 연구 방향" 3줄**을 쓴다. 각 줄에 근거 신호를 단다. 예: "특징 블록 추가 유형을 금지한다(H1·H3·H5)."

### 2. 병목 진단 (내부 수치만)
- 챔피언의 오차 질량이 어디에 몰렸는지 적는다: 리드대역, 고속 셀, shock/CME·HSS 이벤트 창, 정상 구간.
- `references/evidence.md`의 헤드룸·ablation 사실과 1단계 최신 결과를 합친다. **아직 회수 가능한 오차**를 3줄로 쓴다.
  - 근거: 오라클은 개선되지만 실제 입력으로는 안 되는 성분, ablation상 정보가 있는 source, 기간 간 transfer가 깨진 구성.
- 닫힌 축 목록을 갱신한다. evidence.md 목록에 시트 최신 결과를 더한다.
- **닫힘 판정 규칙.** 다음 결과만 축을 닫는다.
  - 리더보드 전체 구간 3-seed lb-eval에서 개선이 없었다.
  - 사건 수가 충분한(≥ 50 사건) 직접 정보 검정에서 기존 대비 정보가 없었다. 예: E0555 도착 시각 MAE.
- 다음은 축을 닫지 않는다.
  - smoke, CAL/SEARCH/W0–W2 같은 내부 창, 7일 블록, 단일 seed 결과.
  - 실행 실패(rc≠0, 산출물 없음).
  - 폐지된 스크린 게이트(0.6095류)나 사전조건 때문에 종료된 행.
  - 2026-10-06 이전의 옛 3구간 평가(dev/selection/audit, rocv).
- 이런 축은 같은 메커니즘이라도 다시 제안할 수 있다. 다만 그 행이 이미 `LBRE-*`/`RERUN-*`로 재평가 대기 중이면 중복이다.

### 3. 모드 결정
- **1.5단계 진단이 먼저다.** 발동한 신호는 후보 탐색 방향을 바꾸지만 A-GATE를 우회할 수 없다.
  - H2 발동 → REPLICATE 행을 먼저 쓴다.
  - H1·H5 발동 → A-GATE를 통과하는 구조 전환(PIVOT) 후보를 우선 찾는다. 없으면 해결 작업 fallback으로 전환한다.
  - H3 발동 → 해당 재료의 COMBINE은 쓰지 않는다.
  - H6 발동 → 모든 행이 고속 셀·shock 창을 겨냥한다.
  - 활성 ARCH 계보도 다음 단계가 A-GATE를 통과할 때만 ARCH-STEP을 낸다. 같은 입력의 architecture-only 다음 단계라면 더 진행하지 않고 해결 작업 fallback 또는 다른 A급 후보로 전환한다.
  - H8 발동 → architecture-only 새 계보는 열지 않는다. A-GATE를 통과하는 새 propagation/arrival 정보가 포함된 ARCH만 허용한다. 동시 활성 계보는 최대 2개다. **큐에 대기 중인 `solar-arch-*-s1` 행도 활성으로 센다.** 이미 2개면 새 ARCH를 내지 않는다.
  - ARCH 후보는 과거 계보 표(N1–N5)와 같은 구조 계열이면 새 계보로 내지 않는다. 종료 계보 재개 조건(evidence.md)을 충족할 때만 그 계보의 다음 단계로 낸다.
- 기본은 **A-GATE를 통과한 INCREMENT/PIVOT**다. 확정 챔피언 레시피에 단일 변경 1개를 얹는다.
  - 챔피언 레시피의 휴리스틱을 학습형 구성요소로 바꾸는 것만으로는 큐 진입 사유가 아니다. A2의 새 propagation/arrival 정보가 함께 없으면 B/C로 기각한다. 새 휴리스틱을 만들거나 그 값을 바꾸는 행은 어떤 모드로도 내지 않는다.
- COMBINE은 재료가 모두 A-GATE를 통과했고 각각 독립 full-lb/REPLICATE에서 유효 신호가 확인된 경우에만 허용한다. B/C 재료의 STACK/COMBINE은 금지한다.
- 다음 중 하나면 **PIVOT**을 최대 1건 허용한다(`references/pivot.md`). 나머지 행은 여전히 INCREMENT다.
  - 정체 카운터 ≥ 8.
  - 현재 intervention 층에서 병목에 대한 새로운 정보를 주거나 중요한 가설을 판별할 비중복 후보를 찾지 못했다.
  - 현재 후보들이 모두 같은 정보원·표현·모델 계열의 국소 변형에 머물러 연구 불확실성을 충분히 줄이지 못한다.

### 4. Literature (웹 검색 필수)
`references/literature.md` 절차를 따른다.
- 2단계 병목마다 질의를 만든다. 4~6개를 병렬로 검색한다.
- 원문을 직접 열어 메커니즘을 확인한다. 초록만 보고 판단하지 않는다.
- 채택 조건을 모두 만족해야 한다.
  - 메커니즘이 우리가 측정한 병목에 대응한다.
  - 기존 데이터와 허용 source로 인과적으로 구현할 수 있다(contract.md).
  - 닫힌 축이나 대기 큐와 메커니즘이 다르다.
  - 챔피언 레시피 위의 단일 변경으로 옮길 수 있다. ARCH·ARCH-STEP은 예외이고, 대신 구조 전체를 재현할 수 있어야 한다(공개 코드·하이퍼파라미터·학습 비용).
- 검토한 논문은 채택 여부와 무관하게 모두 표로 남긴다. 열은 제목·저자·연도·링크·메커니즘·채택/기각 사유다.
- 논문 benchmark의 개선률을 우리 MSE 기대치로 옮기지 않는다.
- 채택할 논문이 부족하면 **수학적 + 물리적 방법론**을 직접 설계한다.
- 수학적으로는 추정 대상, 상태변수, 손실/우도, 제약식 또는 식별 가정을 명시한다.
- 물리적으로는 `source → propagation → arrival/regime → L1 speed trajectory` 중 어느 과정에 개입하는지 명시하고, 태양풍 전파·자기장 연결·CME/HSS/CIR·플라스마 상태 등 해당 메커니즘과 연결한다.
- generic ML 아이디어만으로는 부족하다. 왜 태양풍의 현재 병목과 연결되는지 물리적으로 설명할 수 있어야 한다.

### 5. 수학적·물리적 방법론 (literature가 불충분할 때)
채택 가능한 논문이 0편이거나 남은 후보 수가 부족하면, 직접 방법론을 설계한다. `references/literature.md`의 "수학·물리 대안 작성 규칙"을 따른다.
- 대상 오차 성분을 수식으로 정의한다.

후보마다 다음을 모두 작성한다.

1. **미해결 가설**
   - 현재 어떤 사실을 모르는지 쓴다.
   - 예: `25–72h timing 실패가 interplanetary propagation 정보 부족 때문인가?`
2. **물리 메커니즘**
   - 관측량이 어떤 물리 상태를 측정하고 그 상태가 미래 L1 속도에 어떻게 연결되는지 쓴다.
   - 가능하면 `source → propagation → arrival → trajectory` 형태로 쓴다.
3. **수학적 모델**
   - 상태변수·추정량·조건부 분포·손실·제약을 식으로 정의한다.
   - 예: p(t_arr | X), E[Y_h | X,z], state transition, propagation equation 등.
4. **반증 가능한 예측**
   - 이 가설이 맞다면 어떤 중간 지표가 어떻게 바뀌어야 하는지 사전에 쓴다.
   - 이 가설이 틀리다면 어떤 관측 결과가 나와야 하는지도 쓴다.
5. **정보이득과 의사결정**
   - 성공했을 때 다음에 무엇을 할지 쓴다.
   - 실패했을 때 어떤 축을 닫거나 우선순위를 낮출지 쓴다.
   - 성공·실패가 모두 다음 연구 선택을 바꾸지 못한다면 정보이득이 낮은 후보로 본다.
6. **첫 구현 단계**
   - 코드 수준에서 한 번에 무엇 하나를 바꿀지 쓴다.

예상 MSE 개선률 계산은 선택 사항이다. 계산할 근거가 없으면 억지로 수치를 만들지 않는다.
"복잡도 감소", "pruning", "robustness" 같은 일반론만으로는 후보가 아니다. 반드시 측정된 오차 성분과 연결한다.

### 6. 후보 게이트 (예측 실험은 A-GATE + 아래 항목을 모두 통과해야 큐에 쓴다)
0. **A-GATE**
   - A1–A5 전부 PASS여야 한다.
   - B/C이면 이후 게이트를 평가하지 않고 기각한다.
1. **연구 가치 / 정보이득**
   - 해결하려는 미해결 가설 또는 병목이 명확해야 한다.
   - 후보가 기존 입력·표현·모델과 비교해 어떤 새로운 정보를 제공하는지 명시해야 한다.
   - 최소 두 개의 경쟁 설명을 구분할 수 있어야 한다.
   - 성공 결과와 실패 결과 각각이 후속 큐에 어떤 다른 결정을 만들지 사전에 적어야 한다.
   - 성공해도 실패해도 연구 방향이 전혀 달라지지 않는 실험은 우선순위를 낮춘다.
   - 예상 MSE 개선률은 필수 항목도, hard gate도 아니다.
   - 정보이득은 A-GATE 통과 후보끼리의 순위를 정하는 데만 쓴다. 정보이득만으로 B/C 후보를 승격시키지 않는다.
   - 오라클 headroom과 오차 질량은 `어디에 문제가 있는가`를 정하는 근거로는 사용하지만, 이를 임의의 회수율과 곱해 후보를 통과·탈락시키지 않는다.
   - 다른 메커니즘의 과거 실패는 이 후보의 반증이 아니다. 예: E0609의 기존 입력 기반 보정 실패는 새 정보원 후보를 반증하지 않는다.
2. INCREMENT는 **단일 변경**이어야 한다. 같은 행에 변경 두 개를 섞지 않는다. COMBINE은 이미 측정된 변경끼리만 섞는다.
3. **인과 가용성.** 각 origin에서 쓰는 모든 입력이 관측 시각(+문서화된 지연, ACE는 +2h) ≤ origin이어야 한다. 학습 target은 2026-05-29 이전이어야 한다.
4. **비용.** 3-seed 전체 구간 run이 워커 1개(CPU 3–4스레드, DL은 GPU 1장)로 수 시간 안에 끝나야 한다.
5. **중복 없음.** 시트 두 탭의 Parent/가설/intervention/target/representation과 메커니즘이 달라야 한다. 이름만 다른 같은 메커니즘은 탈락이다.
6. **병목과의 직접성 또는 탐색적 독립성**
   - exploitation 후보는 현재 병목을 직접 줄일 근거가 있어야 한다.
   - exploration 후보도 새로운 propagation/arrival information을 포함해야 한다.
   - 표현·예측대상·architecture만 다른 것은 exploration으로 인정하지 않는다.
   - 단순히 모델 이름이나 threshold만 바꾼 것은 exploration으로 인정하지 않는다.
7. **휴리스틱 금지.** `references/heuristics.md`의 자가 점검 3문항을 통과해야 한다. 사람이 값을 정한 lag·rolling 창·임계값·공간 분할(strip·sector)·수동 스칼라 요약·리드/구간 경계·멤버 선택·고정 가중·analog K/lookback/거리·origin grid를 새로 넣거나 기존 값을 바꾸는 후보는 탈락이다. 표준 물리량은 문헌 정의를 인용하면 허용한다. 결과를 행의 `[휴리스틱 검사]`에 적는다.

**ARCH·ARCH-STEP 게이트**(위 2·4번 대신 적용, 1·3·5·6·7번은 그대로):
- 귀납 편향 근거: 왜 이 구조가 고속 셀·shock 창의 도착 시각과 72h level 오차를 per-lead XGBoost보다 잘 잡는지 쓴다.
- 근거 종류: literature 실측(구조 전체가 비슷한 과제에서 낸 성능) 또는 수학적 근거.
- 학습 예산: DL은 GPU, seed당 ≥ 30분 또는 early stop 수렴, epoch별 curves 저장. 3-seed가 워커 12시간 안에 끝날 것.
- 계보 계획: s1(최소 동작 구조)부터 s6까지의 단계 계획과 계보 목표(현재 확정 챔피언을 재현 가능하게 넘는 독립 구조를 찾거나, 해당 architecture 가설의 가능성과 한계를 판별한다). 첫 단계는 확정 챔피언보다 나빠도 된다.
- 단독 판정: E0472 계열의 예측·잔차를 입력으로 쓰거나 E0472 계열과 블렌딩·앙상블하지 않는다. 데이터 로더와 인과 특징 빌더 재사용은 허용한다.

### 7. 큐 반영
- A-GATE와 나머지 게이트를 통과한 A등급 예측 후보를 우선순위 순으로 **최대 4건** `태양풍 큐`에 추가한다. 4번째 자리는 ARCH-STEP 전용이다. ARCH-STEP이 없으면 최대 3건이다.
- **0건으로 끝내지 않는다.** A등급 예측 후보가 없으면 B/C 예측 실험을 추가하지 말고 `0건 방지 — 해결 작업 fallback`에 따라 최소 1건의 ENABLE-DATA / ENABLE-PHYSICS / INFO-TEST / DATA-DISCOVERY 행을 추가한다.
- 해결 작업은 반드시 이름이 정해진 목표 A급 실험 또는 현재 가장 큰 phase/propagation 불확실성과 연결되어야 한다.
- LOW-CONFIDENCE는 A-GATE를 우회하는 예측 실험 모드로 사용하지 않는다.
  - 열은 `ID`, `항목`, `사양·지시`, `갱신 시각`이다.
  - ID 형식: `solar-<모드소문자>-<짧은이름>-YYYYMMDD`. 예: `solar-inc-cmeeta-20261014`.
  - `사양·지시`는 `references/row-template.md` 양식을 그대로 채운다.
- **동시 실행 방지.** 큐에 쓰기 직전에 `태양풍 큐`를 다시 읽는다. 그 사이 다른 실행이 같은 ID, 같은 대상의 REPLICATE, 같은 계보 행을 추가했으면 그 후보는 쓰지 않는다. 활성 ARCH 계보 수도 다시 센다.
- **행 추가 방식.** 반드시 시트 끝에 append(values.append, INSERT_ROWS)로 추가한다. 행 번호를 계산해 특정 행에 덮어쓰지 않는다. 다른 세션이 같은 시각에 행을 추가하고 있을 수 있다. 추가 후 탭을 다시 읽어, 내 ID가 있는지와 바로 앞뒤 행의 ID가 추가 전과 같은지 확인한다. 다른 행을 덮어썼으면 그 행을 원래 내용으로 복구하고 사용자에게 알린다.
- 기존 행은 수정·삭제하지 않는다.
- 예측 행을 쓰기 직전 `[A-GATE]` 섹션의 A1–A5가 모두 PASS인지 다시 확인한다. 하나라도 FAIL이면 append하지 않는다.
- 해결 작업은 `[해결 작업 게이트]`의 목표 A급 실험·blocker·완료 조건·성공/실패 후속 행동이 모두 채워졌는지 확인한다.
- REPLICATE 행은 6단계 게이트 대상이 아니다. 1.5단계 H2로만 판단한다. 최대 행 수에는 REPLICATE도 포함된다.
- 마지막에 사용자에게 다음을 보여 준다: 1.5단계 진단 표와 방향 3줄, 추가한 행 요약 표(ID, 모드, 기준, 변경, 근거 종류, 겨냥 가설, 정보이득/결정 효과, 휴리스틱 검사), 기각한 후보와 사유, 검토 논문 표.

## 사양에 절대 쓰지 말 것
워커는 사양을 그대로 따른다. 다음 문구는 행을 실패시키므로 넣지 않는다.
- 한 실험에 전체 목표를 요구하는 스크린 게이트: C/A·O/A 비율 임계(예: 0.6095), "MSE<2000", late 감소율 임계, gain ≥ 1281, C vs B ≥ 10%, 오라클 ceiling 미달 시 중단.
- smoke·7일 단일 블록·단일 seed로 판정하거나 종료하라는 지시. 판정은 전체 구간 3-seed lb-eval로만 한다.
- 사전 조건 대기("캠페인 종료 후", "champion pin 후")와 사전 critic PASS 요구.
- 제출, 업로드, freeze, official-score 지시.
- 공식 창 정답이나 미래 정보를 쓰는 설계(오라클은 진단 용도로도 큐 행으로 만들지 않는다).
- 손으로 정한 휴리스틱 지정(`references/heuristics.md`): "26/27/28일 lag", "N시간 rolling", "임계값 X로 CH 영역 추출", "경도 strip N개", "ENLIL을 k개 스칼라로 요약", "1–6/7–24/25–72h 경계", "멤버 3개를 convex 가중", "branch weight 0.2", "K=15·lookback 365일", "3h grid" 같은 고정 값 지시와, 그 값을 바꿔 보라는 지시.
