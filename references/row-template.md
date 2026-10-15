# `태양풍 큐` 행 양식

워커는 `사양·지시` 칸을 그대로 작업 지시로 읽는다. 아래 대괄호 필드명을 바꾸지 말고 순서대로 모두 채운다. 모르는 값은 "미확인(워커가 시트·코드에서 확인)"으로 적는다.

## 열
- `ID`: 예측 실험은 `solar-<inc|comb|pivot|rep>-<짧은영문이름>-YYYYMMDD`. REPLICATE는 `solar-rep-<대상 E id 소문자>-YYYYMMDD`. ARCH·ARCH-STEP은 `solar-arch-<계보>-s<N>-YYYYMMDD`. 해결 작업은 `solar-<enabledata|enablephys|infotest|discovery>-<짧은영문이름>-YYYYMMDD`. 시트 두 탭에 같은 ID가 있으면 안 된다.
- `항목`: `[INCREMENT|COMBINE|PIVOT|REPLICATE|ARCH|ARCH-STEP|ENABLE-DATA|ENABLE-PHYSICS|INFO-TEST|DATA-DISCOVERY] <방법 이름> — <한 줄 요약>`
- `사양·지시`: 아래 본문(한국어, 3,000–8,000자 권장).
- `갱신 시각`: 추가한 시각(KST).

## `사양·지시` 본문
```
[모드] INCREMENT | COMBINE | PIVOT   (REPLICATE는 아래 별도 양식)

[기준 레시피]
- 작업 챔피언 <E id>(현재 계약 준수 lb-eval seed-mean <값>, seed <값/값/값>, 고속 셀 llh/lhh/hlh/hhh <값들>). SKILL 1단계에서 시트 최신값으로 확정한 값을 쓴다. 기본값을 하드코딩하지 않는다.
- 코드: <로컬 경로 또는 GitHub scripts/seongeun/... 경로>. 시트의 해당 E id 결과·experiment에서 확인한다.
- 대조군 = 이 기준 레시피를 같은 코드·같은 seed·ACE obs+2h 규칙으로 전체 구간에서 다시 돌린 결과다. 이미 있으면 그 E id를 쓴다.

[단일 변경]
- 기준 레시피에서 바뀌는 것 하나만 코드 수준으로 쓴다: 어떤 입력/특징/대상/학습 단위를 어떻게 바꾸는지, 새 함수의 입출력, 추가되는 config 키.
- 바뀌지 않는 것을 명시한다(나머지 특징, 하이퍼파라미터, 후처리).
- COMBINE: 결합할 변경들의 E id와 각 단독 결과를 적고, 결합 방식을 쓴다.

[가설·근거]
- 해결하려는 오차 성분(리드대역/셀/이벤트 유형)과 그 크기: 내부 수치로 쓴다.
- 검증할 미해결 가설을 한 문장으로 쓴다.
- 경쟁 설명을 최소 1개 적는다.
- 근거 종류: LITERATURE(제목·저자·연도·링크, 우리 문제에 옮기는 방식) 또는 MATH+PHYSICS.
- 물리 메커니즘과 수학적 추정량을 적는다.

[연구 가치와 정보이득]
- **새로 알게 되는 것:** 이 실험 전에는 구분할 수 없던 가설/상태가 무엇인지.
- **성공 판정이 의미하는 것:** 어떤 가설의 신뢰도를 높이고 다음 큐에서 무엇을 우선할지.
- **실패 판정이 의미하는 것:** 어떤 메커니즘/정보원/표현의 우선순위를 낮추거나 닫을지.
- **기존 연구와의 직교성:** 기존 champion/HPO/feature/head가 보지 못하던 어떤 정보·표현·물리상태를 보는지.
- **병목 연결:** 현재 오차 질량이나 오라클 headroom 중 어느 부분을 직접 조사하는지.
- 예상 MSE 개선률은 신뢰성 있는 근거가 있을 때만 참고로 적을 수 있다. 수치가 없거나 작다는 이유만으로 후보를 탈락시키지 않는다.

[A-GATE]
- A1 25–72h 병목 직접성: PASS/FAIL — <근거>
- A2 새로운 인과 정보: PASS/FAIL — <새 관측 / propagation state / arrival posterior / pre-arrival precursor>
- A3 기존 챔피언 조건부 추가정보: PASS/FAIL — <ACE+CH/SUVI+DONKI/ENLIL+recurrence를 알아도 남는 정보>
- A4 propagation bridge: PASS/FAIL — <관측/상태 -> heliospheric propagation -> arrival phase -> L1 25–72h speed>
- A5 시간 선행성: PASS/FAIL — <lead-time 또는 물리적 선행 근거>
- 최종 등급: A

다섯 항목 모두 PASS가 아니면 이 예측 행 자체를 만들지 않는다. B/C는 기각 후보 표에만 기록한다.

[휴리스틱 검사]
- `references/heuristics.md` 자가 점검 3문항의 답.
- 이 변경에 들어가는 모든 값·요약·경계(lag, 창, 임계값, 분할, 스칼라 요약, 리드 경계, 멤버·가중, K·lookback, grid)를 나열하고 각각 "학습형(무엇이 어떻게 학습하는지)" 또는 "표준 정의(출처 인용)"로 표시한다. 사람이 정한 값이 하나라도 남으면 행을 만들지 않는다.
- 챔피언의 휴리스틱을 학습형으로 바꾸는 행이면, 바꾸는 휴리스틱(heuristics.md 범주 a–f)을 적는다.

[인과 가용성]
- 새로 쓰는 모든 입력의 관측·발행 시각과 지연 규칙(ACE +2h 등), origin 시점에 얻을 수 있었다는 근거, 공식 창(2026-06~09)까지 데이터가 있다는 확인.
- 학습 target < 2026-05-29, 라벨 성숙 u+72h, poison check.

[필요 데이터·자원]
- 기존 store / 새 다운로드(출처 URL, 예상 용량), CPU 또는 GPU, 예상 시간.

[실험]
- 리더보드 전체 구간 full run을 seed 0/1/2로 돌려 predictions_official_s{seed}.parquet을 쓴 뒤 expos lb-eval을 실행한다. smoke와 내부 창 판정은 하지 않는다.
- 같은 기준 레시피 대조군 대비 seed별·평균 MSE, 고속 셀 4개, 리드대역, F1을 보고한다.
- 구현 중 고장은 고쳐서 끝까지 완수한다.

[채택 시]
- 작업 챔피언보다 평균이 1.5% 이상 낮고 모든 seed가 낮으면 이 run이 새 작업 챔피언이 된다(1.5% 미만은 동률). 이후 큐 행은 이 run을 [기준 레시피]로 삼는다.
- run 시작 시 시트에서 확정한 **현재 SOTA**와 동일 평가 계약에서 그 값을 재현 가능하게 넘으면 SOTA 갱신 후보로 기록한다.
- 특정 E id를 SOTA로 하드코딩하지 않는다.

[부분 개선 시]
- 기준 레시피 대비 평균 −0.8% 이상이고 3개 seed 모두 낮으면 "스택 후보"로 기록한다. 다음 COMBINE 행의 재료가 된다.
- 성능 개선 폭이 작더라도, 사전 정의한 중간 정보 검정에서 중요한 가설을 명확히 판별했다면 `정보 획득` 결과로 별도 기록한다. 성능 개선과 연구 정보 획득을 혼동하지 않는다.

[실패 시 닫히는 축]
- 개선이 없으면 어떤 메커니즘을 닫고, 무엇은 닫지 않는지 쓴다.

[중복 검사]
- 시트 `태양풍 연구`/`태양풍 큐`에서 가장 가까운 기존 행 2–3개의 ID와, 메커니즘이 어떻게 다른지.
```

