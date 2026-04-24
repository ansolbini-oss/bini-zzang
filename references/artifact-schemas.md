# 아티팩트 스키마

모든 아티팩트는 **YAML Frontmatter + 본문 + (선택) JSON 사이드카**로 구성된다.

## 공통 Frontmatter 필드

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `artifact_id` | string | ✓ | 고유 ID (F-001, FL-002 등). 재실행 시 변경 금지 |
| `phase` | int | ✓ | 생성 Phase (0~5) |
| `stage` | string | ✓ | 에이전트 이름 (market-researcher 등) |
| `version` | string | ✓ | Semver (1.0, 1.1 등) |
| `generated_by` | string | ✓ | 생성 에이전트 |
| `generated_at` | ISO8601 | ✓ | 생성 시각 |
| `depends_on` | array | ✓ | 선행 아티팩트 파일 경로 목록 |
| `approvals` | object | ✓ | `{pre, mid, post}` 각 게이트의 사용자 승인 ISO 타임스탬프 또는 null |
| `self_check` | object | ✓ | 에이전트 자기검증 결과 (bool 플래그들) |
| `assumptions` | array | ✓ | `[가정]` 태그된 항목 요약 |
| `open_questions` | array |   | 미해결 질문 |

## Phase별 산출물 스키마

### Phase 0: `00_glossary.md`
```yaml
---
artifact_id: GLOSSARY
phase: 0
stage: glossary-writer
depends_on: []
terms:
  - term: "수요반응 (DR)"
    definition: "..."
    aliases: ["Demand Response"]
    source: "전력시장운영규칙 §12"
  - term: "CBL (Customer Baseline Load)"
    definition: "..."
    aliases: ["기준부하", "베이스라인"]
    source: "KPX 가이드"
---

# 용어집

(본문 — 섹션별 정리)
```

### Phase 1-a: `01_market.md`
```yaml
---
artifact_id: MARKET
phase: 1
stage: market-researcher
depends_on: [".plan/00_glossary.md"]
personas_seeded:   # 후속 user-researcher 참조용
  - id: "P-001-seed"
    role: "산업체 에너지 관리자"
pain_points:       # 페인 ID 동결
  - id: "PAIN-001"
    label: "전기요금 연 6억+, 피크 요금 폭등"
    persona_seed: "P-001-seed"
l1_features_seed:
  - id: "L1-001"
    name: "감축 수익 창출"
    related_pains: ["PAIN-001"]
---
```

### Phase 1-b: `02_strategy.md`
```yaml
---
artifact_id: STRATEGY
phase: 1
stage: strategist
depends_on: [".plan/00_glossary.md", ".plan/01_market.md"]
positioning:
  chosen_option: "B"   # 3안 중 사용자 선택
  alternatives: [A, B, C]
  rationale: "..."
value_proposition: "..."
target_segments_primary: ["중소 공장"]
non_goals:
  - "가정용 B2C (Phase 1 scope 외)"
scenarios:
  - id: "SC-001"
    title: "공장 감축 이벤트 대응"
    trigger: "KPX 이벤트 발령"
    type: "happy_path"
    persona_seed: "P-001-seed"
    linked_l1: ["L1-001"]
---
```

### Phase 2-a: `03_personas.md`
```yaml
---
artifact_id: PERSONAS
phase: 2
stage: user-researcher
depends_on: [".plan/00_glossary.md", ".plan/01_market.md", ".plan/02_strategy.md"]
personas:
  - id: "P-001"
    name: "김공장"
    role: "중견 제조업 에너지 관리자"
    demographics: "..."
    goals: ["..."]
    pains: ["PAIN-001"]
    jtbd: "내가 [상황]일 때, [동기]하고 싶어서 [결과]이 되게 한다"
---
```

### Phase 2-b: `03_flows/FL-*.md`
```yaml
---
artifact_id: FL-001
phase: 2
stage: flow-designer
depends_on: [".plan/02_strategy.md", ".plan/03_personas.md"]
scenario_id: SC-001
primary_persona: P-001
---

# FL-001: 공장 DR 이벤트 대응 플로우

## 엔트리 포인트
## Happy Path
## 예외 분기
## 상태 전이
(Mermaid 다이어그램)
```

### Phase 2-c: `03_ia.md`
```yaml
---
artifact_id: IA
phase: 2
stage: ia-architect
depends_on: [".plan/02_strategy.md", ".plan/03_personas.md"]
roles:
  - id: "R-001"
    name: "DR 운영자"
    persona_map: ["P-001"]
feature_tree:
  - id: "L1-001"
    name: "감축 수익 창출"
    children:
      - id: "L2-001"
        name: "이벤트 응동"
        scenarios: ["SC-001"]
        children:
          - id: "L3-001"
            name: "원탭 참여 버튼"
---
```

