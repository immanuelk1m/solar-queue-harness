# Literature 검색과 수학 대안

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
| HSS/CIR 상승 | coronal hole area speed relation forecast 2025; high speed stream onset prediction EUV |
| 고속 셀의 진폭 과소추정 | extreme value regression time series underestimation peaks; conditional quantile forecast heavy tail |
| 리드별 transfer 붕괴 | distribution shift multi-horizon forecasting solar cycle nonstationarity |

### 논문 채택 체크리스트 (모두 예여야 채택)
- [ ] 메커니즘이 2단계에서 측정한 오차 성분 하나를 직접 겨냥한다.
- [ ] 필요한 입력이 contract.md의 허용 source로 인과적으로 얻어진다(origin 시각에 존재).
- [ ] 챔피언 레시피 위에 단일 변경으로 옮길 수 있다(PIVOT 제외).
- [ ] 닫힌 축이나 대기 큐와 메커니즘이 다르다.
- [ ] 워커 1개로 수 시간 안에 3-seed 전체 구간 run이 가능하다.
- [ ] 기대이득을 내부 수치로 계산할 수 있고 그 값이 ≥ 3%다.

검토한 논문은 모두 표로 남긴다: 제목 / 저자 / 연도 / 링크 / 메커니즘 / 채택·기각 사유(체크리스트의 어느 항목 때문인지).

## B. 수학 대안 작성 규칙 (채택 논문이 부족할 때)
일반론("복잡도를 줄인다", "robust하게 만든다")은 후보가 아니다. 아래 다섯 가지를 모두 쓴다.
1. **대상 성분 정의.** 챔피언 오차를 분해한다. 예: e = y − ŷ = (level 오차) + (timing 오차) + (진폭 오차) + 잔여. 겨냥하는 성분을 수식으로 정의하고 그 크기를 내부 수치로 쓴다(예: shock 창 오차 제곱 비중 약 50%, level 오라클 −50%).
2. **모델·추정량.** 새로 추정하는 양과 추정식을 쓴다. 예: 도착 시각 t̂ = f(ENLIL 예보 도착, CME 속도, 배경풍) + 보정, 손실 함수, 정규화.
3. **가정.** 인과성(입력의 가용 시각), 정상성, 독립성 등 결과를 좌우하는 가정을 명시하고, 깨졌을 때의 영향을 쓴다.
4. **기대이득 추정(작업 챔피언 대비).** 대상 성분이 차지하는 MSE × 회수 가능 비율(오라클이나 상관계수에서 추정)로 계산한다. 예: shock 창 SSE 50% × timing 회수율 r² ⇒ 전체 MSE 감소 ≈ 0.5 × r² × (오라클 이득 비율). 추정이 3% 미만이면 제안하지 않는다(SKILL 7단계 LOW-CONFIDENCE 예외 1건).
5. **첫 구현 단계.** 기준 레시피의 어느 함수, 어느 config에 무엇을 추가하는지 코드 수준으로 쓴다.

## C. 근거 표기
- LITERATURE 행: `[가설·근거]`에 논문 링크와 "우리 문제로 옮기는 방식"을 쓴다.
- MATH 행: `[가설·근거]`에 위 1–3을, `[기대이득과 계산]`에 4를, `[단일 변경]`에 5를 쓴다.