## 결과 기록 줄 (모든 모드 공통 — 워커가 결과 칸 첫 줄에 쓴다)
`LB| E=<id> base=<base id> seeds=<s0,s1,s2> mse=<a/b/c> mean=<m> d_base=<값,%> cells=<llh/lhh/hlh/hhh> type=<특징 블록 추가|후처리·잔차|하이퍼파라미터|예측 대상 변경|구조·학습 단위|새 정보원|replicate|arch>`
ARCH·ARCH-STEP 행은 같은 줄 끝에 `lineage=<계보> stage=<N> d_prev=<직전 단계 대비 값,%>`를 붙이고, `base`에는 비교 대상인 확정 챔피언 E id를 쓴다.
사양 끝에 "결과 칸 첫 줄에 위 `LB|` 줄을 쓴다"를 적는다.

## 해결 작업 양식 — A급 예측 실험이 0건일 때

해결 작업은 leaderboard 성능을 주장하는 예측 실험이 아니다. 이름이 정해진 A급 실험의 실행 blocker를 제거하거나 A-GATE 판정에 필요한 직접 정보를 얻는다.

```
[모드] ENABLE-DATA | ENABLE-PHYSICS | INFO-TEST | DATA-DISCOVERY

[목표 A급 실험]
- <완료 후 실행할 A급 후보 ID/이름>. DATA-DISCOVERY만 아직 이름이 없을 수 있으며, 그 경우 겨냥하는 phase/propagation 병목을 정확히 적는다.

[현재 blocker]
- 왜 지금 A급 예측 실험을 실행할 수 없는지 한 가지 blocker만 적는다.
- 예: historical publication timestamp 부재, official-window coverage 미확정, 3D field loader 부재, 조건부 정보 존재 여부 미확인.

[해결 작업]
- 코드/데이터 수준에서 이번 행이 실제로 수행할 작업을 쓴다.
- ENABLE-DATA: byte 수집, timestamp/available_at 증거, manifest, coverage audit 등.
- ENABLE-PHYSICS: propagation model/field/operator의 최소 실행·검증 기반 구축.
- INFO-TEST: target<cutoff pre-official 데이터에서 champion residual에 대한 조건부 추가정보를 직접 검정.
- DATA-DISCOVERY: 새 upstream source 후보를 찾고 provider·기간·latency·historical replay 가능성을 검증해 최소 1개를 A-GATE 후보 또는 명시적 기각으로 끝낸다.

[완료 조건]
- PASS/FAIL을 기계적으로 판정할 산출물과 임계가 아닌 사실 조건을 적는다.
- 예: manifest에 observation_end/publication_time/available_at/SHA256이 있고 official origins의 causal coverage가 계산됨.
- 단순히 "조사 완료", "논문 확인"은 완료 조건이 아니다.

[성공 시 다음 행동]
- 어떤 A급 예측 행을 바로 추가/실행할지 적는다.

[실패 시 다음 행동]
- 어떤 축을 보류하고 다음 A급 후보/다음 blocker로 이동할지 적는다.
- 실패해도 B/C 예측 실험으로 fallback하지 않는다.

[인과 가용성]
- official truth를 fit/selection에 사용하지 않는다.
- INFO-TEST는 target<2026-05-29와 label maturity를 지킨다.
- DATA/PHYSICS 작업도 historical replay 가능성을 증명해야 한다.

[필요 데이터·자원]
- source, 저장 위치, 예상 용량, CPU/GPU, 외부 다운로드 여부.

[중복 검사]
- 같은 blocker를 이미 처리 중인 큐/연구 행이 없는지 확인한다.

[결과 기록]
- 첫 줄 `ENABLE| mode=<mode> target=<A급 후보 또는 병목> blocker=<짧은키> verdict=<ready|blocked|informative|not_informative> next=<다음 ID/행동>`.
```

