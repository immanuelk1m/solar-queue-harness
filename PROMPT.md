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
결과 첫 줄에 SKILL.md에 적힌 버전(`solar-queue-harness v6.2`)을 출력하세요. v6.2가 아니면 실행하지 말고 그 사실만 알려 주세요.

- 시트: https://docs.google.com/spreadsheets/d/1eqmgRn3qJH_44Bkv14piwzhADU2fWZL8-p-d6EzF-nw/edit?gid=0#gid=0 (`태양풍 연구`, `태양풍 큐` 최신 원본 전체)
- 데이터셋 manifest: https://docs.google.com/spreadsheets/d/1KWgNQPlajHWw7FLUMz3lr4rO2z7sxuxXGstD0-YjAw4 (SolarWind DB; 사용 중·수집 중·평가된 데이터셋과 인과 규칙. 미러 `references/datasets.md`)
- 모델 스크립트: https://github.com/TLAB-HGU/sunrun-leaderboard/tree/main/scripts/seongeun
- 특정 E id, SOTA MSE, 목표 MSE, 목표 개선률을 프롬프트에 하드코딩하지 마세요.
- 매 실행마다 `태양풍 연구` 최신 원본 전체를 읽어 현재 유효 SOTA와 현재 평가 계약의 작업 챔피언을 스스로 확정하세요. 서로 다른 평가 계약의 점수는 억지로 하나의 순위로 합치지 마세요.
- 후보 생성과 우선순위는 사전 추정 MSE 개선률이 아니라 **정보이득, 미해결 가설의 판별력, 현재 병목과의 직접성, 기존 실험과의 독립성, 물리적 타당성**을 중심으로 판단하세요.
- 예상 MSE 개선률의 최소 임계값을 사용하지 마세요. 계산 근거가 충분한 경우 참고치로 기록할 수는 있지만 큐 진입 조건으로 사용하지 마세요.
- 축은 리더보드 전체 구간 lb-eval이나 충분한 사건 수의 정보 검정으로만 닫힙니다. smoke·내부 창·실행 실패·옛 3구간 평가 결과는 중복 근거가 아닙니다.
- **1.5단계 연구 건강 진단(H1–H8)을 반드시 코드로 계산해 표로 보이고, 발동한 신호의 큐 조치를 다른 규칙보다 먼저 따르세요.** 재현이 안 된 개선 위에 쌓지 말고(REPLICATE), 같은 유형이 포화됐으면 다른 층(예측 대상·구조)으로 전환하세요. H8(아키텍처/계보 고착)이 발동하면 지배 계보를 버린 **아키텍처 전체 피봇(ARCH) 계보**를 여세요(`references/pivot.md`, `row-template.md` ARCH 양식). 활성 계보가 있으면 다음 단계(ARCH-STEP)를 내세요. ARCH는 단독 판정만 하고 블렌딩하지 않습니다.
- **손으로 정한 휴리스틱은 제안하지 마세요**(`references/heuristics.md`): 고정 lag·rolling 창, 임계값·strip 요약, 시뮬레이션 출력의 스칼라 요약, 리드 경계, 수동 멤버·고정 가중, analog K·lookback, origin grid를 새로 넣거나 그 값을 바꾸는 행은 금지입니다. 챔피언의 휴리스틱을 학습형 구성요소로 바꾸는 행은 권장합니다. 문헌에 정의된 표준 물리량은 출처를 인용하면 허용합니다.
- 근거: 웹 검색으로 literature를 먼저 찾고 원문을 확인하세요. 채택할 논문이 부족하면 내부 결과와 domain knowledge를 사용해 **수학적 모델 + 태양풍 물리 메커니즘**을 함께 제시하세요.
- 물리적으로는 관측원/태양 표면 상태 → heliospheric propagation → arrival/regime → L1 72h trajectory 중 어느 경로를 다루는지 명시하세요.
- 수학적으로는 상태변수, 추정량/조건부 분포, 손실 또는 제약, 반증 가능한 예측을 명시하세요.
- exploitation이 필요하면 현재 작업 챔피언 위 단일 변경(INCREMENT)을 사용합니다.
- 연구 건강 진단이 포화·선택효과·아키텍처 고착을 보이면 PIVOT/ARCH/new-information 후보를 우선합니다.
- 같은 basin의 국소 변경만 반복하지 말고, 후보가 성공 또는 실패했을 때 어떤 큰 연구가설의 확률이 달라지는지를 명시하세요.
- 스킬의 후보 게이트를 통과한 행만 `태양풍 큐`에 최대 4건(4번째는 ARCH-STEP 전용) 추가하세요(`references/row-template.md` 양식). 0건으로 끝내지 말고, 통과 후보가 없으면 근거를 보강해 다시 시도한 뒤 그래도 없으면 현재 가장 큰 연구 불확실성을 줄일 수 있는 1건을 LOW-CONFIDENCE로 추가하세요. 마지막에 상태 표·추가 행 요약·기각 후보·검토 논문 표를 보여 주세요.
