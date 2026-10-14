# 확정 사실과 닫힌 축 — historical evidence

이 파일의 E id와 MSE는 과거 실험 근거다.
현재 SOTA·현재 작업 챔피언·현재 기준값을 이 파일에서 가져오지 않는다.
매 실행 `태양풍 연구` 최신 원본에서 다시 계산한다.

## 기준 run (리더보드 전체 구간, 3-seed 평균 MSE (km/s)²)
| run | 내용 | MSE | llh | lhh | hlh | hhh | F1 |
|---|---|---|---|---|---|---|---|
| E0472 | historical strong run. per-lead mixture XGB + residual analog + DONKI WSA-Enlil 5특징(ACE origin 시각) | 3281.3 | 5037 | 4598 | 3863 | 4174 | 0.283 |
| E1022/E0855 | E0472를 ACE obs+2h로 다시 만든 historical causal replay | 3421.8 | 5322 | 4747 | 4100 | 4313 | 0.268 |
| E0471 | E0147 레시피 + ENLIL | 3487.8 | 6101 | 4524 | 4146 | 4408 | 0.334 |

- 고속 셀 4개(*_*_high)가 오차가 가장 큰 곳이다. 저속 셀은 1100–2800 수준이다.
- ACE 2h 지연만으로 +140(+4.3%)이 생긴다. 예측 차이의 약 87%가 이 규칙에서 나온다.

## 2026-10-14 하네스 1차 결과 (E1022 위 단일 변경, 리더보드 3-seed)
| run | 변경 | MSE | E1022 대비 | 3 seed 모두 낮음 |
|---|---|---|---|---|
| E1198 | GONG polar-cap 자기장 요약 8개 | 3366.5 | −1.6% | 예(초기 3-seed 잠정 신호, 고속 셀 3개는 악화. 현재 상태는 최신 REPLICATE 결과와 시트에서 판정) |
| E1192 | ACE MAG/SWEPAM 변동 파워 15개 | 3377.3 | −1.3% | 예(고속 셀 개선) |
| E1194 | SWPC Type-IV 전파 폭발 | 3382.3 | −1.2% | 아니오 |
| E1149 | L72-COND(72h level 조건화) | 3387.7 | −1.0% | 예(llh 5322→4908) |
| E1157 | CME 쌍 사전조건 | 3394.8 | −0.8% | 예 |
| E1167 | GONG PFSS-WSA 특징 | 4024.3 | +17.6% | — |
- 위 네 스택 후보(E1198·E1192·E1149·E1157)의 결합은 `solar-comb-stack4-20261014`로 큐에 있다.

## 2026-10-14 연구 건강 진단 (하네스 1차 결과가 노이즈와 구별되지 않음)
- E1022 위 단일 변경 3-seed lb-eval 31건 중 근-귀무 23건에서 평균 차이 σ ≈ 37.6 MSE. 효과 0이어도 23번 중 최고치 기대값은 −72 MSE(−2.1%)인데, 관측 최고는 E1198 −55 MSE(−1.6%)다 → 지금까지의 개선은 선택 효과로 설명된다(H1).
- 결합 검증: stack4 arm A(E1214) = GONG polar + ACE 변동 + CME 사전조건 → 3411.6(−0.3%), 가산성 0.08(H3).
- 유형: 거의 모두 "E1022 특징 프레임에 정보원 특징 블록 추가"였고 확정 개선 0건(H5).
- 정렬: E1198의 개선은 저속 셀에서 나왔고 고속 셀 4개 중 3개가 악화됐다(H6).
- 독립 모델 재구현은 크게 악화: CSPR +136%, Crossformer +122%, CSNH +77%, TAPFN +58%, THR +35%, CL +20%(H7).
- 결론: 같은 XGBoost 레시피에 특징을 덧붙이는 방향은 포화 상태다. 남은 오차(고속 셀·shock 창, 72h level, 도착 시각)를 예측 대상·구조 차원에서 겨냥해야 한다(pivot.md "구조 전환 우선순위").