해결 작업은 A-GATE의 대체 예측 실험이 아니다. 완료된 해결 작업 자체를 SOTA/챔피언/스택 후보로 세지 않는다.

## REPLICATE 양식
```
[모드] REPLICATE

[대상과 기준]
- 대상 <E id>(원 seed 0/1/2 MSE, 평균, base 대비 %), 기준 <base E id>(원 seed MSE).
- 재현 이유: 1.5단계 진단의 어떤 신호(H1/H2)와 수치 때문인지.

[단일 변경]
- 없음. 대상 레시피와 기준 레시피를 코드·특징·하이퍼파라미터 그대로 두고 seed 3, 4, 5로 두 run을 새로 돌린다. 원본 코드는 수정하지 않고 이 행 전용 디렉터리에 복사해 seed 목록만 바꾼다.
- 레시피가 seed를 fold/block 인덱스로 쓰는지 먼저 확인한다. 그렇다면 무작위성만 바꾸는 동등한 seed 정의를 두 run에 똑같이 적용하고 기록한다.

[실험]
- 두 run 모두 리더보드 전체 구간 predictions_official_s3/s4/s5.parquet → 각각 expos lb-eval.
- 6-seed 짝 판정: d_k = MSE(대상,k) − MSE(기준,k), k=0..5. 확정 = mean(d)<0 이고 t ≤ −2, 또는 6개 모두 음수. 그 밖은 기각. 새 seed 평균 d ≥ 0이면 "우연"으로 기록.
- 고속 셀 4개·리드대역별 짝 차이도 보고.

[결과 기록]
- 첫 줄 `LB| ... type=replicate`, 둘째 줄 `REP| target=<대상> base=<기준> d=<d0..d5> mean_d=<값> t=<값> verdict=<확정|기각>`.

[채택 시]
- 확정이면 대상이 확정 챔피언(또는 확정 스택 후보)이 된다. 기각이면 "노이즈 가능"으로 표시한다.

[인과 가용성] 원 실험과 동일. [필요 데이터·자원] 새 데이터 없음, CPU 3-seed run 2개. [중복 검사] 원 실험 행 ID.
```

