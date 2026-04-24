# Phase 의존 그래프

## 원칙

1. **상류 에이전트 결과 없이 하류 실행 불가** — Pre-gate에서 차단
2. **같은 Phase 내 병렬 허용** — 컨텍스트는 공유
3. **Phase 간 건너뛰기 불가** — 단, 사용자가 명시적으로 스킵한 경우 해당 아티팩트는 `null`로 기록

## 의존 매트릭스

| 에이전트 | Phase | 필수 선행 | 선택적 선행 | 하류 영향 |
|----------|-------|----------|-----------|---------|
| glossary-writer | 0 | — | — | 전체 |
| market-researcher | 1 | 00_glossary | — | 전체 |
| strategist | 1 | 00_glossary, 01_market | — | 02 이후 전체 |
| user-researcher | 2 | 00, 01, 02 | — | 03_flows, 03_ia, 04 |
| flow-designer | 2 | 00, 02, 03_personas | 03_ia (병렬 시 스텁) | 04 |
| ia-architect | 2 | 00, 02, 03_personas | 03_flows (병렬 시 스텁) | 04 |
| feature-writer | 3 | 00, 02, 03_전체 | — | 04_data-model, 05 전체 |
| data-modeler | 3 | 00, 04_features | — | 05 |
| policy-writer | 4 | 00, 02, 03_ia, 04 | — | — |
| kpi-designer | 4 | 00, 01, 02 | — | — |
| risk-analyst | 4 | 00~04 전체 | — | — |
| consistency-checker | 5 | 00~05 전체 | — | revision 요청 |

## 병렬 실행 규칙

**Phase 2**: `user-researcher` → 먼저 완료되어야 함 (페르소나는 플로우·IA의 입력). 이후 `flow-designer`와 `ia-architect`는 동시 실행.

**Phase 4**: `policy-writer`, `kpi-designer`, `risk-analyst` 모두 동시 실행 가능. 모두 독립적으로 전체 상류 컨텍스트만 읽음.

## 재실행 정책

| 변경 유형 | 영향받는 Phase | 권장 조치 |
|----------|--------------|---------|
| 사용자 주제 변경 | 전체 | 처음부터 재실행 |
| 시장 데이터 변경 | 1~5 전체 | Phase 1부터 재실행 |
| 전략(Non-Goals) 변경 | 2~5 | Phase 2부터 재실행 |
| 페르소나 추가 | 2(부분), 3~5 | user-researcher + 하류 revision |
| 기능 1개 수정 | 3_features 부분, 4~5 부분 | feature-writer 단일 + 정합성 재검증 |

## ID 동결

재실행 시 기존 ID(SC-001, FL-001, F-001 등)는 **변경 금지**. 새 항목은 다음 빈 번호로 append. 삭제 필요 시 `status: deprecated` 상태로 보존(tombstone).
