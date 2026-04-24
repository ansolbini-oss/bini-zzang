# 정합성 검증 규칙 (consistency-checker 가이드)

이 문서는 `consistency-checker` 에이전트가 Phase 5에서 실행하는 **교차검증 체크리스트**다. 규칙별로 심각도(critical/warning/info)를 정의하고, 사용자 확인이 필요한 수준을 명시한다.

## 심각도 정의

| 심각도 | 의미 | 처리 |
|-------|------|------|
| 🔴 critical | 논리적 모순. 하나가 참이면 다른 하나는 반드시 거짓 | **사용자 체크 필수**, 미해결 시 Post-gate 통과 불가 |
| 🟠 warning | 일관성 저하. 치명적이진 않지만 품질 훼손 | 사용자 고지 후 계속 여부 결정 |
| 🟡 info | 개선 제안. 형식·네이밍·스타일 | 리포트에만 기록, 진행에 영향 없음 |

## 검증 규칙 카탈로그

### C1. 용어 일관성 (🔴 critical)
- 모든 아티팩트에서 `00_glossary.md` 정의 용어만 사용하는가?
- 동일 개념이 다른 이름으로 등장하지 않는가? (예: "운영자" vs "관리자" vs "담당자")
- 용어집에 없는 고유명사·약어가 본문에 등장하면 용어집에 추가 필요

### C2. 수치 일관성 (🔴 critical)
- `_constants.json`의 핵심 수치가 아티팩트마다 동일한가?
- 시장 규모·단가·KPI 목표값이 여러 곳에서 다르게 적혀있지 않은가?
- 계산값(예: 연 수익 = 용량 × 단가 × 12개월)이 논리 식과 일치하는가?

### C3. 페르소나 참조 정합성 (🔴 critical)
- `03_personas.md`에 정의된 페르소나 ID만 다른 아티팩트에서 참조하는가?
- 플로우(FL-*)의 주체, 기능명세(F-*)의 사용자, 권한 매트릭스 행이 페르소나와 일치하는가?

### C4. 시나리오↔플로우 매핑 (🔴 critical)
- `02_strategy.md` Happy Path 시나리오가 모두 `03_flows/FL-*.md`에 존재하는가?
- 플로우의 `scenario_id` frontmatter가 유효한 시나리오를 가리키는가?

### C5. 페인↔기능 추적성 (🟠 warning)
- `01_market.md`의 모든 페인이 최소 1개 L2 기능에 매핑되는가?
- 매핑 안 된 페인은 "의도적 제외"인지 `02_strategy.md`의 Non-Goals에 명시되었는가?

### C6. IA↔기능명세 일대일 (🔴 critical)
- `03_ia.md`의 L2 기능 노드 각각에 `04_features/F-*.md` 파일이 존재하는가?
- 기능명세 파일의 `ia_node_id` frontmatter가 유효한 IA 노드를 가리키는가?
- IA에 없는 기능이 명세되지 않았는가?

### C7. 권한 일관성 (🔴 critical)
- `03_ia.md` 역할 매트릭스의 역할이 기능명세 권한 섹션과 일치하는가?
- 권한 경계(누가 어떤 작업 가능한지)가 여러 아티팩트에서 모순되지 않는가?

### C8. 데이터 모델 정합성 (🔴 critical)
- `04_data-model.md`의 엔티티가 기능명세의 "데이터 모델" 섹션과 일치하는가?
- 같은 엔티티 필드가 다르게 정의되지 않았는가? (타입·길이·nullable)
- 참조 무결성(외래키)이 실제 상위 테이블 존재 여부와 일치하는가?

### C9. 상태전이 일관성 (🔴 critical)
- 기능명세의 상태전이표 간 모순 없는가?
- IA의 플로우 상태와 기능명세 상태 머신이 일치하는가?
- 정책·규제(05_policies.md)의 상태 요구사항과 충돌하지 않는가?

### C10. KPI↔기능 연결 (🟠 warning)
- `05_kpis.md`의 각 지표가 어느 기능에서 측정되는지 `04_features/`에 로깅 요구사항이 있는가?
- 측정 불가능한 KPI는 warning으로 표시

### C11. 리스크↔완화책 (🟡 info)
- `05_risks.md`의 각 리스크에 대응 완화책(기능·정책·운영 절차)이 매핑되었는가?
- 완화 없는 리스크는 수용 리스크(accepted risk)로 명시되었는가?

### C12. Non-Goals 일관성 (🟠 warning)
- `02_strategy.md`에서 Non-Goals로 지정한 항목이 어딘가에서 구현되지 않았는가?
- 예: "가정용 B2C Non-Goals" → 기능명세에 가정용 기능이 들어가면 warning

### C13. [가정]·[추론] 태그 누락 (🟡 info)
- 근거 없는 단정이 `[가정]`·`[추론]` 태그 없이 본문에 등장하는가?
- 공식 출처 인용이 누락된 수치가 있는가?

### C14. 의사결정 근거(ADR) 완결성 (🟠 warning)
- Phase 2·3의 주요 선택(포지셔닝·데이터모델 등)에 ADR 형식 결정 기록이 있는가?
- 각 ADR에 대안·트레이드오프·결정·이유·철회 조건이 모두 있는가?

---

## 검증 리포트 포맷 (`06_consistency-report.md`)

```markdown
---
phase: 5
generated_by: consistency-checker
generated_at: 2026-04-23T18:00:00+09:00
rule_total: 14
violations_critical: 2
violations_warning: 5
violations_info: 3
---

# 정합성 검증 리포트

## 요약
- 🔴 Critical: 2건 — **사용자 확인 필수**
- 🟠 Warning: 5건
- 🟡 Info: 3건

## Critical 위반 상세

### V-001 (C2 수치 일관성)
- 위치: `01_market.md:42` vs `_constants.json:market_size_mw`
- 내용: market.md 본문엔 "약 4.5 GW", _constants.json엔 `4500` (일치)
- 판정: 일치 — 오탐
- 처리: 무시

### V-002 (C3 페르소나 참조 정합성)
- 위치: `03_flows/FL-003.md:8`
- 내용: 페르소나 ID "P-005" 참조하지만 `03_personas.md`엔 P-004까지만 정의됨
- 처리 후보:
  - [1] `user-researcher`에게 P-005 정의 추가 요청
  - [2] FL-003의 주체를 P-004 중 하나로 수정
  - [3] P-005는 실제로 있어야 함을 명시 (user-researcher 재호출)

## Warning 위반 상세
(...)
```

---

## 사용자 체크 대화 포맷

critical 발견 시, `pm-orchestrator`가 다음 포맷으로 사용자에게 질문한다:

```
⚠ 정합성 검증에서 🔴 Critical 모순 N건 발견

[V-002] 페르소나 ID 불일치
  위치: FL-003.md:8이 P-005를 참조하지만 personas.md엔 없음

어떻게 처리할까요?
  [1] user-researcher에게 P-005 정의 추가 요청 (권장)
  [2] FL-003 수정 (P-001~P-004 중 선택)
  [3] 무시하고 계속 (사유 입력)

→ 번호 선택:
```

사용자 선택은 `.plan/.decisions/06_consistency_<ISO>.json`에 기록되어 감사 가능.