## ARCH / ARCH-STEP 양식 (아키텍처 전체 피봇, 계보 예산제)
```
[모드] ARCH (새 계보 s1) | ARCH-STEP (활성 계보의 다음 단계)

[계보 ID] <계보 이름>   [단계] s<N> / 6

[전체 구조]
- 입력 표현 → 모델 → 출력(72h 속도 궤적 전체, 시간 단위)을 그림처럼 단계별로 쓴다. E0472 계열(per-lead XGBoost mixture + residual analog)과 무엇이 근본적으로 다른지 한 줄로 쓴다.
- ARCH-STEP: 직전 단계(E id, 코드 디렉터리 `leaderboard/experiments/arch_<계보>/`)에서 바꾸는 것 하나만 코드 수준으로 쓴다.

[입력·데이터 파이프라인]
- 쓰는 source와 표현(원시 시계열, 영상, 자기장 지도, 사건 목록 등), 인과 정렬 방법. 기존 데이터 로더·인과 특징 빌더 재사용은 허용한다.
- 금지: E0472 계열 모델의 예측·잔차·OOF를 입력이나 타깃으로 쓰는 것. E0472 계열과의 블렌딩·앙상블.

[학습 설계]
- 손실(궤적 MSE 등), 공식 창 이전 chronological 검증 분할(공식 창 정답으로 고르지 않음), seed 0/1/2, 옵티마이저·주요 하이퍼파라미터.
- DL은 GPU, seed당 ≥ 30분 또는 early stop 수렴, epoch별 valid loss를 `curves_s{seed}.json`에 저장. 3-seed가 워커 12시간 안에 끝나는 규모.

[귀납 편향 근거]
- 이 구조가 확정 챔피언의 오차 질량(고속 셀 4개·shock 창의 도착 시각, 72h level)을 per-lead XGBoost보다 잘 잡는 이유.

[가설·근거]
- LITERATURE(구조 전체가 비슷한 과제에서 낸 실측 성능, 링크, 공개 코드·하이퍼파라미터) 또는 MATH+PHYSICS(물리 메커니즘·정의·가정·추정량·반증 조건).
- 이 계보가 판별하는 architecture 가설과, 성공·실패 시 연구 방향이 어떻게 달라지는지.

[휴리스틱 검사]
- `references/heuristics.md` 자가 점검 3문항의 답. 입력 표현·출력 구조·결합에 사람이 정한 lag·창·임계값·분할·스칼라 요약·리드 경계·가중·grid가 없어야 한다. 남는 값은 "학습형" 또는 "표준 정의(출처)"로 표시한다.

[계보 목표와 단계 계획]
- 계보 목표: 현재 확정 챔피언을 재현 가능하게 넘는 독립 구조를 찾거나, 해당 architecture 가설의 가능성과 한계를 충분히 판별한다.
- s1은 확정 챔피언보다 나빠도 된다.
- 각 단계는 성능 개선 또는 계보의 핵심 불확실성 감소 중 적어도 하나를 만들어야 한다.
- s1~s6에서 각 단계가 바꿀 것 한 줄씩(예: s2 손실, s3 입력 표현 추가, s4 학습 단위 …). 다음 단계는 하네스가 결과를 보고 다시 정한다.

[인과 가용성]
- 모든 입력의 관측·발행 시각과 지연 규칙(ACE +2h 등), 학습 target < 2026-05-29, 라벨 성숙, poison check.

[실험]
- 리더보드 전체 구간 3-seed 단독 run → predictions_official_s{seed}.parquet → expos lb-eval. 비교: 확정 챔피언 lb-eval(SKILL 1단계에서 확정한 값)과 계보 직전 단계.
- 구현 고장은 끝까지 고쳐 완수한다.

[계보 종료·승격 규칙]
- 종료: 연속 단계에서 성능 개선도 없고, 사전 정의한 핵심 가설/정보 검정에서도 새 정보를 얻지 못해 더 진행할 연구 근거가 없을 때. 최대 6단계. `3%를 못 넘었다`는 이유만으로 초기 계보를 종료하지 않는다.
- 승격 대기: 확정 챔피언 대비 평균 −1.5% 이상 + 3 seed 모두 낮음 → REPLICATE.

[결과 기록] 첫 줄 `LB| ... type=arch lineage=<계보> stage=<N> d_prev=<값,%>`.

[중복 검사] evidence.md "과거 아키텍처 계보"(N1–N5)와 시트에서 같은 구조 계열을 쓴 이전 run(E id, lb-eval 값)을 모두 적고, 그 시도와 구조·학습 대상·입력 표현 중 무엇이 다른지 쓴다. 학습 규모만 다르면 새 계보가 아니다.
```

## 쓰면 안 되는 문구
C/A·O/A 비율 임계, "MSE<2000", late 감소율 임계, "smoke로 판정", "7일 CAL 블록으로 결정", "캠페인 종료 후 진행", "critic PASS 후 실행", 제출·업로드 지시.
손으로 정한 휴리스틱 지정과 그 값 변경: 고정 lag("26/27/28일"), rolling 창 길이, 임계값 기반 영역 추출, strip·sector·bin 개수, 시뮬레이션 출력의 k개 스칼라 요약, 리드·구간 경계, 수동 멤버 선택·고정 가중, analog K·lookback·거리, origin grid 간격(`references/heuristics.md`).
