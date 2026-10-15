# 데이터셋 manifest (태양풍)

원본은 Google Sheet [SolarWind DB](https://docs.google.com/spreadsheets/d/1KWgNQPlajHWw7FLUMz3lr4rO2z7sxuxXGstD0-YjAw4/edit) 탭 `데이터셋 manifest`다. 이 파일은 그 시트의 미러다(2026-10-14 기준). 시트를 열 수 있으면 시트를 우선한다.

쓰는 법:
- 후보 행이 쓰는 데이터는 아래 표의 ID(SW-xxx)와 로컬 경로로 지정한다. 표에 없는 새 source는 `references/contract.md`의 허용 규칙과 인과 규칙을 따른다.
- 인과(가용 시각) 규칙 칸을 그대로 지킨다. 규칙이 비어 있거나 "증빙 없음"이면 행에 보수적 지연 상수를 적는다.
- "평가했지만 미채택" 데이터로 다시 행을 낼 때는 그 lb 점수를 근거로 적고, 무엇이 달라지는지 밝힌다.
- lb 점수는 시트 `태양풍 연구`에서 뽑은 seed 평균 근사치다. 판단은 매 실행 시트 원본으로 다시 확인한다.

## 사용 중 (챔피언 계열 입력과 원천)

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-001 | ACE 실시간 hourly (수집기 원본) | 2000-08-01 ~ 현재 | `Postgres spaceweather.solar, ~/spaceweather-collector/data, gdrive:spaceweather/solar.csv` | 관측 시각 + 2h ≤ origin (ACE-LATENCY:20261012) | 모든 ACE 입력의 원천 |  |
| SW-002 | ACE hourly stage (sun-db) | 2022-08-22 ~ 현재 | `~/.local/state/sundb/stage/ace_solar/` | 관측 + 2h | alfven72, alphahe72 MAG, mambats, aceturb72 |  |
| SW-003 | ACE frozen snapshot | 2022-08-22 ~ 2026-10-03 | `~/sunrun/leaderboard/store/ch-breakthrough-v2/upload/raw-ace-snapshot/` | 관측 + 2h | E0472/E1022 레시피(e0472_ablation_hpo/recipe.py allowed_paths) |  |
| SW-004 | ACE 결측 보간 속도 (학습 타깃·입력) | 2022-08-23 ~ 2026-10-02 | `~/sunrun/leaderboard/store/ch-v1/ace.parquet` | 관측 + 2h; 2026-06-01 이후 타깃은 채점에 쓰지 않음 | E0472/E1022/E1198 입력·타깃 |  |
| SW-005 | Coronal hole (CH) hourly | 2022-08-22 ~ 2026-10-01 | `~/sunrun/leaderboard/store/ch-v1/ch-hourly.parquet (+masks/)` | SUVI proxy 규칙 | E0472/E1022/E1198 CH 그룹 (ablation +251 MSE) |  |
| SW-006 | SUVI Fe195 경도 strip | 2022-08-22 ~ 2026-10-01 | `~/sunrun/leaderboard/store/suvi-fusion/suvi-hourly-v2.parquet` | S3 LastModified ≤ origin, 로컬 바이트 = S3, obs_end ≤ origin−1h | E0472/E1022/E1198 strip (ablation +135) |  |
| SW-007 | SUVI hourly 메타데이터 | 2022-08-22 ~ 2026-10-01 | `~/.local/state/sundb/stage/suvi_hourly/` | SUVI proxy 규칙 | E0472 레시피 allowed_paths |  |
| SW-008 | GOES-18 SUVI L1b Fe195 원본 | 2022-08-22 ~ 현재 | `~/sunrun/sun-img/data/g18/, solar_g18.duckdb` | SUVI proxy 규칙 | CH·strip·euvwave·dim·chmorph·DINOv3/V-JEPA 원천 |  |
| SW-009 | DONKI WSA-Enlil 아카이브 | 2022-07 ~ 2026-06 | `~/sunrun/leaderboard/store/headroom-audit/donki/` | modelCompletionTime ≤ origin | E0472/E1022/E1198 ENLIL 특징 (~2026-05) |  |
| SW-010 | DONKI WSA-Enlil live | 2026-06 ~ 2026-10-07 | `~/sunrun/leaderboard/store/donki-live/` | modelCompletionTime ≤ origin | E0472/E1022/E1198 (2026-06~), srmag72 |  |

## 수집 중 (새 데이터, 아직 실험에 쓰지 않음)

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-030 | STEREO-A beacon HI1 | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/stereo_a/raw/hi_1/` | available_at = DATE-END + 1h (2026-10-15 실측 기준; fetched_at은 과거 공개 시각 아님) | 아직 없음 | 2026-10-14 사용자 지정. 2024 이후 947일치. #55 HI는 실시간 track 0건으로 닫힘 |
| SW-031 | STEREO-A beacon HI2 | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/stereo_a/raw/hi_2/` | available_at = DATE-END + 1h (2026-10-15 실측 기준) | 아직 없음 | 2024 이후 1017일치 |
| SW-032 | ISEE IPS (STEL) | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/ips/raw/stel/` | available_at = 관측 이후 처음 오는 18:00 UT (STEL 파일 매일 약 17:20 UT 재생성, 2026-10-15 실측 기준) | 아직 없음 | #58-1(E0598)에서 공개 시각 부재로 닫힘. 2010년 파일부터 공개 |

## 평가했지만 미채택 (같은 데이터로 새 행을 낼 때는 이 결과를 근거로 적는다)

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-011 | GONG polar-cap 특징 | 2022-10-01 ~ 2026-09-30 | `~/sunrun/leaderboard/store/solar-inc-gongpolar72-20261014-cache/` | max(관측 + 2h, HTTP Last-Modified) | E1198 = E1022 + GONG polar (3366.5) | 6-seed 재현(E1224/E1225) 실패. STACK4(E1269, 실행 중)에 포함 |
| SW-013 | ACE K1 SWEPAM alpha/He | 2022-09-01 ~ 2026-10-03 | `~/sunrun/leaderboard/store/alphahe72-cache/` | K1 244h 지연; MAG 관측 + 2h | alphahe72 (3394.3) | 부분 개선 |
| SW-016 | ACE 파생 캐시 (alfven/aceturb/mambats/L72/tapfnx) | 2022-08 ~ 2026-09 | `~/sunrun/leaderboard/store/<이름>-cache` | 관측 + 2h | alfven 3417.2, aceturb72 3406.1, L72-COND 3383.2, MambaTS 5247, TAPFN 5991 | L72-COND 개선(미채택), aceturb 재현 실패. 둘 다 STACK4(E1269, 실행 중)에 포함 |
| SW-017 | ACE SWEPAM strahl 272 eV | - ~ - | `~/sunrun/leaderboard/store/solar-inc-strahl72-20261014/` | 관측 + 2h | strahl72 3391.3 (재현 3424.4) |  |
| SW-018 | GONG-OCB72 PFSS 위상 | 2022-10 ~ 2026-09 | `~/sunrun/leaderboard/store/solar-inc-gongocb72-20261014-cache (+r2~r7)` | max(관측 + 2h, Last-Modified) | 3405.3 | 판정: 해당 없음 |
| SW-019 | PFSS connectivity | 2022-10 ~ 2026-09 | `~/sunrun/leaderboard/store/pfssconn-cache/` | SW-011과 같음 | 3742.5 |  |
| SW-020 | PFSS-WSA 직접 | - ~ - | `~/sunrun/leaderboard/store/solar-inc-pfss-wsa-20261014-cache/` | SW-011과 같음 | 4024.3 | 실패 |
| SW-021 | CME-HCS72 | - ~ - | `~/sunrun/leaderboard/store/cmehcs72-hcs-cache/` | SW-011과 같음 | 3450.1 |  |
| SW-022 | CH unipolarity / 자기장 집중도 | - ~ - | `~/sunrun/leaderboard/store/solar-inc-chunip72-20261014-cache, solar-inc-chmagconc72-20261014-cache` | SW-011 + SUVI proxy | 3372.8 / 3385.7 |  |
| SW-023 | CME-chip | - ~ - | `~/sunrun/leaderboard/store/cmechip72-cache/` | SW-011 + SUVI proxy | 3444.9 |  |
| SW-024 | NOAA SWPC WSA-Enlil ambient 아카이브 | 2023-04-04 ~ 2026-09-22 | `~/sunrun/leaderboard/store/solar-inc-ambsw72-20261014/, ~/sunrun/leaderboard/store/enlil79-step1/` | init + 12h 또는 HTTP Last-Modified | ambsw72 3421.0 | 부분 개선 |
| SW-025 | DONKI CME·CMEAnalysis·IPS·HSS | 2022-07 ~ 2026-10 | `~/sunrun/leaderboard/store/donki-cme/` | submissionTime ≤ origin, isMostAccurate 안 씀 | CME-PRECOND72 3405.5, cme-fd-huxt72 실행 중 | CME-PRECOND72 개선(미채택), STACK4(E1269, 실행 중)에 포함 |
| SW-026 | ENLIL revision 궤적 | - ~ - | `SW-009/010 파생` | modelCompletionTime | enlilrev72 E1277 |  |
| SW-027 | CCMC CME Scoreboard | 2022-10-01 ~ 2026-10-14 | `~/sunrun/leaderboard/store/cmesb72/` | submissionTime ≤ origin | cmesb72 3527.5 | 결론 없음 |
| SW-028 | SOHO LASCO C2 512px | 2022-09-30 ~ 2026-09-26 | `~/sunrun/leaderboard/store/solar-inc-lascovision-20261014/` | realtime 증거 기록 | lascovision 3413.0, lascomassdrag E1270 |  |
| SW-029 | CACTus LASCO CME 카탈로그 | 2022-10 ~ 2026-10 | `~/sunrun/leaderboard/store/solar-inc-cactuskin-20261014/` | snapshot vintage | 3483.1 |  |
| SW-033 | PUNCH QAM J-map | 2026-07-23 ~ 2026-10-13 | `~/sunrun/leaderboard/store/punch-jmap72-cache/ (+latency-proxy-cache)` | max(FITS DATE, Last-Modified, DATE-END + 1h) | 3406.7 | 부분 개선 |
| SW-034 | SWPC Solar & Geophysical Event Reports (Type II/III/IV) | 2022-09-01 ~ 2026-09-30 | `~/sunrun/leaderboard/store/typeii72_20261014/, solar-inc-typeiiie72-20261014/, solar-inc-typeiv72-20261014/` | 보고서 생성 시각 | Type II 3561.3, III 3423.7, IV 3416.5 |  |
| SW-035 | GOES-18 XRS | 2022 ~ 2026-10 | `~/sunrun/leaderboard/store/goes-xrs72-20261014/` | 측정 + 10분 | 3408.7 |  |
| SW-036 | GOES-18 MPS-HI SEP | 2022-09-24 ~ 2026-10-12 | `~/sunrun/leaderboard/store/goes-sep72-20261014/` | manifest 기준 | 3487.8 |  |
| SW-037 | SDO/HMI SHARP NRT | 2024-12-24 ~ 2026-10-13 | `~/sunrun/leaderboard/store/solar-inc-srmag72-20261014/` | T_REC + 4h ≤ origin | 3397.7 |  |
| SW-038 | NMDB 중성자 모니터 | 2022-09-24 ~ 2026-09-26 | `~/sunrun/leaderboard/store/nmdb-anis72-cache/` | 시간 bin 종료 + 지연 | NMDB-ANIS72 3594.3, E0625 3547 |  |
| SW-039 | SUVI Fe171 | 2022-08-22 ~ 2026-10-12 | `~/sunrun/sun-img/solar_g18_fe171.duckdb, ~/sunrun/leaderboard/store/fe171-cache/` | SUVI proxy 규칙 | directive #26, 신경망 E0449/E0451/E0479/E0484 |  |
| SW-040 | SUVI 파생 (EUV wave, dimming, CH 형태, CH mask) | 2022-08 ~ 2026-10-13 | `~/sunrun/leaderboard/store/euvwave72, suvi-dim72-cache, chmorph72, chmask-cache` | SUVI proxy 규칙 | 3406.0 / 3421.6 / 3493.5 |  |
| SW-041 | 영상 임베딩 (DINOv3, V-JEPA) | 2022-11-22 ~ 2026-05-28 | `~/sunrun/leaderboard/store/dinov3-cache, vjepa-frozen-fusion, vjepa-120h-fusion` | SUVI proxy 규칙 | E0055~58 실패 |  |
| SW-042 | SWPC Forecast Discussion | 2024-03-01 ~ 2026-10-09 | `~/sunrun/leaderboard/store/swpc-discussion76/` | 발행 시각 | directive #76 | 시트 LB 행 없음 |
| SW-058 | GONG multi-feature (gongmulti72) | - ~ - | `~/.local/state/research-dispatch/runs/solar/solar-inc-gongmulti72-20261014-*` | SW-011과 같음 | solar-inc-gongmulti72-20261014 |  |

## 부분 사본·유휴·평가 전용

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-012 | GONG mrzqs FITS 원본 (부분) | 2026-06-01 ~ 2026-09-26 | `~/sunrun/leaderboard/store/scratch/pfssconn_20261014/gong_fits (+cmehcs72 85장)` | 관측 + 2h / Last-Modified | pfssconn, cmehcs, gongocb, chunip, cmechip | 2022-10~2026-05 bulk 3534장은 없어짐. GONG 금지는 2026-10-11 해제 |
| SW-014 | ACE Level 2 (검증본) | 2000-08-01 ~ 2026-09-27 | `~/spaceweather-collector/data/cache-l2` | L2 지연으로 인과 사용 안 함 | - |  |
| SW-043 | 공식 리더보드 스냅샷 | 2026-06-01 ~ 2026-09-30 | `~/.cache/huggingface (expos/evaluate.py 안에서만 읽음)` | 평가 전용 | lb-eval | 선택에 쓰면 안 됨. baseline-replay.parquet도 금지 |
| SW-057 | GONG 자기장 디코딩 캐시 | 2022-10 ~ 2026-09 | `~/sunrun/leaderboard/store/solar-inc-chunip72-20261014-cache/source-cache/gong/` | SW-011과 같음 | chunip72 | 원본 FITS(2022-10~2026-05)가 삭제돼 남은 거의 유일한 GONG 사본(가공본). 파일별 날짜는 미확인 |

## 증거만 있는 폴더 (데이터셋 아님)

probe 응답·샘플만 있다. 학습 데이터로 쓰지 않는다: SW-015 `~/spaceweather-collector/data/ace_release` (시간별 available_at 상한), SW-044 `~/sunrun/leaderboard/store/baron69-evidence/` (STEREO SWAVES beacon (#69 차단)), SW-045 `~/sunrun/leaderboard/store/beacon2science68-assets/` (Beacon2Science (#68)), SW-046 `~/sunrun/leaderboard/store/strudl62a-resume/` (STRUDL 코드 + HI beacon 일부 (#62a)), SW-047 `~/sunrun/leaderboard/store/turner70-evidence/` (BRaVDA/STEREO PLASTIC 샘플 (#70)), SW-048 `~/sunrun/leaderboard/store/l5l1-72-evidence/` (JPL Horizons 위치), SW-049 `~/sunrun/leaderboard/store/ips73-evidence/` (MWA·중국 IPS 목록 (#73/#74)), SW-050 `~/sunrun/leaderboard/store/coronagraph58-3-evidence/` (GOES-19 CCOR-1 샘플 (#58-3)), SW-051 `~/sunrun/leaderboard/store/solarorbiter58-4-evidence/` (Solar Orbiter LL 샘플 (#58-4)), SW-052 `~/sunrun/leaderboard/store/stoarc58_2a-evidence/` (e-CALLISTO burst list 색인 (#58-2a)), SW-053 `~/sunrun/leaderboard/store/surya67-metadata-20261011/` (SDO AIA/HMI 목록 (#67)), SW-054 `~/sunrun/leaderboard/store/punch66-metadata-20261011/` (PUNCH 메타데이터 (#66)), SW-059 `~/sunrun/leaderboard/store/strudl62a-evidence/` (STRUDL 검사 기록 (#62a)), SW-060 `~/sunrun/leaderboard/store/q117-cosa-source/` (COSA 코드 (#117)), SW-061 `~/sunrun/leaderboard/store/surf77-source/` (SURF 저장소 코드 (#77)).

## 미수집·중단

- SW-055 GFZ Kp/ap/F10.7, OMNI2, Kyoto Dst: 지자기·태양 지수. 미수집.
- SW-056 nrt-collect 중단 소스: STEREO COR2/EUVI/PLASTIC/IMPACT, CCOR-2, e-CALLISTO, Solar Orbiter LL, LASCO C2/C3 RT, SOLAR-1, UCSD tomography. 사용자 중단. 2026-10-14 사용자 결정: 연구 데이터와 겹치거나 범위 밖이라 수집 중단·삭제

<!-- LATENCY-TABLE:START -->
## 입수지연 (관측 → 실제 입수까지 걸리는 시간)

SolarWind DB의 `입수지연` 열(O열)과 같다. 실측은 수집 기록의 HTTP Last-Modified·생성·제출 시각에서 잰 중앙값과 p90이고, `/` 뒤는 연구에서 쓰는 인과 규칙이다. 근거 계산은 `~/sunrun/data-v2/latency_report.py`(결과 `reports/latency-20261015.json`). 행에서 인과 지연 상수를 정할 때는 규칙값 이상, 실측 p90을 넘는 쪽을 권장한다.

| ID | 데이터셋 | 입수지연 |
|---|---|---|
| SW-001 | ACE 실시간 hourly (수집기 원본) | 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-002 | ACE hourly stage (sun-db) | 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-003 | ACE frozen snapshot | 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-004 | ACE 결측 보간 속도 (학습 타깃·입력) | 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-005 | Coronal hole (CH) hourly | 원천 SW-008과 같음 (SUVI 실측 약 1분) / 규칙 SUVI proxy(obs_end+1h) |
| SW-006 | SUVI Fe195 경도 strip | 실측 중앙 1분 (p90 1분, n=1,429, last 60 days, S3 LastModified−obs_end) / 규칙 obs_end+1h 및 S3 LastModified≤origin |
| SW-007 | SUVI hourly 메타데이터 | 실측 중앙 1분 (p90 1분, n=1,429, last 60 days, S3 LastModified−obs_end) / 규칙 obs_end+1h 및 S3 LastModified≤origin |
| SW-008 | GOES-18 SUVI L1b Fe195 원본 | 실측 중앙 1분 (p90 1분, n=1,429, last 60 days, S3 LastModified−obs_end) / 규칙 obs_end+1h 및 S3 LastModified≤origin |
| SW-009 | DONKI WSA-Enlil 아카이브 | 실측 중앙 5.8h (p90 15.6h, n=399, 2026-06..09, modelCompletionTime−입력 CME 시작) / 규칙 modelCompletionTime≤origin |
| SW-010 | DONKI WSA-Enlil live | 실측 중앙 5.8h (p90 15.6h, n=399, 2026-06..09, modelCompletionTime−입력 CME 시작) / 규칙 modelCompletionTime≤origin |
| SW-011 | GONG polar-cap 특징 | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-012 | GONG mrzqs FITS 원본 (부분) | 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-013 | ACE K1 SWEPAM alpha/He | 실측 약 242h(아카이브 끝 vs 2026-10-14 조회) / 규칙 244h |
| SW-014 | ACE Level 2 (검증본) | 실측 수개월(2026-10-14 기준 MAG 끝 08-04 약 70일, SWEPAM 2024-07) — 인과 사용 안 함 |
| SW-015 | ACE daily list 공개 시각 | 해당 없음 (ACE 공개 시각 증거 자료) |
| SW-016 | ACE 파생 캐시 (alfven/aceturb/mambats/L72/tapfnx) | 원천 ACE와 같음: 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-017 | ACE SWEPAM strahl 272 eV | 원천 ACE와 같음: 실측: daily list가 일 끝+11분에 생성(n=499, 2025-06~2026-10), 실시간 2-hour list는 2~3분 / 규칙 관측+2h |
| SW-018 | GONG-OCB72 PFSS 위상 | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-019 | PFSS connectivity | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-020 | PFSS-WSA 직접 | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-021 | CME-HCS72 | 원천 SW-012·SW-025와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-022 | CH unipolarity / 자기장 집중도 | 원천 SW-012·SW-008과 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-023 | CME-chip | 원천 SW-012·SW-008·SW-025와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-024 | NOAA SWPC WSA-Enlil ambient 아카이브 | 실측 중앙 24.5h (p90 3.6일, n=15, runs fetched 2026-10-14, archive Last-Modified−init 00:00) / 규칙 init+12h 또는 Last-Modified — init+12h는 실측보다 짧으므로 Last-Modified 기준이 안전 |
| SW-025 | DONKI CME·CMEAnalysis·IPS·HSS | 실측 CMEAnalysis 중앙 3.3h (p90 11.0h, n=244, time21_5 기준), CME 기록 중앙 7.0h (p90 4.4일, n=564, startTime 기준), 2026-06~09 / 규칙 submissionTime≤origin |
| SW-026 | ENLIL revision 궤적 | 원천 SW-009와 같음: 실측 중앙 5.8h (p90 15.6h, n=399, 2026-06..09, modelCompletionTime−입력 CME 시작) / 규칙 modelCompletionTime≤origin |
| SW-027 | CCMC CME Scoreboard | 실측 중앙 17.4h (p90 2.4일, n=412, CMEs since 2026-06, submissionTime−CME 시각) / 규칙 submissionTime≤origin |
| SW-028 | SOHO LASCO C2 512px | 실측 중앙 48분 (p90 4.4h, n=1,786, last 60 days, Last-Modified−프레임 시각) / 규칙 Last-Modified(약 관측+1h) |
| SW-029 | CACTus LASCO CME 카탈로그 | live 페이지 6시간 주기 / 월간 카탈로그는 다음 달 초 발행 |
| SW-030 | STEREO-A beacon HI1 | 실측 중앙 20분 (p90 25분, n=23, live fetches since 2026-10-14, Last-Modified−DATE-OBS) / 규칙 DATE-END+1h (2026-10-15 실측 p90의 약 2배) |
| SW-031 | STEREO-A beacon HI2 | 실측 중앙 29분 (p90 31분, n=21, live fetches since 2026-10-14, Last-Modified−DATE-OBS) / 규칙 DATE-END+1h (2026-10-15 실측 p90의 약 2배) |
| SW-032 | ISEE IPS (STEL) | 실측: 파일이 매일 약 17:20 UT에 재생성(10-13 17:20, 10-14 17:16), 관측은 20:00~09:00 UT라 관측→입수 약 8~22h (마지막 관측 기준 중앙 9.2h) / 규칙 관측 시각 이후 처음 오는 18:00 UT |
| SW-033 | PUNCH QAM J-map | 실측 중앙 36.0h (p90 2.9일, n=170, 2026-09..10, Last-Modified−관측); 2026-06 초기분은 07-28 일괄 공개(수 주) / 규칙 max(FITS DATE, Last-Modified, DATE-END+1h) |
| SW-034 | SWPC Solar & Geophysical Event Reports (Type II/III/IV) | 실측 NGDC 아카이브 일 끝+2.2일 (n=123, 2026-06~09), 실시간 텍스트는 수 분 / 규칙 보고서 생성 시각 |
| SW-035 | GOES-18 XRS | 규칙 측정+10분 (운영 1분 자료). NCEI 연 파일은 일 단위 갱신(실측 약 1일) |
| SW-036 | GOES-18 MPS-HI SEP | 실측 첫 공개 일 끝+4.5h, 같은 파일명으로 +2.2일·+6.2일 재발행(최신본 기준이면 약 6일) / 규칙 manifest의 버전별 Last-Modified |
| SW-037 | SDO/HMI SHARP NRT | 규칙 T_REC+4h (JSOC NRT 키워드, 레코드별 공개 시각 없음) |
| SW-038 | NMDB 중성자 모니터 | 규칙 시간 bin 끝+64분(spec latency 4분 + bin) |
| SW-039 | SUVI Fe171 | 실측 중앙 1분 (p90 1분, n=1,429, last 60 days, S3 LastModified−obs_end) / 규칙 SUVI proxy(obs_end+1h) |
| SW-040 | SUVI 파생 (EUV wave, dimming, CH 형태, CH mask) | 원천 SW-008과 같음 (SUVI 실측 약 1분) / 규칙 SUVI proxy |
| SW-041 | 영상 임베딩 (DINOv3, V-JEPA) | 원천 SW-008과 같음 (SUVI 실측 약 1분) / 규칙 SUVI proxy |
| SW-042 | SWPC Forecast Discussion | 실측 발행 즉시 (NGDC Last-Modified−발행 1분, n=109) / 규칙 발행 시각≤origin |
| SW-055 | GFZ Kp/ap/F10.7, OMNI2, Kyoto Dst | - |
| SW-057 | GONG 자기장 디코딩 캐시 | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
| SW-058 | GONG multi-feature | 원천 SW-012와 같음: 실측 중앙 38분 (p90 4.9h, n=58, last 60 days, Last-Modified−지도 시각) / 규칙 max(관측+2h, Last-Modified) |
<!-- LATENCY-TABLE:END -->

<!-- DATA-V2-TABLE:START -->
## v2 최신화 상태

위 표는 v1 재현 이력이다. 아래 상태는 SolarWind DB의 S:W 열과 함께 갱신한다. `ready` 전환 조건은 서버의 `data-v2/reports/defaults.json`이며, 수집·해시 검사와 실험 사용 검증은 별개다. 실제 관측 끝이 미검증이거나 일부 생성/전송이 남은 항목을 최신화 완료로 취급하지 않는다.

| ID | 데이터셋 | v2 경로 | 관측 최신 | 검증 상태 |
|---|---|---|---|---|
| SW-001 | ACE 실시간 hourly (수집기 원본) |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-002 | ACE hourly stage (sun-db) |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-003 | ACE frozen snapshot | /home/t-lab01/sunrun/leaderboard/store/v2/ace-snapshot |  | 미완료 / manifest 없음 |
| SW-004 | ACE 결측 보간 속도 (학습 타깃·입력) | /home/t-lab01/sunrun/leaderboard/store/v2/ch |  | 미완료 / manifest 없음 |
| SW-005 | Coronal hole (CH) hourly | /home/t-lab01/sunrun/leaderboard/store/v2/ch |  | 미완료 / manifest 없음 |
| SW-006 | SUVI Fe195 경도 strip | /home/t-lab01/sunrun/leaderboard/store/v2/suvi-fusion |  | 미완료 / manifest 없음 |
| SW-007 | SUVI hourly 메타데이터 | /home/t-lab01/sunrun/leaderboard/store/v2/suvi-meta |  | 미완료 / manifest 없음 |
| SW-008 | GOES-18 SUVI L1b Fe195 원본 |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-009 | DONKI WSA-Enlil 아카이브 | /home/t-lab01/sunrun/leaderboard/store/v2/donki | 관측 끝 미검증 | donki: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-010 | DONKI WSA-Enlil live | /home/t-lab01/sunrun/leaderboard/store/v2/donki | 관측 끝 미검증 | donki: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-011 | GONG polar-cap 특징 | /home/t-lab01/sunrun/leaderboard/store/v2/gong-derived | 관측 끝 미검증 | gong-derived: partial |
| SW-012 | GONG mrzqs FITS 원본 (부분) | /home/t-lab01/sunrun/leaderboard/store/v2/gong-mrzqs |  | 미완료 / manifest 없음 |
| SW-013 | ACE K1 SWEPAM alpha/He | /home/t-lab01/sunrun/leaderboard/store/v2/ace-k1 | 관측 끝 미검증 | ace-k1: complete |
| SW-014 | ACE Level 2 (검증본) | /home/t-lab01/sunrun/leaderboard/store/v2/ace-l2 | 관측 끝 미검증 | ace-l2: complete |
| SW-015 | ACE daily list 공개 시각 | /home/t-lab01/sunrun/leaderboard/store/v2/ace-release | 관측 끝 미검증 | ace-release: complete_with_upstream_gaps |
| SW-016 | ACE 파생 캐시 (alfven/aceturb/mambats/L72/tapfnx) | /home/t-lab01/sunrun/leaderboard/store/v2/aceturb<br>/home/t-lab01/sunrun/leaderboard/store/v2/alfven<br>/home/t-lab01/sunrun/leaderboard/store/v2/mambats-inputs<br>/home/t-lab01/sunrun/leaderboard/store/v2/mambats-source | 관측 끝 미검증<br>관측 끝 미검증<br>관측 끝 미검증<br>관측 끝 미검증 | aceturb: completed<br>alfven: completed<br>mambats-inputs: completed<br>mambats-source: running |
| SW-017 | ACE SWEPAM strahl 272 eV |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-018 | GONG-OCB72 PFSS 위상 | /home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r2<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r3<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r4<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r5<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r6<br>/home/t-lab01/sunrun/leaderboard/store/v2/gong-ocb-r7 | 관측 끝 미검증 | gong-ocb: partial<br>미완료 / manifest 없음<br>미완료 / manifest 없음<br>미완료 / manifest 없음<br>미완료 / manifest 없음<br>미완료 / manifest 없음<br>미완료 / manifest 없음 |
| SW-019 | PFSS connectivity | /home/t-lab01/sunrun/leaderboard/store/v2/pfssconn | 관측 끝 미검증 | pfssconn: partial |
| SW-020 | PFSS-WSA 직접 | /home/t-lab01/sunrun/leaderboard/store/v2/pfss-wsa |  | 미완료 / manifest 없음 |
| SW-021 | CME-HCS72 | /home/t-lab01/sunrun/leaderboard/store/v2/cme-hcs | 관측 끝 미검증 | cme-hcs: partial |
| SW-022 | CH unipolarity / 자기장 집중도 | /home/t-lab01/sunrun/leaderboard/store/v2/chunip<br>/home/t-lab01/sunrun/leaderboard/store/v2/magconc |  | 미완료 / manifest 없음<br>미완료 / manifest 없음 |
| SW-023 | CME-chip | /home/t-lab01/sunrun/leaderboard/store/v2/cmechip | 관측 끝 미검증 | cmechip: running |
| SW-024 | NOAA SWPC WSA-Enlil ambient 아카이브 | /home/t-lab01/sunrun/leaderboard/store/v2/enlil-ambient |  | 미완료 / manifest 없음 |
| SW-025 | DONKI CME·CMEAnalysis·IPS·HSS | /home/t-lab01/sunrun/leaderboard/store/v2/donki | 관측 끝 미검증 | donki: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-026 | ENLIL revision 궤적 |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-027 | CCMC CME Scoreboard | /home/t-lab01/sunrun/leaderboard/store/v2/ccmc-cme-scoreboard | 관측 끝 미검증 | ccmc-cme-scoreboard: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-028 | SOHO LASCO C2 512px | /home/t-lab01/sunrun/leaderboard/store/v2/lasco-c2-512 | 관측 끝 미검증 | lasco-c2-512: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-029 | CACTus LASCO CME 카탈로그 | /home/t-lab01/sunrun/leaderboard/store/v2/cactus | 관측 끝 미검증 | cactus: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-030 | STEREO-A beacon HI1 |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-031 | STEREO-A beacon HI2 |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-032 | ISEE IPS (STEL) |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-033 | PUNCH QAM J-map | /home/t-lab01/sunrun/leaderboard/store/v2/punch-qam |  | 미완료 / manifest 없음 |
| SW-034 | SWPC Solar & Geophysical Event Reports (Type II/III/IV) | /home/t-lab01/sunrun/leaderboard/store/v2/swpc-events | 관측 끝 미검증 | swpc-events: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-035 | GOES-18 XRS | /home/t-lab01/sunrun/leaderboard/store/v2/goes18-xrs | 2026-10-12T23:59:00.000000000 | goes18-xrs: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-036 | GOES-18 MPS-HI SEP | /home/t-lab01/sunrun/leaderboard/store/v2/goes18-sep | 20261013 | goes18-sep: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-037 | SDO/HMI SHARP NRT | /home/t-lab01/sunrun/leaderboard/store/v2/hmi-sharp-nrt | 관측 끝 미검증 | hmi-sharp-nrt: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-038 | NMDB 중성자 모니터 | /home/t-lab01/sunrun/leaderboard/store/v2/nmdb | 2026-10-14 11:00:00+00:00 | nmdb: 생성됨; 관측 끝/실험 사용 별도 검증 |
| SW-039 | SUVI Fe171 | /home/t-lab01/sunrun/leaderboard/store/v2/fe171 | 관측 끝 미검증 | fe171: 생성됨; 관측 끝/실험 사용 별도 검증 |
| SW-040 | SUVI 파생 (EUV wave, dimming, CH 형태, CH mask) | /home/t-lab01/sunrun/leaderboard/store/v2/euvwave<br>/home/t-lab01/sunrun/leaderboard/store/v2/suvi-dim<br>/home/t-lab01/sunrun/leaderboard/store/v2/chmorph<br>/home/t-lab01/sunrun/leaderboard/store/v2/chmask | 관측 끝 미검증 | euvwave: 생성됨; 관측 끝/실험 사용 별도 검증<br>미완료 / manifest 없음<br>미완료 / manifest 없음<br>미완료 / manifest 없음 |
| SW-041 | 영상 임베딩 (DINOv3, V-JEPA) | /home/t-lab01/sunrun/leaderboard/store/v2/dinov3<br>/home/t-lab01/sunrun/leaderboard/store/v2/vjepa-frozen<br>/home/t-lab01/sunrun/leaderboard/store/v2/vjepa-120h | 관측 끝 미검증<br>관측 끝 미검증 | dinov3: blocked_credentials<br>vjepa-frozen: running<br>미완료 / manifest 없음 |
| SW-055 | GFZ Kp/ap/F10.7, OMNI2, Kyoto Dst |  |  | 기존 원천 / 증거 / 갱신 대상 아님 (이력 보존) |
| SW-057 | GONG 자기장 디코딩 캐시 | /home/t-lab01/sunrun/leaderboard/store/v2/gong-decoded | 관측 끝 미검증 | gong-decoded: partial |
| SW-042 | SWPC Forecast Discussion | /home/t-lab01/sunrun/leaderboard/store/v2/swpc-discussion | 관측 끝 미검증 | swpc-discussion: 생성됨; 관측 끝/실험 사용 별도 검증; SHA256 index verified |
| SW-058 | GONG multi-feature | /home/t-lab01/sunrun/leaderboard/store/v2/gong-derived | 관측 끝 미검증 | gong-derived: partial |

<!-- DATA-V2-TABLE:END -->
