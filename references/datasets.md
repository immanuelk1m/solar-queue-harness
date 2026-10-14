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
| SW-030 | STEREO-A beacon HI1 | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/stereo_a/raw/hi_1/` | available_at 증빙 없음 → 보수적 지연 상수 필요; fetched_at은 과거 공개 시각 아님 | 아직 없음 | 2026-10-14 사용자 지정. 2024 이후 947일치. #55 HI는 실시간 track 0건으로 닫힘 |
| SW-031 | STEREO-A beacon HI2 | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/stereo_a/raw/hi_2/` | SW-030과 같음 | 아직 없음 | 2024 이후 1017일치 |
| SW-032 | ISEE IPS (STEL) | 2024-01-01 ~ 현재 | `~/sunrun/nrt-collect/data/ips/raw/stel/` | 관측별 공개 시각 없음 → 보수적 지연 상수 필요 | 아직 없음 | #58-1(E0598)에서 공개 시각 부재로 닫힘. 2010년 파일부터 공개 |

## 평가했지만 미채택 (같은 데이터로 새 행을 낼 때는 이 결과를 근거로 적는다)

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-011 | GONG polar-cap 특징 | 2022-10-01 ~ 2026-09-30 | `~/sunrun/leaderboard/store/solar-inc-gongpolar72-20261014-cache/` | max(관측 + 2h, HTTP Last-Modified) | E1198 = E1022 + GONG polar (3366.5) | 6-seed 재현(E1224/E1225) 실패 |
| SW-013 | ACE K1 SWEPAM alpha/He | 2022-09-01 ~ 2026-10-03 | `~/sunrun/leaderboard/store/alphahe72-cache/` | K1 244h 지연; MAG 관측 + 2h | alphahe72 (3394.3) | 부분 개선 |
| SW-016 | ACE 파생 캐시 (alfven/aceturb/mambats/L72/tapfnx) | 2022-08 ~ 2026-09 | `~/sunrun/leaderboard/store/<이름>-cache` | 관측 + 2h | alfven 3417.2, aceturb72 3406.1, L72-COND 3383.2, MambaTS 5247, TAPFN 5991 | L72-COND만 개선(미채택), aceturb 재현 실패 |
| SW-017 | ACE SWEPAM strahl 272 eV | - ~ - | `~/sunrun/leaderboard/store/solar-inc-strahl72-20261014/` | 관측 + 2h | strahl72 3391.3 (재현 3424.4) |  |
| SW-018 | GONG-OCB72 PFSS 위상 | 2022-10 ~ 2026-09 | `~/sunrun/leaderboard/store/solar-inc-gongocb72-20261014-cache (+r2~r7)` | max(관측 + 2h, Last-Modified) | 3405.3 | 판정: 해당 없음 |
| SW-019 | PFSS connectivity | 2022-10 ~ 2026-09 | `~/sunrun/leaderboard/store/pfssconn-cache/` | SW-011과 같음 | 3742.5 |  |
| SW-020 | PFSS-WSA 직접 | - ~ - | `~/sunrun/leaderboard/store/solar-inc-pfss-wsa-20261014-cache/` | SW-011과 같음 | 4024.3 | 실패 |
| SW-021 | CME-HCS72 | - ~ - | `~/sunrun/leaderboard/store/cmehcs72-hcs-cache/` | SW-011과 같음 | 3450.1 |  |
| SW-022 | CH unipolarity / 자기장 집중도 | - ~ - | `~/sunrun/leaderboard/store/solar-inc-chunip72-20261014-cache, solar-inc-chmagconc72-20261014-cache` | SW-011 + SUVI proxy | 3372.8 / 3385.7 |  |
| SW-023 | CME-chip | - ~ - | `~/sunrun/leaderboard/store/cmechip72-cache/` | SW-011 + SUVI proxy | 3444.9 |  |
| SW-024 | NOAA SWPC WSA-Enlil ambient 아카이브 | 2023-04-04 ~ 2026-09-22 | `~/sunrun/leaderboard/store/solar-inc-ambsw72-20261014/, ~/sunrun/leaderboard/store/enlil79-step1/` | init + 12h 또는 HTTP Last-Modified | ambsw72 3421.0 | 부분 개선 |
| SW-025 | DONKI CME·CMEAnalysis·IPS·HSS | 2022-07 ~ 2026-10 | `~/sunrun/leaderboard/store/donki-cme/` | submissionTime ≤ origin, isMostAccurate 안 씀 | CME-PRECOND72 3405.5, cme-fd-huxt72 실행 중 | CME-PRECOND72 개선(미채택) |
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

