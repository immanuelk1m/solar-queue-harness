# 평가 계약 (고정값 — 워커 규칙 AGENTS.md LB10:20261013 기준, 2026-10-13)

## 과제와 평가
- 과제: 매 시각 origin에서 L1 태양풍 속도를 1–72시간 앞까지 시간 단위로 예측한다. 단위는 km/s다.
- 공식 평가 창(리더보드): target 2026-06-01 00:00 ~ 2026-09-30 00:00 UTC, 2,833개 origin × 72h. 리더보드 scorer와 snapshot 정답을 쓴다.
- 판정은 `expos lb-eval`로만 한다. seed 0/1/2 세 개가 각각 `predictions_official_s{seed}.parquet`을 쓴다(모든 공식 origin×72h). 지표는 seed-mean total MSE (km/s)²이고, 고속 셀 4개(low_low_high, low_high_high, high_low_high, high_high_high)와 사건 F1을 함께 보고한다.
- smoke와 7일 블록 판정은 없다(LB10). CAL/SEARCH/W0–W2 같은 내부 창은 디버깅 참고용일 뿐이다.

## 기준과 목표
- SOTA 고정 기준: **E0472 = 3281.307**(seed 3248.7/3300.4/3294.9). 고속 셀 5037/4598/3863/4174, F1 0.283.
- 목표: seed-mean ≤ 0.9 × SOTA = **2953.18**. lb-eval 출력 `goal.total_mse_10pct_vs_sota`로 확인한다.
- ACE+2h 인과 replay: E1022/E0855 = **3421.8**. 신규 run은 모두 이 규칙을 따르므로, 같은 레시피도 약 +140 불리하게 출발한다.
- **작업 챔피언**: ACE obs+2h 규칙을 지킨 lb-eval 최고값이다. 현재 E1022 3421.806이다. 점진 체인의 기준이고, 3% 게이트(약 103 MSE)도 이 값 기준이다.
- 노이즈: 약 90 MSE(seed 간 편차와 재현 편차). 3% ≈ 100 MSE가 의미 있는 최소 효과다.
- **SOTA 갱신 조건 #13/#13b(고정 SOTA 대비, lb-eval이 자동 판정):** 3-seed 평균과 모든 seed가 현재 최고보다 낮을 것, 고속 셀 4개의 seed-mean이 악화되지 않을 것, F1 ≥ champion F1. 통과하면 독립 critic 1회를 거친다.
- 자동 최고점(`lb-best`)에서 오라클 E0474/E0481은 제외된다.

## 인과 규칙 (위반은 무효)
- 학습은 target < 2026-05-29만 쓴다. 미래 정보, 공식 창 정답, 공식 창으로 고른 하이퍼파라미터를 쓰지 않는다.
- ACE/DSCOVR/Wind 실시간 태양풍은 관측 시각 + 2h ≤ origin일 때만 쓴다(ACE-LATENCY:20261012). 다른 외부 source는 문서화된 보수적 지연 상수로 인과성을 확보한다. 바이트 단위 과거 이력 입증은 요구하지 않는다.
- 라벨 성숙: target u는 u + 72h ≤ (학습 시점)일 때만 학습에 쓴다.
- poison check(미래·미성숙 라벨을 바꿔도 예측이 변하지 않음)를 통과해야 한다.

## 허용 데이터 source (2026-10-13)
- 기존 로컬 store: ACE 플라스마·IMF·EPAM, 27일 recurrence, coronal hole(CH) 특징, SUVI 경도 strip·Fe171 영상(proxy 규칙: S3 LastModified ≤ origin, obs_end ≤ origin−1h), DONKI WSA-Enlil(live API, `modelCompletionTime` ≤ origin).
- GONG 자기장 지도와 그 파생물(PFSS/WSA/HUXt)은 2026-10-11부터 다시 허용된다. NOAA SWPC WSA-Enlil도 허용된다. 공식 창까지 도달하는 대체 공개 source(OMNI, CDAWeb, SWPC archive)도 쓸 수 있다. source·버전·다운로드 날짜·해시는 기록한다.
- 새 source는 공식 창(2026-06~09)의 origin마다 그 시각에 얻을 수 있었어야 한다.

## 자원 (워커 1개 기준)
- CPU 3–4스레드에 고정된다. 트리·부스팅은 CPU, 딥러닝·파운데이션 모델은 GPU가 필수다(2장 공용, lease 대기).
- 워커 1개의 제한 시간은 12시간이다. 실패하면 디스패처가 최대 4회까지 이어서 재시도한다.
- 참고: E0472 계열 레시피의 공식 창 3-seed run은 `experiments/e0472_ablation_hpo` config + unit 방식으로 20–35분 걸린다.

## 코드 위치 (워커 로컬 경로)
- 챔피언 계열 레시피: `~/sunrun/leaderboard/experiments/e0472_ablation_hpo/{recipe,runner,configs,campaign}.py`. config json 하나가 변형 하나다. 공식 unit 예시는 `~/sunrun/research/lbre/units/*__official__official.json`.
- E0472 원본: `experiments/d148_enlil_live/lean_enlil.py`(ACE origin 시각 사용, 2h 규칙 이전).
- 제출본 스크립트: GitHub `scripts/seongeun/<name>/<sha>.py`.
