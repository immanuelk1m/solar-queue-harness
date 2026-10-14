solar-queue-harness 스킬의 절차를 처음부터 끝까지 그대로 실행하세요.

**먼저 최신 스킬 원본을 GitHub에서 직접 열어 읽으세요.** 업로드되거나 기억된 사본은 오래됐을 수 있으니 쓰지 마세요.
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/SKILL.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/contract.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/evidence.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/row-template.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/literature.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/pivot.md
- https://raw.githubusercontent.com/immanuelk1m/solar-queue-harness/main/references/diagnostics.md
결과 첫 줄에 SKILL.md에 적힌 버전(`solar-queue-harness v4`)을 출력하세요. v4가 아니면 실행하지 말고 그 사실만 알려 주세요.

- 시트: https://docs.google.com/spreadsheets/d/1eqmgRn3qJH_44Bkv14piwzhADU2fWZL8-p-d6EzF-nw/edit?gid=0#gid=0 (`태양풍 연구`, `태양풍 큐` 최신 원본 전체)
- 모델 스크립트: https://github.com/TLAB-HGU/sunrun-leaderboard/tree/main/scripts/seongeun
- 목표: 고정 기준 SOTA E0472(3281.307) 대비 리더보드 전체 구간 3-seed 평균 MSE 10% 개선(≤ 2953.18). 한 번에 달성하지 않아도 됩니다. 각 실험이 **작업 챔피언**(ACE+2h 규칙을 지킨 lb-eval 최고값, 기본 E1022 3421.806)을 3% 이상 낮추고, 그 개선이 쌓이도록 설계하세요. 3% 게이트를 E0472 기준으로 환산하지 마세요.
- 기대이득은 순위를 매기기 위한 추정입니다. 대상 오차 성분(오라클 headroom·오차 질량·ablation) × 회수 가능 비율(근거 명시)로 계산하세요. 판정은 lb-eval이 합니다.
- 축은 리더보드 전체 구간 lb-eval이나 충분한 사건 수의 정보 검정으로만 닫힙니다. smoke·내부 창·실행 실패·옛 3구간 평가 결과는 중복 근거가 아닙니다.
- **1.5단계 연구 건강 진단(H1–H8)을 반드시 코드로 계산해 표로 보이고, 발동한 신호의 큐 조치를 다른 규칙보다 먼저 따르세요.** 재현이 안 된 개선 위에 쌓지 말고(REPLICATE), 같은 유형이 포화됐으면 다른 층(예측 대상·구조)으로 전환하세요. H8(아키텍처 고착)이 발동하면 E0472 계열을 버린 **아키텍처 전체 피봇(ARCH) 계보**를 여세요(`references/pivot.md`, `row-template.md` ARCH 양식). 활성 계보가 있으면 다음 단계(ARCH-STEP)를 내세요. ARCH는 단독 판정만 하고 블렌딩하지 않습니다.
- 근거: 웹 검색으로 literature를 먼저 찾고 원문을 확인하세요. 채택할 논문이 부족하면 내부 수치에서 유도한 수학적 방법론을 제시하세요.
- 기본은 현재 챔피언 위의 단일 변경(INCREMENT)입니다. 스택 후보가 2개 이상이면 COMBINE 1건, 정체 카운터 ≥ 8이면 PIVOT 1건까지 허용합니다.
- 스킬의 후보 게이트를 통과한 행만 `태양풍 큐`에 최대 4건(4번째는 ARCH-STEP 전용) 추가하세요(`references/row-template.md` 양식). 0건으로 끝내지 말고, 통과 후보가 없으면 근거를 보강해 다시 시도한 뒤 그래도 없으면 최선 1건을 LOW-CONFIDENCE로 추가하세요. 마지막에 상태 표·추가 행 요약·기각 후보·검토 논문 표를 보여 주세요.
