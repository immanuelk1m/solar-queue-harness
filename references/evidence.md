# 확정 사실과 닫힌 축 (2026-10-13 기준 — 매 실행 시트 최신 결과로 재검증하고 갱신된 값을 우선한다)

## 기준 run (리더보드 전체 구간, 3-seed 평균 MSE (km/s)²)
| run | 내용 | MSE | llh | lhh | hlh | hhh | F1 |
|---|---|---|---|---|---|---|---|
| E0472 | SOTA. per-lead mixture XGB + residual analog + DONKI WSA-Enlil 5특징(ACE origin 시각) | 3281.3 | 5037 | 4598 | 3863 | 4174 | 0.283 |
| E1022/E0855 | E0472를 ACE obs+2h로 다시 만든 replay(신규 run의 실질 출발선) | 3421.8 | 5322 | 4747 | 4100 | 4313 | 0.268 |
| E0471 | E0147 레시피 + ENLIL | 3487.8 | 6101 | 4524 | 4146 | 4408 | 0.334 |

- 고속 셀 4개(*_*_high)가 오차가 가장 큰 곳이다. 저속 셀은 1100–2800 수준이다.
- ACE 2h 지연만으로 +140(+4.3%)이 생긴다. 예측 차이의 약 87%가 이 규칙에서 나온다.

## 헤드룸 (정보가 어디에 남아 있나)
- shock/CME 창(DONKI 기준)이 오차 제곱의 47–58%, HSS가 약 30%를 차지한다.
- 오라클: 사후 IPS 충격 도착 시각을 E0472에 넣으면(E0481) 2594.2(−21%)다. timing을 ±24h 안에서 완벽히 맞추면 −21%, origin별 72h 평균 level을 완벽히 맞추면 −50%다. → **회수 가능한 큰 성분은 이벤트 도착 시각과 72h 평균 level이다.** 둘 다 현재 입력으로는 잘 예측되지 않는다. 기존 입력으로 level·timing·amplitude·shape를 보정한 시도(E0609)에서 level r=0.19–0.39, timing·amplitude≈0, shape R²<0이었다.
- 정보가 있는 source(E0472 계열 CAL ablation, 제거 시 MSE 증가): recurrence +95%, ENLIL +30%(그중 since_arr_h +18%, n_pending +19%), CH +9.8%, EPAM +8.5%, plasma +7.5%, v_roll +4.3%. 실시간 timing source로 실제 개선을 낸 것은 ENLIL뿐이다(E0466 3574.7 → E0472 3281.3, −8%).
- 큰 개선은 매번 **새 정보**에서 나왔다(SUVI strip E0003 −3.9%, ENLIL −8%). 후처리·HPO에서는 나온 적이 없다.

## 기간 간 반전 (내부 창 결과를 믿으면 안 되는 이유)
- drop_strip: CAL −39.9% → 공식 E0857 3706.7(replay 대비 +8.3%).
- xgb_019: dev W1/W2 −4~−9% → 공식 E1024 3827.9(+11.9%).
- xgb_004: CAL +4.3% / SEARCH −28.7%. xgb_008: CAL +12.6% / SEARCH −19.3%.
- headroom audit: direct XGB가 selection/audit에서 −12%였지만 공식에서는 비유의.
→ 효과는 반드시 공식 전체 구간 3-seed로만 판정한다. 기대이득 근거로 내부 7일 창 수치를 쓰지 않는다.

## 닫힌 축 (같은 메커니즘이면 신규 후보로 인정하지 않는다)
- XGB HPO(depth/lr/trees/subsample 변형 20종, 캠페인 xgb_000–020): 공식 최선 xgb_003 E0858 3476.3(replay 대비 +1.6%).
- analog/KNN 잔차 보정의 K·거리·lookback 변경: 공식 E0856 3427.1(replay와 동일 수준).
- residual head·post-hoc correction·alpha/blend·bias correction: q104 SECC, q105 COV-BLEND, q108 CFC-AR1, q109 RLS 오라클, q174 RLLR, q152 RIHC, q154 CMCR, q151 FRAM 등.
- RLS expert bank: q99 E0695 4689.5(+43%).
- chronology robustness / worst-era / minimax / REx / random-effects / MAML 류: q141, q150, q184, q186, q185, q155, q156, q157 등.
- tree pruning / subset selection / Rashomon / post-prune: crbts72(MILP infeasible), rcp72, cspr72 계열.
- 신경망 backbone 교체: TabPFN E0545 3488.8, xLSTM 5106, NBEATSx 5984, Transformer/Crossformer 계열 모두 E0472보다 나쁨.
- 같은 source의 다른 요약: DONKI CME 분석 특징 E0475(+0.7%), DBM 도착 E0476(+1.5%, E0408 기준), NOAA Enlil 특징 E0241(+0.03%).
- 위 축의 threshold·게이트만 바꾼 재시도.

## 아직 제대로 시도되지 않은 축 (출발점 후보)
- 코로나그래프 CME 운동학(속도·폭·방향) + 개선된 drag-based 도착 모델. E0476은 단순 DBM 하나만 시도했다.
- GONG 자기장 지도 기반 PFSS/WSA 배경풍 + HUXt 전파. GONG 금지는 10-11에 해제됐고, HUXt는 아직 설치하지 않았다.
- ENLIL 예보의 시간 정렬 오차를 학습해 보정하는 도착 시각 보정(ENLIL은 유일하게 작동한 timing source다).
- 72h 평균 level(오라클 −50%) 자체를 별도 대상으로 예측하고 그 예측을 per-lead 모델에 조건으로 주는 구조.
- 공식 창까지 도달하는 다른 실시간 source(STEREO-A는 Earth보다 70° 앞서 있어 CIR 선행 정보가 없다. 이건 이미 확인됐다).

## 출처
리더보드 lb-eval 기록(`research/expos/graph.jsonl`), 캠페인 `research/e0472-ablation-hpo/campaign/trials-and-ablations.csv`, headroom audit(`leaderboard/store/headroom-audit/report.md`), 시트 `태양풍 연구`.
