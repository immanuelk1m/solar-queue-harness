solar-queue-harness 스킬의 절차를 처음부터 끝까지 그대로 실행하세요.

**먼저 최신 스킬 원본을 GitHub에서 직접 열어 읽으세요.** 업로드되거나 기억된 사본은 오래됐을 수 있으니 쓰지 마세요.
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/SKILL.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/contract.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/evidence.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/row-template.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/literature.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/pivot.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/diagnostics.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/heuristics.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/datasets.md
결과 첫 줄에 SKILL.md에 적힌 버전(`solar-queue-harness v6.3`)을 출력하세요. v6.3이 아니면 실행하지 말고 그 사실만 알려 주세요.

- 시트: https://docs.google.com/spreadsheets/d/1eqmgRn3qJH_44Bkv14piwzhADU2fWZL8-p-d6EzF-nw/edit?gid=0#gid=0 (`태양풍 연구`, `태양풍 큐` 최신 원본 전체)
- 데이터셋 manifest: https://docs.google.com/spreadsheets/d/1KWgNQPlajHWw7FLUMz3lr4rO2z7sxuxXGstD0-YjAw4 (SolarWind DB; 사용 중·수집 중·평가된 데이터셋과 인과 규칙. 미러 `references/datasets.md`)
- 모델 스크립트: https://github.com/TLAB-HGU/sunrun-leaderboard/tree/main/scripts/seongeun
- 특정 E id, SOTA MSE, 목표 MSE, 목표 개선률을 프롬프트에 하드코딩하지 마세요.
- 매 실행마다 `태양풍 연구` 최신 원본 전체를 읽어 현재 유효 SOTA와 현재 평가 계약의 작업 챔피언을 스스로 확정하세요. 서로 다른 평가 계약의 점수는 억지로 하나의 순위로 합치지 마세요.
- 예측 후보는 반드시 SKILL.md의 **A-GATE(A1–A5)**를 통과해야 합니다. 25–72h event phase / heliospheric propagation / arrival uncertainty를 더 식별할 새 인과 정보가 없는 same-input loss·target·representation·latent-state·architecture·weighting·sampling 변경은 B/C로 기각하고 예측 실험 큐에 넣지 마세요.
- 정보이득, 가설 판별력, 병목 직접성, 독립성, 물리적 타당성은 **A등급 후보끼리의 우선순위**를 정하는 기준으로 사용하세요. 정보이득만으로 B/C 예측 후보를 승격시키지 마세요.
- 예상 MSE 개선률의 최소 임계값을 사용하지 마세요. 계산 근거가 충분한 경우 참고치로 기록할 수는 있지만 큐 진입 조건으로 사용하지 마세요.
- 축은 리더보드 전체 구간 lb-eval이나 충분한 사건 수의 정보 검정으로만 닫힙니다. smoke·내부 창·실행 실패·옛 3구간 평가 결과는 중복 근거가 아닙니다.
- **1.5단계 연구 건강 진단(H1–H8)을 반드시 코드로 계산해 표로 보이세요.** H 신호는 탐색 방향을 바꾸지만 A-GATE를 우회할 수 없습니다. H8이 발동해도 architecture-only ARCH를 강제 생성하지 말고, 새 propagation/arrival 정보를 포함해 A-GATE를 통과하는 경우에만 ARCH/ARCH-STEP을 내세요.
- **손으로 정한 휴리스틱은 제안하지 마세요**(`references/heuristics.md`): 고정 lag·rolling 창, 임계값·strip 요약, 시뮬레이션 출력의 스칼라 요약, 리드 경계, 수동 멤버·고정 가중, analog K·lookback, origin grid를 새로 넣거나 그 값을 바꾸는 행은 금지입니다. 챔피언의 휴리스틱을 학습형 구성요소로 바꾸는 행은 권장합니다. 문헌에 정의된 표준 물리량은 출처를 인용하면 허용합니다.
- 근거: 웹 검색으로 literature를 먼저 찾고 원문을 확인하세요. 채택할 논문이 부족하면 내부 결과와 domain knowledge를 사용해 **수학적 모델 + 태양풍 물리 메커니즘**을 함께 제시하세요.
- 물리적으로는 관측원/태양 표면 상태 → heliospheric propagation → arrival/regime → L1 72h trajectory 중 어느 경로를 다루는지 명시하세요.
- 수학적으로는 상태변수, 추정량/조건부 분포, 손실 또는 제약, 반증 가능한 예측을 명시하세요.
- exploitation이 필요하면 현재 작업 챔피언 위 단일 변경(INCREMENT)을 사용합니다.
- 연구 건강 진단이 포화·선택효과·아키텍처 고착을 보이면 PIVOT/ARCH/new-information 후보를 우선합니다.
- 같은 basin의 국소 변경만 반복하지 말고, 후보가 성공 또는 실패했을 때 어떤 큰 연구가설의 확률이 달라지는지를 명시하세요.
- A-GATE와 후보 게이트를 모두 통과한 A등급 예측 행만 `태양풍 큐`에 최대 4건 추가하세요(`references/row-template.md`). **0건으로 끝내지 않습니다.** 다만 A등급 예측 후보가 없을 때 B/C LOW-CONFIDENCE 예측 실험을 억지로 넣지 말고, 목표 A급 실험의 blocker를 제거하는 ENABLE-DATA / ENABLE-PHYSICS / INFO-TEST 해결 작업(시트에 있는 데이터만 사용, 내려받기·새 원천 탐색 금지)을 최소 1건 추가하세요. 시트에 없는 데이터가 필요하면 행을 내지 말고 '데이터 추가 제안'으로만 적으세요. 마지막에 상태 표·추가 행 요약·기각 후보·검토 논문 표를 보여 주세요.