## 부분 사본·유휴·평가 전용

| ID | 데이터셋 | 기간 | 로컬 경로 | 인과 규칙 | 사용처·결과 | 비고 |
|---|---|---|---|---|---|---|
| SW-012 | GONG mrzqs FITS 원본 (부분) | 2026-06-01 ~ 2026-09-26 | `~/sunrun/leaderboard/store/scratch/pfssconn_20261014/gong_fits (+cmehcs72 85장)` | 관측 + 2h / Last-Modified | pfssconn, cmehcs, gongocb, chunip, cmechip | 2022-10~2026-05 bulk 3534장은 없어짐. GONG 금지는 2026-10-11 해제 |
| SW-014 | ACE Level 2 (검증본) | 2000-08-01 ~ 2026-09-27 | `~/spaceweather-collector/data/cache-l2` | L2 지연으로 인과 사용 안 함 | - |  |
| SW-043 | 공식 리더보드 스냅샷 | 2026-06-01 ~ 2026-09-30 | `~/.cache/huggingface (expos/evaluate.py 안에서만 읽음)` | 평가 전용 | lb-eval | 선택에 쓰면 안 됨. baseline-replay.parquet도 금지 |

## 증거만 있는 폴더 (데이터셋 아님)

probe 응답·샘플만 있다. 학습 데이터로 쓰지 않는다: SW-015 `~/spaceweather-collector/data/ace_release` (시간별 available_at 상한), SW-044 `~/sunrun/leaderboard/store/baron69-evidence/` (STEREO SWAVES beacon (#69 차단)), SW-045 `~/sunrun/leaderboard/store/beacon2science68-assets/` (Beacon2Science (#68)), SW-046 `~/sunrun/leaderboard/store/strudl62a-resume/` (STRUDL 코드 + HI beacon 일부 (#62a)), SW-047 `~/sunrun/leaderboard/store/turner70-evidence/` (BRaVDA/STEREO PLASTIC 샘플 (#70)), SW-048 `~/sunrun/leaderboard/store/l5l1-72-evidence/` (JPL Horizons 위치), SW-049 `~/sunrun/leaderboard/store/ips73-evidence/` (MWA·중국 IPS 목록 (#73/#74)), SW-050 `~/sunrun/leaderboard/store/coronagraph58-3-evidence/` (GOES-19 CCOR-1 샘플 (#58-3)), SW-051 `~/sunrun/leaderboard/store/solarorbiter58-4-evidence/` (Solar Orbiter LL 샘플 (#58-4)), SW-052 `~/sunrun/leaderboard/store/stoarc58_2a-evidence/` (e-CALLISTO burst list 색인 (#58-2a)), SW-053 `~/sunrun/leaderboard/store/surya67-metadata-20261011/` (SDO AIA/HMI 목록 (#67)), SW-054 `~/sunrun/leaderboard/store/punch66-metadata-20261011/` (PUNCH 메타데이터 (#66)).

## 미수집·중단

- SW-055 GFZ Kp/ap/F10.7, OMNI2, Kyoto Dst: 지자기·태양 지수. 미수집.
- SW-056 nrt-collect 중단 소스: STEREO COR2/EUVI/PLASTIC/IMPACT, CCOR-2, e-CALLISTO, Solar Orbiter LL, LASCO C2/C3 RT, SOLAR-1, UCSD tomography. 사용자 중단. 2026-10-14 사용자 결정: 연구 데이터와 겹치거나 범위 밖이라 수집 중단·삭제
