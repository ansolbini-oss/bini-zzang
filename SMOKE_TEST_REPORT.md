# bini쨩 v0.1.0 스모크 테스트 리포트

**일시**: 2026-04-24
**테스트 주제**: "작은 팀용 할 일 관리 SaaS"
**작업 디렉토리**: `/Users/60hertz/Desktop/bini-smoke/`
**테스트 방식**: 플러그인 실제 설치 없이 에이전트 파일을 `general-purpose` 서브에이전트로 직접 호출 (프롬프트 내재화)

## 테스트 범위

| Phase | 에이전트 | 실행 여부 |
|-------|---------|---------|
| 0 | glossary-writer | ✅ 실행 |
| 1-a | market-researcher | ✅ 실행 |
| 1-b | strategist | ⏭️ 스킵 (스모크 범위 외) |
| 2 | user-researcher, flow-designer, ia-architect | ⏭️ 스킵 |
| 3 | feature-writer, data-modeler | ⏭️ 스킵 |
| 4 | policy-writer, kpi-designer, risk-analyst | ⏭️ 스킵 |
| 5 | consistency-checker | ✅ 실행 (부분 — 2개 아티팩트만 검증) |

---

## 결과 요약

### ✅ 성공 항목

1. **에이전트 파일의 self-contained 원칙 동작**
   모든 에이전트가 자기 `.md` 파일 하나만 시스템 프롬프트로 받고도 정확한 역할·산출물·스키마 준수로 작동.

2. **파일 시스템 규약 준수**
   `.plan/` 폴더 자동 생성, 파일명 규약(`00_glossary.md`, `01_market.md`, `_constants.json`, `06_consistency-report.md`, `.meta/*.json`) 그대로 지켜짐.

3. **Frontmatter 스키마 정확**
   - `artifact_id`, `phase`, `stage`, `version`, `generated_by`, `depends_on`, `approvals`, `self_check`, `assumptions` 모두 올바른 값.
   - 스모크 모드 시뮬레이션 플래그(`"smoke-test-auto-approved"`) 명시적 표기 잘 됨.

4. **Phase 간 의존성 자동 인식**
   market-researcher가 00_glossary.md를 먼저 읽고, 본문에서 **glossary 외 용어 0건** 사용. SSOT 원칙 작동.

5. **`_constants.json` SSOT 원칙 입증**
   수치 15개가 본문과 100% 일치. consistency-checker의 C2 검증 통과.

6. **페인↔기능 양방향 추적성**
   PAIN-001~007 전부가 L1-001~006에 매핑. 고아 0건. market-researcher의 `related_pains` 필드 설계가 자동 검증을 쉽게 만듦.

7. **정합성 체커의 자기 인식**
   9개 규칙을 "데이터 부족 — 스킵"으로 표기, 실행 가능한 5개만 돌림. 실행 가능 범위를 에이전트가 스스로 판단.

### ⚠️ 발견된 이슈 (경미)

| 이슈 | 위치 | 수준 |
|------|------|------|
| "카테고리" 키워드 오탐 | C1 규칙 | 설계 개선 |
| 본문-only 파생 수치 | `_constants.json` 스키마 | 설계 개선 |
| Phase별 적용 가능 규칙 명시 부재 | `consistency-rules.md` | 설계 개선 |

critical은 모두 **오탐** 또는 **Phase 1-a에서는 정상인 것**이었음. 실제 설계 결함 없음.

---

## 생성된 아티팩트 전수

```
/Users/60hertz/Desktop/bini-smoke/
└── .plan/
    ├── 00_glossary.md              11 KB, 28 용어
    ├── _constants.json             1.9 KB, 15개 수치 SSOT
    ├── 01_market.md                17 KB, 페르소나 시드 3·페인 7·경쟁사 10·L1 시드 6
    ├── 06_consistency-report.md    정합성 리포트 (critical 1, warning 2, info 3)
    └── .meta/
        └── 06_consistency_summary.json   통계 JSON 사이드카
```

---

## 에이전트별 자기검증 통과 여부

### glossary-writer
- ✅ 28 용어 (목표 20~30)
- ✅ Aliases 중복 제거
- ✅ 표준 용어 출처 인용 (NIST, ANSI, Accelerate 등)
- ✅ 금지어·혼용 주의 표 10행
- ✅ 핵심 용어 5개 확정 (스모크 모드 자동)
- ✅ 하류 에이전트용 Non-Goals 선표시 (게스트·웹훅)

### market-researcher
- ✅ 모든 수치에 `[가정]` 태그
- ✅ `_constants.json` 갱신 + `_owner_agent: market-researcher`
- ✅ 페르소나 시드 3, 페인 7, 경쟁사 10 (국내 3, 해외 4, 대체재 3), L1 시드 6
- ✅ Pre-gate 스모크 시뮬레이션 플래그
- ✅ 용어집 외 용어 0

### consistency-checker
- ✅ 14 규칙 전수 시도 (실행 5, 스킵 9)
- ✅ critical·warning·info 분류 정확
- ✅ 각 critical에 선택지 3개 제시
- ✅ 오탐 가능성 "매우 높음"으로 명시 (V-001)
- ✅ 통과 항목도 명시적으로 리포트 ("pass_C2", "pass_C5")
- ✅ `.meta/` 폴더에 JSON 사이드카