## 과거 아키텍처 계보 (ARCH 중복 검사 필수 대조표)
2026-10-06~10에 sun-nn 세션이 E0472와 독립인 신경망 계열을 방향(N1–N5)별로 돌렸다. ARCH 제안은 이 표와 먼저 대조한다. 같은 구조 계열이면 새 계보가 아니라 **이 계보의 연장**이다. 단계 수도 이어서 센다.
| 방향 | 구조 계열 | 리더보드 3-seed run 수 | 최고(E id) | E1022 대비 | 계보 규칙상 상태 |
|---|---|---|---|---|---|
| N5 | HSS-SpeedNet-SUVI (EUV CNN + 이진 CH 지도) | 17 | 4261.5 (E0479) | +25% | 종료(6단계 초과, 챔피언 미달) |
| N2 | SUVI 영상 인코더 (U-Net/CNN + temporal head) | 12 | 4655.1 (E0484) | +36% | 종료 |
| N4 | PROSWIN-SUVI (Swin-T + recurrence/B0/cycle, log-normal 분포 head) | 5 | 4987.9 (E0520) | +46% | 종료(E0393 7356.5→E0414 5227.9→E0463 5035.0→E0485 5114.3→E0520 4987.9, 마지막 2단계 <3%) |
| N1 | neuralforecast 계열 (NHITS/TiDE/PatchTST, ACE 원시 시퀀스) | 24 | 5106.6 (E0574) | +49% | 종료 |
| N3 | foundation 모델 (Chronos2/Moirai zero-shot·미세조정) | 10 | 5797.1 (E0259) | +69% | 종료 |
- 종료 계보를 다시 열려면 `[중복 검사]`에 그 계보의 최고 run을 적고, 무엇이 실질적으로 다른지 쓴다. 학습 규모만 늘리는 것은 다른 것이 아니다. 구조·학습 대상·입력 표현 중 하나 이상이 달라야 한다. 그리고 그 차이가 이 계보의 +25~69% 격차를 어떻게 메우는지 수치로 쓴다.
- 아직 계보가 없는 구조: 사건 단위 도착 시각 분포 + 궤적 합성, 물리 전파(HUXt/drag)+학습 hybrid, 확률적 상태공간(Kalman류), 기타 위 5계열과 구조가 다른 것.

## E0472 계열 레시피 안의 휴리스틱 (v5: 값 변경 금지, 학습형 대체만 허용)
챔피언 레시피는 아래 결정을 사람이 손으로 정했다. 이 값들을 바꾸는 행은 내지 않는다. 하나를 학습형으로 바꾸는 행은 INCREMENT 단일 변경으로 낼 수 있다(`heuristics.md`).
| # | 휴리스틱 | 범주 |
|---|---|---|
| 1 | rolling 통계, 27일 회귀 직관의 26/27/28일 고정 lag | a |
| 2 | Coronal hole 특징: 영상에 threshold를 건 뒤 공간 요약해 XGB 입력 | b |
| 3 | SUVI 영상을 경도 strip 17개로 요약(E0003) | b |
| 4 | WSA-Enlil field를 6개 스칼라로 요약 | c |
| 5 | D33 horizon 경계 1–6 / 7–24 / 25–72h | d |
| 6 | 멤버로 E0003-line / D33-hsplit / D45-lvlmean 세 개를 선택 | e |
| 7 | per-horizon convex mixture(neural gating 학습 없음) | e |
| 8 | analog residual 자체(“비슷한 과거 오차가 반복된다”는 가정) | f |
| 9 | analog 상태 96차원 벡터, 차원 표준화·동일 weight | f |
| 10 | K = 15, 이웃 uniform 평균, lookback 365일 | f |
| 11 | residual branch weight 0.2 | e |
| 12 | 3시간 origin grid(00/03/06/…) | f |

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
→ 효과는 반드시 공식 전체 구간 3-seed 및 필요한 재현으로 판정한다.
  내부 7일 창 수치는 병목 탐색·가설 생성에는 사용할 수 있으나 후보의 사전 성능 개선률을 만들어내는 근거로 사용하지 않는다.

