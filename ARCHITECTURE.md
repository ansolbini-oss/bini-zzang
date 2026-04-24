# bini쨩 아키텍처

## 설계 원칙 5가지

### 1. 의존-순차 파이프라인 (Dependency-Ordered Pipeline)
기획은 선행 결과물에 의존한다. 시장조사 없이 전략 못 쓰고, 전략 없이 IA 못 그리고, IA 없이 기능명세 못 쓴다. 이 자연스러운 순서를 파이프라인으로 강제한다.

**병렬 가능한 구간은 여전히 병렬로 돌린다.** 단, 같은 Phase 내에서만.

### 2. 아티팩트 우선 (Artifact-First, Not Report-First)
최종 산출물은 "통합 리포트 한 장"이 아니라 **실제 사용 가능한 파일들**이다. 각 에이전트는 자기 아티팩트 파일을 직접 씀.

### 3. 단일 진실원 (Single Source of Truth, SSOT)
모든 에이전트가 참조하는 공유 컨텍스트가 있다:
- `.plan/00_glossary.md` — 용어 정의
- `.plan/index.json` — 프로젝트 상수(도메인, 타겟, 핵심 수치)
- `.plan/_constants.json` — 단일 진실원 수치(시장 규모·단가·KPI 목표값)

핵심 수치는 **이 파일들에서만** 정의되고, 다른 에이전트는 참조만 한다.

### 4. 정합성 능동 검증 (Active Consistency)
파이프라인 끝에 `consistency-checker` 에이전트가 **전수 교차검증**을 수행한다. 모순 발견 시:
1. `06_consistency-report.md`에 모순 내역 기록
2. PM 오케스트레이터가 사용자에게 경고 표시
3. **사용자 확인 후**에만 해당 에이전트 revision 또는 무시 결정
4. 자동 revision 안 함 (사용자 통제권 우선)

### 5. 3게이트 시스템 (Pre/Mid/Post Gates)
각 Phase마다:
- **Pre**: 범위·입력·가정 확인 → 불필요한 생성 차단
- **Mid**: 핵심 의사결정 지점에서 사용자 승인 → 하류 오염 방지
- **Post**: 완성 산출물 전체/부분/재실행 선택

---

## Phase 의존 그래프

```
Phase 0: 용어집 시드 (glossary-writer)
    ↓
Phase 1: 기반 분석 (순차)
    market-researcher → strategist
    ↓
Phase 2: 사용자·구조 (병렬, Phase 1 결과 input)
    user-researcher ‖ flow-designer ‖ ia-architect
    ↓
Phase 3: 기능·데이터 (순차)
    feature-writer → data-modeler
    ↓
Phase 4: 지원 산출물 (병렬, 전체 컨텍스트 input)
    policy-writer ‖ kpi-designer ‖ risk-analyst
    ↓
Phase 5: 정합성 검증
    consistency-checker → (모순 시) PM 사용자 체크 → revision
```

전 Phase 사이에 PM 오케스트레이터가 **게이트**를 운영한다. 사용자가 승인해야 다음 Phase로.

---

## 에이전트 13명 명세

| 에이전트 | Phase | 주 산출물 | 의존 |
|----------|-------|----------|------|
| `pm-orchestrator` | 0~5 | 게이트 승인 JSON, `index.json` 관리 | 전체 |
| `glossary-writer` | 0 | `00_glossary.md` | 사용자 주제만 |
| `market-researcher` | 1 | `01_market.md` | 00 |
| `strategist` | 1 | `02_strategy.md` | 00, 01 |
| `user-researcher` | 2 | `03_personas.md` | 00, 01, 02 |
| `flow-designer` | 2 | `03_flows/FL-*.md` | 00, 02, 03_personas |
| `ia-architect` | 2 | `03_ia.md`, `03_role-matrix.csv` | 00, 02, 03_personas |
| `feature-writer` | 3 | `04_features/F-*.md` | 00, 02, 03 전체 |
| `data-modeler` | 3 | `04_data-model.md` | 00, 04_features |
| `policy-writer` | 4 | `05_policies.md` | 00, 02, 03_ia, 04 |
| `kpi-designer` | 4 | `05_kpis.md` | 00, 01, 02 |
| `risk-analyst` | 4 | `05_risks.md` | 전체 |
| `consistency-checker` | 5 | `06_consistency-report.md` | 전체 |

