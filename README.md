# bini쨩 (`bini-zzang`)

**기획자 1인이 감당하기엔 양이 너무 많다. 그렇다고 흩어지게 쓰면 정합성이 무너진다.**
bini쨩은 13명의 전문 기획 에이전트 팀이 **의존-순차 파이프라인**과 **정합성 검증**으로 기획 풀 파이프라인을 대신 굴려주는 Claude Code 플러그인입니다.

## 왜 또 만드는가

기존 `wj-magic:ideation` 같은 기획 지원 스킬은 **5명 에이전트를 병렬로 한 번 돌리고 끝**입니다. 그래서:

- 에이전트들끼리 서로 안 보여서 **용어·수치·가정이 불일치**
- **1회성 병렬** — 모순 발견해도 수정 루프가 없음
- 결과물이 **리포트 한 장** — PRD·IA 트리·용어집 같은 **실사용 아티팩트** 부재
- Best Option 추천은 있지만 **근거 체인(ADR)** 얕음

bini쨩의 차별화 4축:

1. **의존-순차 파이프라인**: 시장조사 결과를 알고 있어야 전략을 쓸 수 있음. Phase 간 의존성을 강제한다.
2. **아티팩트 우선**: 리포트가 아니라 `glossary.md`, `personas.md`, `ia-tree.md`, `features/*.md` 같은 **실사용 파일들**을 생성한다.
3. **정합성 체커 에이전트**: 13번째 전담 에이전트가 모든 산출물을 교차 검증한다. 모순 발견 시 **사용자에게 보여주고 확인받은 뒤**에만 재개.
4. **ADR 형식 의사결정**: 대안 / 트레이드오프 / 결정 / 이유 / 철회 조건까지 명시.

## 설치

Claude Code 입력창에서 아래 3줄을 순서대로 실행:

```
/plugin marketplace add ansolbini-oss/bini-zzang
/plugin install bini-zzang@bini-zzang
/reload-plugins
```

> **Note**: `ansolbini-oss/bini-zzang`은 GitHub `<owner>/<repo>` 형식. 본인이 fork/clone한 경우 그에 맞춰 바꿔주세요.

성공하면 `/help` 실행 시 `/bini-zzang:기획` 등 5개 커맨드와 13명 에이전트가 노출됩니다.

### 로컬 개발용 설치

플러그인을 직접 수정하며 개발할 때:

```bash
# 1) 레포 clone
git clone https://github.com/ansolbini-oss/bini-zzang.git ~/bini-zzang

# 2) Claude Code에서
/plugin marketplace add ~/bini-zzang
/plugin install bini-zzang@bini-zzang
/reload-plugins
```

`~/bini-zzang` 내 파일을 수정한 뒤 `/reload-plugins`만 다시 실행하면 즉시 반영됩니다.

### 제거

```
/plugin uninstall bini-zzang@bini-zzang
/plugin marketplace remove bini-zzang
/reload-plugins
```

## 사용법

```bash
# 풀 파이프라인 — PM 오케스트레이터가 Phase 0~5까지 주도
/bini-zzang:기획 수요반응 DR 운영 플랫폼

# 단일 에이전트 실행
/bini-zzang:기획-시장조사
/bini-zzang:기획-ia
/bini-zzang:기획-기능명세

# 기존 산출물 정합성만 재검증
/bini-zzang:기획-검증
```

실행 결과는 현재 작업 디렉토리 아래 `.plan/` 폴더에 저장됩니다.

```
.plan/
├── 00_glossary.md
├── 01_market.md
├── 02_strategy.md
├── 03_personas.md
├── 03_flows/FL-*.md
├── 03_ia.md
├── 04_features/F-*.md
├── 04_data-model.md
├── 05_policies.md
├── 05_kpis.md
├── 05_risks.md
├── 06_consistency-report.md
└── index.json              # 전체 진행도 + 의존 관계 메타
```

## 13명 에이전트

| # | 에이전트 | Phase | 역할 |
|---|----------|-------|------|
| 1 | `pm-orchestrator` | 전체 | 게이트·Phase 관리·사용자 소통·합성 |
| 2 | `glossary-writer` | 0 | 프로젝트 용어집 SSOT 구축 |
| 3 | `market-researcher` | 1 | 시장·경쟁사·페인 분석 |
| 4 | `strategist` | 1 | 포지셔닝·가치제안·Non-Goals |
| 5 | `user-researcher` | 2 | 페르소나·JTBD |
| 6 | `flow-designer` | 2 | 유저 시나리오·플로우 |
| 7 | `ia-architect` | 2 | 정보구조·네비게이션·권한 매트릭스 |
| 8 | `feature-writer` | 3 | 기능 상세명세(인수기준) |
| 9 | `data-modeler` | 3 | 데이터 모델·ERD |
| 10 | `policy-writer` | 4 | 비즈니스 룰·규제·정책 |
| 11 | `kpi-designer` | 4 | 북극성·선행지표·측정 방법 |
| 12 | `risk-analyst` | 4 | 리스크 레지스터·엣지케이스 |
| 13 | `consistency-checker` | 5 | 전수 교차검증, 모순 발견→사용자 체크 |

자세한 아키텍처는 [`ARCHITECTURE.md`](./ARCHITECTURE.md) 참조.

## 라이선스

MIT