## 닫힌 축 (같은 메커니즘이면 신규 후보로 인정하지 않는다)
- XGB HPO(depth/lr/trees/subsample 변형 20종, 캠페인 xgb_000–020): 공식 최선 xgb_003 E0858 3476.3(replay 대비 +1.6%).
- analog/KNN 잔차 보정의 K·거리·lookback 변경: 공식 E0856 3427.1(replay와 동일 수준).
- residual head·post-hoc correction·alpha/blend·bias correction: q104 SECC, q105 COV-BLEND, q108 CFC-AR1, q109 RLS 오라클, q174 RLLR, q152 RIHC, q154 CMCR, q151 FRAM 등.
- RLS expert bank: q99 E0695 4689.5(+43%).
- chronology robustness / worst-era / minimax / REx / random-effects / MAML 류: q141, q150, q184, q186, q185, q155, q156, q157 등.
- tree pruning / subset selection / Rashomon / post-prune: crbts72(MILP infeasible), rcp72, cspr72 계열.
- 신경망 backbone 교체: TabPFN E0545 3488.8, xLSTM 5106, NBEATSx 5984, Transformer/Crossformer 계열 모두 E0472보다 나쁨. 단, 대부분 짧은 학습(다수 < 3분, directive #14 이전)과 한 행짜리 단발 시도였다. 닫힌 것은 "빠른 backbone 교체"이고, 계보 예산으로 키우는 아키텍처 전체 피봇(ARCH)은 아직 열려 있다.
- 같은 source의 다른 요약: DONKI CME 분석 특징 E0475(+0.7%), DBM 도착 E0476(+1.5%, E0408 기준), NOAA Enlil 특징 E0241(+0.03%).
- ENLIL 특징 확장 6개 arm(E0518-F, 공식 창 3272–3307): 모두 E0472와 같거나 나쁨. 학습 보정도 표본 밖에서 기존 예측을 못 이김.
- 기존 입력으로 shock 도착 시각을 직접 학습(E0555, 89개 충격파): MAE 13.2h로 Enlil 12.5h보다 나쁨.
- SURF hydro / HUXt 단독 전파 예보(q77 E0665): recurrence보다 MSE가 크게 나쁨(31262·32857 vs 11515). 특징으로 결합하는 실험은 아직 없다.
- 위 축의 threshold·게이트만 바꾼 재시도.

### 닫힘 판정 규칙
- 축은 리더보드 전체 구간 3-seed lb-eval이나 사건 수가 충분한 직접 정보 검정(예: E0555)으로만 닫힌다.
- smoke·CAL/SEARCH/W 창·7일 블록·단일 seed, 실행 실패(rc≠0), 폐지된 스크린 게이트·사전조건 종료, 2026-10-06 이전의 옛 3구간 평가(dev/selection/audit, rocv)는 축을 닫지 않는다. 예: E0093은 rc=1 실패, E0136은 옛 3구간 평가다. 그런 축은 다시 제안할 수 있다(LBRE/RERUN 재평가 대기 중이면 중복).

## 아직 리더보드로 검증되지 않은 축 (출발점 후보 — 큐에 이미 있으면 중복)
- q193 HSS/CIR 도착·형상, q194 ENLIL 원시 전파장, q195 Carrington 공간장, q196 stream-interface 표적, q197 IPS 관측 피봇은 2026-10-14에 실행 없이 큐에서 제거됐다(옛 사양 일괄 정리). 결과가 없으므로 중복이 아니다. 구조 전환 후보로 다시 제안할 수 있고, 그때는 확정 챔피언 위 단일 구조 변경으로 쓴다.
- 코로나그래프 CME 운동학(속도·폭·방향)을 per-lead 모델의 **특징**으로 넣는 것. E0476은 단순 DBM 도착 시각 하나만 특징으로 썼다.
- GONG 자기장 지도·PFSS/WSA 배경풍 field를 **학습형 인코더 표현**으로 per-lead 모델에 넣는 것. 단독 전파 예보(q77)는 나빴지만, 결합은 검증된 적이 없다. 사람이 고른 스칼라 요약(E1167 PFSS-WSA 특징 +17.6%, E1198 polar-cap 요약 8개)은 v5부터 금지 형태다(`heuristics.md`).
- 72h 평균 level(오라클 −50%)을 별도 대상으로 예측하고 per-lead 모델에 조건으로 주는 구조.
- E0472 계열 레시피 안의 단일 변경 중 공식 창에서 아직 안 돌린 것: LBRE 재평가 결과가 나오면 스택 후보로 쓴다.
- 참고: STEREO-A는 Earth보다 70° 앞서 있어 CIR 선행 정보가 없다(확인됨).

## 출처
리더보드 lb-eval 기록(`research/expos/graph.jsonl`), 캠페인 `research/e0472-ablation-hpo/campaign/trials-and-ablations.csv`, headroom audit(`leaderboard/store/headroom-audit/report.md`), 시트 `태양풍 연구`.