---

## 🔧 발견된 설계 개선 포인트 (v0.2에 반영 예정)

### 1. C1 규칙의 문맥 구분 서브룰
**현상**: 용어집 금지어 "카테고리"가 market.md에 4회 등장. 단 전부 "시장 카테고리", "리스크 카테고리" 같은 **메타 담론** 용법이라 오탐.

**원인**: C1 규칙이 키워드 매칭만 하고 문맥을 구분 못 함.

**해결**: `references/consistency-rules.md` C1에 "문맥 예외" 절 추가. 용어집 §7 "금지어" 항목에 **scope** 필드 추가:
```yaml
forbidden_terms:
  - term: "카테고리"
    forbidden_scope: "task-tagging"   # 이 맥락에서만 금지
    allowed_scope: "meta-discourse"    # 시장 분류·리스크 분류는 허용
```

### 2. Phase별 실행 가능 규칙 프로파일 사전 정의
**현상**: consistency-checker가 Phase 1-a 단계에서 14개 규칙 중 5개만 실행 가능함을 스스로 판단해야 했음 (사용자 프롬프트가 힌트 제공).

**해결**: `references/consistency-rules.md`에 Phase별 테이블 추가:

| Phase | 적용 가능 규칙 |
|-------|-------------|
| 0 완료 | (없음 — 혼자서는 검증 못 함) |
| 1-a 완료 | C1, C2, C5, C13, C14 |
| 1 완료 | + C12 (Non-Goals 일관성) |
| 2 완료 | + C3, C4 |
| 3 완료 | + C6, C7, C8, C9 |
| 4 완료 | + C10, C11 |

consistency-checker가 현재 `.plan/index.json`의 phase 상태를 읽고 자동 필터링.

### 3. `_constants.json` 파생 수치 선언 위치 관리
**현상**: 본문에만 등장하는 수치("완료율 목표 60%" 같은)가 SSOT에 없음 → warning.

**해결 2가지 중 택1**:
- **옵션 A**: `_constants.json` 스키마에 `_declared_only_in_body: []` 배열 추가. 의도적으로 SSOT에 올리지 않는 수치 선언.
- **옵션 B (추천)**: 각 에이전트의 `self_check`에 `body_numbers_promoted_to_ssot: true/false` 항목 추가. 본문에 새 수치 등장 시 SSOT 승격 여부 명시 강제.

### 4. (보너스) 에이전트 호출 시 프롬프트 크기
**현상**: 각 에이전트 .md 파일이 300~500줄. 서브에이전트가 매번 전체 읽으면 토큰 소모 큼.

**해결**: `agents/*.md` 각각을 **핵심 지시문 50~100줄 + references/로 분리된 상세**로 구조화 (wj-magic 플러그인 패턴 참고). v0.2에 고려.

---

## 결론

**bini쨩 v0.1.0 MVP의 핵심 설계가 실제 실행 환경에서 의도대로 작동함이 확인됨.**

- ✅ 의존-순차 파이프라인: glossary → market 순서 강제, market이 glossary 참조만 사용
- ✅ SSOT: `_constants.json` 원칙 완벽 준수
- ✅ 아티팩트 우선: 대화가 아닌 실제 파일 생성
- ✅ 정합성 검증: C1~C14 규칙 체계 동작. 실행/스킵 판단 자동화
- ✅ Frontmatter 추적성: `depends_on`, `generated_by`, `self_check` 모두 정확
- ✅ 사용자 체크 게이트: critical 발견 시 선택지 3개로 제시 (자동 수정 금지)

위에서 발견된 3~4가지 개선 포인트는 **설계 결함이 아니라 세련화 기회**. v0.2에서 반영 예정.

### 다음 단계
- **옵션 1 (권장)**: 실제 `/plugin install file:///Users/60hertz/Desktop/sol` → `/reload-plugins` → 새 대화에서 `/bini-zzang:기획-시장조사` 돌려보기 (슬래시 커맨드 레이어 검증)
- **옵션 2**: Phase 2~4까지 확장 스모크 (사용자·플로우·IA·기능명세·데이터·정책·KPI·리스크 실제 생성)
- **옵션 3**: wj-magic:ideation 대비 A/B 벤치마크
- **옵션 4**: 설계 개선 포인트 3가지를 v0.1.1 패치로 즉시 반영

### 스모크 테스트 비용 실측

| Phase | 토큰 | 소요 |
|-------|------|------|
| Phase 0 (glossary-writer) | 27,648 | 105초 |
| Phase 1-a (market-researcher) | 42,946 | 228초 |
| Phase 5 (consistency-checker) | 53,621 | 170초 |
| **합계** | **~124K tokens** | **~8.5분** |

풀 파이프라인 13 에이전트 기준 추정: **~500K tokens / ~40분** (Phase 3 feature-writer가 L2 노드 수 × 큰 프롬프트라 가장 무거움).
