# bini쨩 테스트 가이드

이 문서는 bini쨩(`bini-zzang`) v0.1.0 MVP를 **로컬 설치**하고 **실제 동작 확인**하는 방법을 설명한다.

---

## 1. 로컬 설치

### 1-A. 프로젝트를 직접 플러그인으로 등록

Claude Code는 로컬 경로의 플러그인을 설치할 수 있다.

```bash
# 옵션 1: 슬래시 커맨드
/plugin install file:///Users/60hertz/Desktop/sol

# 옵션 2: 설정 파일 직접 편집
# ~/.claude/settings.json 에 추가:
{
  "plugins": [
    {
      "source": "local",
      "path": "/Users/60hertz/Desktop/sol"
    }
  ]
}

# 재시작 or /reload-plugins
```

### 1-B. 설치 검증

```bash
/help
```

출력에 아래 커맨드가 나타나야 한다:
- `/bini-zzang:기획`
- `/bini-zzang:기획-시장조사`
- `/bini-zzang:기획-ia`
- `/bini-zzang:기획-기능명세`
- `/bini-zzang:기획-검증`

에이전트 목록에도 13명이 등록되어야 함:
```
Agent list에서 확인:
- pm-orchestrator
- glossary-writer
- market-researcher
- strategist
- user-researcher
- flow-designer
- ia-architect
- feature-writer
- data-modeler
- policy-writer
- kpi-designer
- risk-analyst
- consistency-checker
```

---

## 2. 스모크 테스트 (5분, 기본 동작)

### 2-1. 빈 디렉토리에서 Phase 0~1 실행

```bash
mkdir ~/Desktop/bini-test && cd ~/Desktop/bini-test
# Claude Code 재진입 (디렉토리 변경 반영)
```

Claude Code 대화창에서:
```
/bini-zzang:기획-시장조사
```

**예상 동작**:
1. PM 오케스트레이터가 주제 묻기 → 예: "팀 단위 할 일 관리 SaaS"
2. Pre-gate: 도메인·타겟 사용자 질문
3. `glossary-writer` 호출 → `~/.plan/00_glossary.md` 생성
4. 핵심 용어 5개 확인 요청
5. `market-researcher` 호출 → `01_market.md` + `_constants.json`
6. Mid-gate에서 페르소나·페인 검토
7. `strategist` 호출 → 포지셔닝 3안 제시
8. 사용자 선택 → `02_strategy.md` 완성

**검증 체크**:
- [ ] `.plan/` 폴더가 현재 디렉토리 아래에 생성됐는가?
- [ ] 각 아티팩트 파일이 실제로 있는가?
- [ ] Frontmatter의 `depends_on` 필드가 정확히 선행 파일을 가리키는가?
- [ ] `_constants.json`의 핵심 수치가 `01_market.md` 본문과 일치하는가?

### 2-2. Phase 2 이어서

```
/bini-zzang:기획-ia
```

**예상**:
1. `01_market.md`, `02_strategy.md` 존재 검증
2. `user-researcher` → `03_personas.md`
3. `flow-designer` + `ia-architect` 병렬 호출
4. `03_flows/FL-*.md`, `03_ia.md`, `03_role-matrix.csv` 생성

**검증**:
- [ ] 페르소나 ID가 플로우·IA에서 일관 참조되는가?
- [ ] Mermaid 다이어그램이 렌더링 가능 문법인가?
- [ ] 권한 매트릭스에 빈 셀·모순 없는가?

### 2-3. 정합성 검증

```
/bini-zzang:기획-검증
```

**예상**:
1. `consistency-checker` 호출
2. `06_consistency-report.md` 생성
3. Critical 발견 시 사용자 체크 대화

**의도적 모순 삽입으로 테스트**:
1. `03_personas.md`에 정의된 페르소나 ID를 편집기로 변경 (예: P-001 → P-099)
2. 다시 `/bini-zzang:기획-검증` 실행
3. C3(페르소나 참조) critical 위반 감지되는지 확인
4. 선택지 3개가 명확하게 나오는지 확인

---

## 3. 풀 파이프라인 테스트 (30~60분)

### 3-1. 실제 기획 주제로 풀 실행

```bash
mkdir ~/Desktop/bini-fulltest && cd ~/Desktop/bini-fulltest
```

```
/bini-zzang:기획 수요반응 DR 운영 플랫폼
```

파이프라인이 Phase 0~5까지 순서대로 돌아야 한다.