### Phase 3-a: `04_features/F-*.md` (15 섹션)
```yaml
---
artifact_id: F-001
phase: 3
stage: feature-writer
depends_on: [".plan/03_ia.md", ".plan/02_strategy.md"]
ia_node_id: L2-001
scenarios: [SC-001]
priority: P0
roles_allowed: [R-001]
---

# F-001: 이벤트 응동

## 1. 개요
## 2. 유저 스토리
## 3. 인수 기준 (Given-When-Then)
## 4. 화면 요소
## 5. 데이터 모델
## 6. API 힌트
## 7. 비즈니스 규칙
## 8. 상태 전이
## 9. 엣지 케이스
## 10. 권한
## 11. 로깅·감사
## 12. 비기능 요구사항 (NFR)
## 13. 의존성
## 14. Non-Goals
## 15. ADR (의사결정 기록)
```

### Phase 3-b: `04_data-model.md`
```yaml
---
artifact_id: DATA_MODEL
phase: 3
stage: data-modeler
depends_on: [".plan/04_features/"]
entities:
  - name: "DrEvent"
    fields: [...]
    relations: [...]
---

(ERD Mermaid + 엔티티별 상세)
```

### Phase 4-a: `05_policies.md`
```yaml
---
artifact_id: POLICIES
phase: 4
stage: policy-writer
depends_on: [".plan/02_strategy.md", ".plan/03_ia.md", ".plan/04_features/"]
policies:
  - id: "POL-001"
    category: "business_rule"
    statement: "정산금은 감축량 × 단가 × 분배율 × (1 - 세금률)"
    sources: ["KPX 운영규칙 §12.3"]
rules_of_permission:
  - id: "PERM-001"
    role: "R-001"
    can: ["view_event", "approve_dispatch"]
glossary_additions: []   # 정책 작성 중 나온 새 용어
---
```

### Phase 4-b: `05_kpis.md`
```yaml
---
artifact_id: KPIS
phase: 4
stage: kpi-designer
depends_on: [".plan/01_market.md", ".plan/02_strategy.md"]
north_star:
  name: "월 누적 감축 용량 (MW-hours)"
  target_year_1: "25MW·h"
  target_year_3: "180MW·h"
leading_indicators:
  - name: "이행률"
    target: ">= 95%"
    measured_in_feature: F-001
  - name: "온보딩 리드타임"
    target: "<= 45일"
---
```

### Phase 4-c: `05_risks.md`
```yaml
---
artifact_id: RISKS
phase: 4
stage: risk-analyst
depends_on: ["전체"]
risks:
  - id: "R-001"
    category: "regulatory"
    description: "KPX 단가 인하 추세"
    probability: "high"
    impact: "high"
    mitigation: "SaaS 구독료 비중 확대"
    mitigation_feature: F-XXX
---
```

### Phase 5: `06_consistency-report.md`
(`consistency-rules.md` 참조)

## SSOT 파일: `_constants.json`

```json
{
  "_schema_version": "1.0",
  "domain": "수요반응(DR)",
  "target_market": "...",
  "market_size_mw": 4500,
  "market_size_krw_billion_2024": 2000,
  "primary_users": ["..."],
  "key_kpis": {
    "north_star_name": "월 누적 감축 용량",
    "target_year_1": 25,
    "target_year_3": 180
  },
  "_owner_agent": "market-researcher",
  "_last_updated_at": "2026-04-23T17:00:00+09:00"
}
```

## `index.json` (프로젝트 상태 메타)

```json
{
  "project_name": "수요반응 DR 운영 플랫폼",
  "created_at": "2026-04-23T16:00:00+09:00",
  "last_updated_at": "2026-04-23T18:00:00+09:00",
  "phases": {
    "0": {"status": "complete", "artifact": "00_glossary.md"},
    "1": {"status": "complete", "artifacts": ["01_market.md", "02_strategy.md"]},
    "2": {"status": "in_progress", "artifacts": ["03_personas.md"]},
    "3": {"status": "pending"},
    "4": {"status": "pending"},
    "5": {"status": "pending"}
  },
  "coverage": {
    "l1_covered": 5,
    "l2_covered": 12,
    "l3_covered": 0,
    "total_l1": 7,
    "total_l2": 18,
    "total_l3": 35
  },
  "consistency_status": {
    "last_run": null,
    "critical_open": 0,
    "warning_open": 0
  }
}
```