---

## 아티팩트 파일 포맷

### 공통 Frontmatter (YAML)
```yaml
---
artifact_id: "F-001"            # 고유 ID (재실행 시 절대 변경 금지)
phase: 3                         # 생성 Phase
stage: "feature-spec"           # 에이전트 이름
version: "1.0"                  # Semver
generated_by: "feature-writer"
generated_at: "2026-04-23T17:00:00+09:00"
depends_on:                      # 선행 아티팩트 경로
  - ".plan/00_glossary.md"
  - ".plan/02_strategy.md"
  - ".plan/03_ia.md"
approvals:                       # 게이트 승인 이력
  pre: "2026-04-23T16:30:00+09:00"
  mid: null
  post: null
self_check:                      # 에이전트 자기검증 결과
  all_fields_present: true
  glossary_terms_consistent: true
  cross_references_valid: true
assumptions:                     # [가정] 태그된 항목들
  - "이행률 평균 92% 가정 (근거 없음)"
---
```

### 본문 규약
- 한국어 기본
- 구체 수치는 **`_constants.json`** 참조만 (하드코딩 금지)
- 주장은 반드시 출처(근거 문서 or `[가정]` or `[추론]`)
- 코드·플로우는 ``` 코드블록 또는 ```mermaid``` 다이어그램

---

## 모순 발견 시 사용자 체크 플로우

```
consistency-checker가 모순 감지
    ↓
06_consistency-report.md 기록 (severity: critical/warning/info)
    ↓
pm-orchestrator가 사용자에게 표시:
    "⚠ 모순 발견 N건
     1. [A.3] 시장 규모 4.5GW vs [B.7] 4.2GW (critical)
     2. [C.1] 페르소나 "김관리자" vs [D.4] "김소장" (warning)
     ...
     어떻게 할까요?
     [1] 해당 에이전트 revision 요청
     [2] 사용자가 직접 수정
     [3] 무시하고 계속 (사유 필요)"
    ↓
사용자 선택에 따라 분기
    ↓
선택 기록 → .plan/.decisions/06_consistency_<ISO>.json
```

**자동 revision 안 함.** 사용자가 판단해야 할 문제를 AI가 임의 수정하면 신뢰성 훼손.

---

## SSOT 파일 구조

### `.plan/_constants.json`
```json
{
  "domain": "수요반응(DR)",
  "target_market": "한국 KPX 수요반응자원 거래시장",
  "primary_users": ["산업체 공장", "상업 빌딩", "아파트 관리사무소"],
  "market_size_mw": 4500,
  "market_size_krw_billion_2024": 2000,
  "agg_share_percent_default": 25,
  "cbl_method": "High 5 of 10",
  "generated_by": "market-researcher",
  "last_updated_at": "2026-04-23T17:00:00+09:00"
}
```

다른 에이전트는 이 파일을 **읽기만** 한다. 쓰기 권한은 정의한 에이전트 하나만.

---

## 커맨드 구조

| 커맨드 | 동작 |
|-------|------|
| `/bini-zzang:기획 <주제>` | 풀 파이프라인 실행 (Phase 0~5) |
| `/bini-zzang:기획-시장조사` | Phase 1만 |
| `/bini-zzang:기획-ia` | Phase 2의 ia-architect만 |
| `/bini-zzang:기획-기능명세` | Phase 3만 |
| `/bini-zzang:기획-검증` | Phase 5만 (기존 산출물 재검증) |

---

## 도메인 확장

`references/domain-packs/<domain>.md` 에 도메인 특화 지식을 담으면, 에이전트가 자동으로 참조한다. 기본 제공:

- `generic.md` — 도메인 중립 기본값
- (로드맵) `dr.md`, `saas.md`, `fintech.md` 등

---

## 버전 전략

- `0.1.0`: MVP (13 에이전트, 의존 파이프라인, 정합성 체커, 사용자 체크 플로우)
- `0.2.0`: 도메인팩 3개 추가 (dr, saas, fintech)
- `0.3.0`: 벤치마크 자동화, wj-magic 대비 A/B 보고
- `1.0.0`: 마켓 정식 출시