**각 Phase 완료 후 검증**:

| Phase | 산출물 확인 | 품질 체크 |
|-------|----------|---------|
| 0 | `00_glossary.md` | 핵심 용어 20~40개, 중복 0 |
| 1 | `01_market.md`, `02_strategy.md`, `_constants.json` | 포지셔닝 3안 제시, 선택된 안만 상세화 |
| 2 | `03_personas.md`, `03_flows/`, `03_ia.md` | 페르소나 ID 일관, L1 5~9개 |
| 3 | `04_features/F-*.md`, `04_data-model.md` | 각 F-* 15섹션, ADR 최소 1개 |
| 4 | `05_policies.md`, `05_kpis.md`, `05_risks.md` | 규제 태그 출처 명시, NSM 1개 |
| 5 | `06_consistency-report.md` | C1~C14 전수 실행, critical·warning·info 분류 |

### 3-2. `index.json` 최종 상태 확인

```bash
cat .plan/index.json | jq .
```

출력 예:
```json
{
  "phases": {
    "0": {"status": "complete"},
    "1": {"status": "complete"},
    "2": {"status": "complete"},
    "3": {"status": "complete"},
    "4": {"status": "complete"},
    "5": {"status": "complete"}
  },
  "coverage": {"l1_covered": 7, "l2_covered": 18, "l3_covered": 30, "total_l2": 18},
  "consistency_status": {"critical_open": 0, "warning_open": 3}
}
```

---

## 4. wj-magic 대비 벤치마크 (수동)

### 4-1. 같은 주제로 wj-magic 실행

```bash
mkdir ~/Desktop/wj-compare && cd ~/Desktop/wj-compare
/wj-magic:ideation 수요반응 DR 운영 플랫폼
```

### 4-2. 정량 비교

| 항목 | wj-magic | bini-zzang |
|-----|---------|-----------|
| 생성 파일 수 | ? | 15+ |
| 용어 일관성 (같은 용어 쓰기) | ? | 100% (glossary 준수) |
| 수치 일관성 (같은 수치 일치) | ? | 100% (`_constants.json`) |
| 교차 참조 유효성 | ? | consistency-checker 검증 |
| 의사결정 근거 (ADR) | ? | 각 주요 선택 ADR 필수 |
| 재실행 시 ID 안정성 | 변동 가능 | 동결 |

### 4-3. 정성 비교

- 읽고 "실제로 개발 착수 가능한가?" — 어느 쪽 산출물이 바로 쓸만한가?
- 수정 발생 시 어느 쪽이 영향 범위 추적 쉬운가?
- 사용자 판단권이 존중되는가?

---

## 5. 트러블슈팅

### `/bini-zzang:...` 커맨드가 안 보임
- `/reload-plugins` 실행
- `~/.claude/settings.json`에 플러그인 경로가 등록됐는지 확인
- Claude Code 재시작

### 에이전트가 `.plan/` 폴더를 못 만듦
- 현재 작업 디렉토리 권한 확인 (`ls -la`)
- `pm-orchestrator.md`의 "아티팩트 관리" 섹션 참조

### 모순 검증 오탐이 너무 많음
- `references/consistency-rules.md`의 규칙 해석 검토
- `06_consistency-report.md`의 "오탐일 가능성" 섹션 활용
- 필요 시 규칙 임시 skip: 리포트 상단에 `# skip: C13` 주석 추가 (향후 기능)

### Frontmatter validation 실패
- 에이전트가 생성한 파일의 frontmatter YAML 문법 확인
- `artifact_id`·`phase`·`generated_by`·`depends_on`은 필수

---

## 6. 알려진 제약 (v0.1.0)

- ❌ 자동 revision 없음 (의도된 제약. 사용자 통제권 우선)
- ❌ 한국어 외 언어 미지원
- ❌ 도메인팩은 `generic.md` 하나만 (DR·SaaS·FinTech 확장 v0.2에서)
- ❌ `.plan/` 재개(resume) 기능 없음 (파이프라인 중단 시 처음부터)
- ❌ 멀티 프로젝트 동시 실행 미지원 (현재 디렉토리의 `.plan/` 단일)

---

## 7. 피드백

이슈·개선 제안:
- GitHub Issues (v1.0 출시 후)
- 개발 중: 직접 `ARCHITECTURE.md` 참조 + 에이전트 프롬프트 수정 후 `/reload-plugins`
