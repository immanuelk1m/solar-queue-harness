# 연구 건강 진단 계산법 (SKILL 1.5단계)

목적: 개선처럼 보이지만 노이즈·선택 효과인 결과에 체인을 쌓지 않는다. 같은 유형의 시도를 반복하지 않는다. 실제 오차가 있는 곳으로 방향을 돌린다.

## 0. 데이터 만들기
`태양풍 연구` 탭 전체를 표로 읽고, 리더보드 3-seed lb-eval이 있는 행만 남긴다. 행마다 다음을 뽑는다.
- `E`, `base`(그 행의 기준 레시피 E id), `mse_seed[k]`(seed별), `mean`, `cells`(llh/lhh/hlh/hhh), `type`(아래 6유형), `queue_id`.
- 우선 결과 칸 첫 줄의 `LB| E=... base=... seeds=... mse=a/b/c mean=... d_base=... cells=... type=...`를 파싱한다. 재현 결과는 `REP| target=... base=... d=... mean_d=... t=... verdict=...`를 파싱한다.
- 이 줄이 없으면 자유 텍스트에서 seed MSE 세 개와 기준 E id를 뽑는다. 뽑지 못하면 그 행은 제외하고, 제외 수를 표에 적는다.
- base의 seed별 MSE는 base 행에서 가져온다. E1022: seed 0/1/2 = 3409.80/3420.45/3435.17, 고속 셀 llh/lhh/hlh/hhh = 5322/4747/4100/4313.

```python
# df: 컬럼 E, base, s0, s1, s2, mean, llh, lhh, hlh, hhh, type, queue_id, ts
import numpy as np, pandas as pd
def diag(df, base_id, base_seeds, N=30):
    g = df[df.base == base_id].sort_values('ts').tail(N).copy()
    g['d'] = g['mean'] - np.mean(base_seeds)
    g['all_lower'] = (g[['s0','s1','s2']].values < np.array(base_seeds)).all(axis=1)
    sigma = g['d'].std(ddof=1)
    z = np.interp(len(g), [10, 20, 30, 50], [1.54, 1.87, 2.04, 2.25])
    null_best = -sigma * z
    obs_best = g['d'].min()
    return dict(N=len(g), sigma=sigma, null_best=null_best, obs_best=obs_best,
                null_like=bool(obs_best > null_best), table=g)
```

## H1 선택 효과
- σ는 같은 base 위 최근 N개(최대 30) 결과의 평균 차이 d의 표준편차다. 큰 악화(+20% 초과, 대개 구현 실패)는 σ 계산에서 빼고 따로 센다.
- 효과가 0이어도 N번 시도하면 최고치는 약 −σ·z_N이 나온다. 관측 최고치가 이 값보다 덜 낮으면 `NULL-LIKE`다. 이때 지금까지의 "개선"은 우연으로 설명된다.
- 2026-10-14 기준값: base E1022, N=23(근-귀무 행), σ ≈ 37.6 MSE, 귀무 최고치 ≈ −72 MSE(−2.1%), 관측 최고 E1198 −55 MSE(−1.6%) → 발동.

## H2 미재현
- 잠정 챔피언 = 확정 챔피언 대비 평균 −1.5% 이상이고 3 seed 모두 낮은 run.
- 스택 후보 = 평균 −0.8% 이상이고 3 seed 모두 낮은 run.
- 이 중 `REP|` 결과가 없고 큐에 `solar-rep-<E>` 행도 없는 run마다 REPLICATE 후보를 만든다. 감소량이 큰 순으로 실행당 최대 2건이다.
- 6-seed 짝 판정: d_k = MSE(후보,k) − MSE(base,k), k=0..5. 확정 조건은 mean(d) < 0이고 t = mean(d)/(sd(d)/√6) ≤ −2, 또는 d_k가 6개 모두 음수일 것이다. 새 seed 3개의 평균 d가 0 이상이면 "우연"으로 기록한다.

## H3 결합 비가산
- COMBINE 행마다 가산성 = (base − 결합 결과) ÷ Σ(base − 각 재료 결과)를 구한다. 0.3 미만이면 발동한다.
- 2026-10-14 기준값: stack4 arm A(E1214, GONG polar + ACE 변동 + CME 사전조건) = 10.2 ÷ 126.8 ≈ 0.08 → 발동. arm B(+L72-COND) 결과가 나오면 갱신한다.

## H5 유형 포화
- 유형 분류는 `[단일 변경]`이 무엇을 바꾸는지로 정한다.
  - 특징 블록 추가: base 특징 프레임에 열 추가.
  - 후처리·잔차: base 예측 위 보정 head, blend, analog.
  - 하이퍼파라미터.
  - 예측 대상 변경: 속도 대신 도착 시각·level·진폭 등을 예측하고 속도로 환원.
  - 구조·학습 단위: 2단계 구조, 이벤트 궤적·episode 단위, 물리 전파 결합.
  - 새 정보원(새 관측 자체를 처음 들이는 것): 특징으로만 넣으면 "특징 블록 추가"로 센다.
- 최근 15개 행 중 한 유형이 60% 이상이고 그 유형의 확정 개선이 0이면 발동한다.
- 2026-10-14 기준값: E1022 위 행 대부분이 "특징 블록 추가"이고 확정 0 → 발동.

## H6 오차 정렬
- 잠정 이상 개선 행마다 고속 셀 4개의 변화를 본다. 개선이 저속 셀에서 나오고 고속 셀 4개 중 2개 이상이 악화되면 "오정렬"이다.
- 확정 챔피언의 오차 질량은 고속 셀과 shock/CME 창에 몰려 있다. evidence.md: shock 창이 오차 제곱의 47–58%.
- 2026-10-14 기준값: E1198은 고속 셀 4개 중 3개 악화(hhh 4313→4397, hlh 4100→4268, lhh 4747→4898) → 발동.

## H7 반복 고장
- 같은 큐 메커니즘에서 rc≠0 run이 2회 이상이거나, 처음부터 새로 짠 독립 모델이 base 대비 +20% 이상이면 발동한다.
- 2026-10-14 예: CSPR +136%, CSNH +77%, TAPFN +58%, Crossformer q82 +122%.

## 출력 예시
| 신호 | 수치 | 판정 | 큐 조치 |
|---|---|---|---|
| H1 | N=23, σ=37.6, 귀무 최고 −72, 관측 최고 −55 | 발동 | 특징 추가 INCREMENT ≤1, PIVOT ≥1 |
| H2 | 잠정 1(E1198, 재현 대기), 후보 E1192·E1149·E1157·E1208 | 발동 | REPLICATE 2건 |
| … | … | … | … |

방향 3줄 예시: (1) "E1022 위 특징 블록 추가는 이번 실행에서 1건 이하로 줄인다(H1·H3·H5)." (2) "잠정 결과 2건을 seed 3·4·5로 재현한다(H2)." (3) "고속 셀 오차를 겨냥한 level/도착 시각 구조 전환 1건을 연다(H1·H6)."
