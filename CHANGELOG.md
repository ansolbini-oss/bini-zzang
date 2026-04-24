# Changelog

모든 주요 변경사항은 이 파일에 기록된다. 포맷은 [Keep a Changelog](https://keepachangelog.com/) 따름.

## [0.1.0] — 2026-04-24

### 초기 출시 (MVP)

#### 에이전트 13인 팀 구성
- `pm-orchestrator`: 최상위 PM. Phase 0~5 관장, 게이트 운영, 사용자 체크 담당.
- `glossary-writer`: Phase 0 용어집 SSOT 작성.
- `market-researcher`: Phase 1-a 시장·페인·경쟁 분석 + `_constants.json` 작성.
- `strategist`: Phase 1-b 포지셔닝 3안·가치제안·Non-Goals·시나리오.
- `user-researcher`: Phase 2-a 페르소나 (JTBD 기반).
- `flow-designer`: Phase 2-b 유저 플로우 (Mermaid 상태 기계).
- `ia-architect`: Phase 2-c 정보구조·역할 매트릭스·네비게이션.
- `feature-writer`: Phase 3-a 기능명세 15섹션 (ADR 포함).
- `data-modeler`: Phase 3-b 통합 ERD·엔티티 정규화.
- `policy-writer`: Phase 4-a 비즈니스 룰·규제·권한 정책.
- `kpi-designer`: Phase 4-b 북극성·입력·결과·가드레일 지표.
- `risk-analyst`: Phase 4-c 리스크 레지스터·확률×영향 매트릭스.
- `consistency-checker`: Phase 5 정합성 교차검증 (C1~C14 규칙).

#### 슬래시 커맨드 5개
- `/bini-zzang:기획` — 풀 파이프라인 (Phase 0~5)
- `/bini-zzang:기획-시장조사` — Phase 1만
- `/bini-zzang:기획-ia` — Phase 2만
- `/bini-zzang:기획-기능명세` — Phase 3만
- `/bini-zzang:기획-검증` — Phase 5만 (재검증)

#### 핵심 차별화 (vs wj-magic:ideation)
- ✅ 의존-순차 파이프라인 (병렬만 아님)
- ✅ 아티팩트 우선 (리포트 우선 아님) — 15종 실사용 파일
- ✅ 정합성 체커 에이전트 (14개 규칙 전수 검증)
- ✅ 모순 발견 시 사용자 체크 게이트 (자동 수정 금지)
- ✅ ADR 형식 의사결정 (대안/이유/철회 조건)
- ✅ SSOT `_constants.json` + 용어집
- ✅ ID 동결 append-only 재실행
- ✅ 도메인팩 확장 구조

#### 문서
- `README.md` — 플러그인 소개·사용법
- `ARCHITECTURE.md` — 설계 원칙·Phase 그래프·스키마
- `TESTING.md` — 로컬 설치·스모크·풀 파이프라인 테스트
- `references/artifact-schemas.md` — 15종 아티팩트 Frontmatter 스키마
- `references/consistency-rules.md` — C1~C14 정합성 검증 규칙
- `references/phase-dependencies.md` — Phase 의존 그래프
- `references/domain-packs/generic.md` — 도메인 중립 기본값

### 알려진 제약
- 도메인팩 `generic.md` 하나만 (v0.2에서 `dr.md`, `saas.md`, `fintech.md` 예정)
- `.plan/` 재개(resume) 미지원
- 멀티 프로젝트 동시 실행 미지원
- 실제 `/plugin install` 검증 미완 (별도 세션 예정)
- wj-magic과 A/B 벤치마크 정량 결과 미확보 (v0.2에서 수행)

### 다음 계획 (v0.2.0 로드맵)
- 도메인팩 3종 추가 (dr, saas, fintech)
- wj-magic:ideation 대비 벤치마크 (skill-creator 평가 세트)
- `.plan/` resume 기능
- Phase별 오류 복구 전략 강화
