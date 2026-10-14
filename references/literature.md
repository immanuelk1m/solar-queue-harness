# Literature 검색과 수학·물리 대안

## A. 웹 검색 절차
1. 2단계에서 확정한 병목마다 검색 질의 1–2개를 만든다. 총 4–6개를 한 번에 병렬로 검색한다. 태양풍 논문으로 한정하지 않는다. 방법론이 옮겨질 수 있으면 다른 분야 논문도 허용한다.
2. 기간은 최근 2년(현재 날짜 기준) 논문을 우선한다. 수학적 기초가 되는 고전 방법(drag-based model, Kalman/state-space, quantile 회귀 등)은 연도 제한 없이 인용할 수 있다.
3. 후보 논문은 원문(PDF/HTML)을 열어 메커니즘, 입력, 대상, 평가 방식을 확인한다. 초록만 보고 채택하지 않는다.
4. 같은 논문 판정: 제목, 제1저자, arXiv id 중 하나라도 같으면 같은 논문이다.
   - 그 논문의 메커니즘이 이미 리더보드 전체 구간 lb-eval로 평가됐거나 지금 큐에 대기 중이면 제외한다.
   - 시트에서 검토만 됐거나 smoke·실행 실패로 끝난 논문은 제외하지 않는다(evidence.md 닫힘 판정 규칙). 이 경우 이전 행 ID를 `[중복 검사]`에 적는다.

### 병목별 질의 예시 (현재 병목에 맞게 바꿔 쓴다)
| 병목 | 질의 예시 |
|---|---|
| 이벤트 도착 시각(timing) | CME arrival time machine learning coronagraph kinematics 2025; drag-based model ensemble arrival correction; ENLIL arrival error correction machine learning |
| 72h 평균 level | solar wind background speed WSA PFSS GONG machine learning 2025; HUXt ensemble solar wind forecast data assimilation |
| HSS/CIR 상승 | end-to-end EUV image solar wind speed deep learning 2025; learned temporal attention solar wind high speed stream onset |
| 손으로 정한 휴리스틱의 학습형 대체 | learnable lag selection time series attention; mixture of experts learned gating multi-horizon forecasting; learned metric retrieval-augmented time series forecasting |
| 고속 셀의 진폭 과소추정 | extreme value regression time series underestimation peaks; conditional quantile forecast heavy tail |
| 리드별 transfer 붕괴 | distribution shift multi-horizon forecasting solar cycle nonstationarity |

### 논문 채택 체크리스트 (모두 예여야 채택)
- [ ] 메커니즘이 2단계에서 측정한 오차 성분 하나를 직접 겨냥한다.
- [ ] 필요한 입력이 contract.md의 허용 source로 인과적으로 얻어진다(origin 시각에 존재).
- [ ] 챔피언 레시피 위에 단일 변경으로 옮길 수 있다(PIVOT·ARCH 제외).
- [ ] (ARCH) 구조 전체를 재현할 수 있다: 공개 코드 또는 충분한 구조·하이퍼파라미터 기술, 비슷한 과제의 실측 성능, 우리 GPU·12시간 예산 안의 학습 비용.
- [ ] 닫힌 축이나 대기 큐와 메커니즘이 다르다.
- [ ] 우리 문제로 옮겼을 때 손으로 정한 lag·창·임계값·분할·스칼라 요약·경계·가중이 필요 없다(`heuristics.md`). 논문이 그런 값을 쓰면 그 부분을 학습형으로 바꿔 옮길 수 있어야 한다.
- [ ] 워커 1개로 수 시간 안에 3-seed 전체 구간 run이 가능하다.
- [ ] 이 방법이 테스트하는 **미해결 연구가설**을 한 문장으로 쓸 수 있다.
- [ ] 성공과 실패가 서로 다른 후속 연구 결정을 만든다.
- [ ] 기존 후보와 다른 정보원·표현·물리상태·예측대상·학습단위 중 적어도 하나를 새로 검증한다.
- [ ] 태양풍 문제에 옮길 때 물리적으로 어떤 경로를 통해 L1 속도에 영향을 주는지 설명할 수 있다.

논문의 benchmark 개선률은 후보 통과 조건으로 사용하지 않는다.
우리 문제에서 예상 개선률을 신뢰성 있게 계산할 수 없는 것은 기각 사유가 아니다.

검토한 논문은 모두 표로 남긴다: 제목 / 저자 / 연도 / 링크 / 메커니즘 / 채택·기각 사유(체크리스트의 어느 항목 때문인지).

## B. 수학·물리 대안 작성 규칙 (채택 논문이 부족할 때)
일반론("복잡도를 줄인다", "robust하게 만든다")은 후보가 아니다. 아래 여섯 가지를 모두 쓴다.
1. **대상 성분 정의.** 챔피언 오차를 분해한다. 예: e = y − ŷ = (level 오차) + (timing 오차) + (진폭 오차) + 잔여. 겨냥하는 성분을 수식으로 정의하고 그 크기를 내부 수치로 쓴다(예: shock 창 오차 제곱 비중 약 50%, level 오라클 −50%).
2. **물리 메커니즘.** 관측 또는 latent state가 미래 태양풍에 영향을 주는 물리 경로를 적는다. 가능하면 `solar source / current L1 state → heliospheric propagation or magnetic connectivity → event/regime arrival → 72h speed trajectory` 형태로 쓴다. CME이면 kinematics/drag/shock propagation, HSS이면 CH/자기장 연결/solar rotation, plasma이면 compression·entropy·beta·composition 등 실제 상태변수와 연결한다.
3. **모델·추정량.** 새로 추정하는 양과 추정식을 쓴다. 예: p(t_arr|X), E[y_h|X,z], state transition, drag equation, conditional trajectory, 도착 시각 t̂ = f(ENLIL 예보 도착, CME 속도, 배경풍) + 보정, 손실 함수, 정규화.
4. **가정과 반증 조건.** 인과성(입력의 가용 시각), 정상성, 식별 가능성 등 결과를 좌우하는 가정을 명시하고, 깨졌을 때의 영향을 쓴다. 가설이 맞다면 관측돼야 할 중간 지표와, 틀리다면 나와야 할 결과를 적는다.
5. **정보이득 / 결정 효과.**
   - 성공하면 어떤 가설을 강화하고 다음에 무엇을 할지.
   - 실패하면 어떤 가설을 약화하거나 축을 닫을지.
   - 이 실험이 없으면 구분할 수 없는 두 경쟁 설명이 무엇인지.
   - 예상 MSE 개선값은 신뢰성 있게 산출 가능할 때만 보조 정보로 적는다. 필수값이 아니다.
6. **첫 구현 단계.** 기준 레시피의 어느 함수, 어느 config에 무엇을 추가하는지 코드 수준으로 한 가지만 쓴다.

## C. 근거 표기
- LITERATURE 행: `[가설·근거]`에 논문 링크와 "우리 문제로 옮기는 방식"을 쓴다.
- MATH/PHYSICS 행: `[가설·근거]`에 대상 성분·물리 메커니즘·수학적 추정량·반증 조건(위 1–4)을 적고, `[연구 가치와 정보이득]`에 성공/실패 시 의사결정 변화(5)를, `[단일 변경]`에 6을 쓴다.
