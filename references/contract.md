# 평가 계약 (고정값 — 워커 규칙 AGENTS.md LB10:20261013 기준, 2026-10-13)

## 과제와 평가
- 과제: 매 시각 origin에서 L1 태양풍 속도를 1–72시간 앞까지 시간 단위로 예측한다. 단위는 km/s다.
- 공식 평가 창(리더보드): target 2026-06-01 00:00 ~ 2026-09-30 00:00 UTC, 2,833개 origin × 72h. 리더보드 scorer와 snapshot 정답을 쓴다.
- 판정은 `expos lb-eval`로만 한다. seed 0/1/2 세 개가 각각 `predictions_official_s{seed}.parquet`을 쓴다(모든 공식 origin×72h). 지표는 seed-mean total MSE (km/s)²이고, 고속 셀 4개(low_low_high, low_high_high, high_low_high, high_high_high)와 사건 F1을 함께 보고한다.
- smoke와 7일 블록 판정은 없다(LB10). CAL/SEARCH/W0–W2 같은 내부 창은 디버깅 참고용일 뿐이다.

## 동적 기준 선택
- 이 문서에는 특정 E id나 SOTA MSE를 현재 기준으로 고정하지 않는다.
- 매 실행 시작 시 `태양풍 연구` 최신 전체를 읽어 현재 SOTA와 현재 인과·평가 계약의 작업 챔피언을 확정한다.
- 현재 SOTA는 비교 가능한 공식 전체구간 lb-eval 결과 중 오라클·hindsight·계약 위반·무효 run을 제외한 유효 run을 대상으로 정한다.
- 평가 계약이 다른 run은 같은 순위에 섞지 않는다. 필요한 경우 `공식 SOTA`와 `현재 계약 작업 챔피언`을 별도로 기록한다.
- 작업 챔피언은 신규 실험이 따라야 하는 현재 계약 안에서 재현이 확인된 최선 run이다.
- 과거 E0472/E1022 등의 값은 historical evidence로는 사용할 수 있지만 현재 기준값으로 자동 사용하지 않는다.
- 후보 생성에는 고정 최소 예상 개선률을 두지 않는다.
- 실제 실험 결과의 통계적 의미는 seed별 paired 결과와 REPLICATE 결과를 사용해 판단한다.
- noise scale이 필요하면 최신 반복·재현 실험의 paired difference에서 동적으로 추정하고, 특정 MSE 값을 영구 상수로 두지 않는다.

## SOTA 갱신
- run 완료 시 그 시점의 `현재 SOTA`와 동일 평가 계약인지 먼저 확인한다.
- 동일 계약이라면 seed-mean MSE, seed별 방향, 고속 셀, F1 및 사전 정의 guardrail로 비교한다.
- 평가 계약이 다르면 `SOTA 갱신`이라고 부르지 않고 별도 comparator 결과로 기록한다.
- SOTA 후보가 되더라도 독립 재현 절차를 거친다.

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
- 현재 작업 챔피언의 코드 경로는 `태양풍 연구`의 최신 결과 및 연결된 experiment/E id에서 매 실행 확인한다.
- 특정 과거 레시피를 기본 champion으로 가정하지 않는다.
- 과거 E0472 계열 경로는 historical reproduction 또는 해당 계열 INCREMENT가 필요할 때만 참고한다: `~/sunrun/leaderboard/experiments/e0472_ablation_hpo/{recipe,runner,configs,campaign}.py`(config json 하나가 변형 하나, 공식 unit 예시 `~/sunrun/research/lbre/units/*__official__official.json`), E0472 원본 `experiments/d148_enlil_live/lean_enlil.py`(ACE origin 시각 사용, 2h 규칙 이전).
- 제출본 스크립트: GitHub `scripts/seongeun/<name>/<sha>.py`.
