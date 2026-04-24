# 공식 마켓플레이스 제출 가이드

> **제출 폼**: [clau.de/plugin-directory-submission](https://clau.de/plugin-directory-submission)
> **플러그인**: bini-zzang v0.1.0
> **레포**: https://github.com/ansolbini-oss/bini-zzang
> **작성일**: 2026-04-24

이 문서는 Anthropic 공식 플러그인 디렉토리 제출 폼에 그대로 붙여넣기 위한 자료 모음입니다.

---

## 폼 필드 예상 답변

> ⚠️ 실제 폼 필드 구성은 접속해서 확인 필요. 아래는 표준 플러그인 디렉토리 제출 기준 통상 요구 항목.

### 1. Plugin Name
```
bini-zzang
```

### 2. Display Name (한글 브랜드)
```
bini쨩
```

### 3. Short Description (80자 이내, 검색 노출용)
```
13명 전문가 에이전트가 완결성·정합성을 보장하는 기획 풀 파이프라인. PM·IA·기능명세·정책·KPI·리스크까지.
```

### 4. Long Description (300~500자, 플러그인 페이지 본문)
```
한국 기획자를 위한 Claude Code 플러그인. 혼자 하기엔 양이 너무 많은 기획 작업을, 13명의 전문가 에이전트 팀이 의존-순차 파이프라인으로 대신 진행합니다.

6단계 파이프라인 (Phase 0~5):
- Phase 0: 용어집 구축 (단일 진실원)
- Phase 1: 시장조사 + 전략·포지셔닝
- Phase 2: 페르소나 + 플로우 + IA
- Phase 3: 기능명세(15섹션) + 데이터 모델
- Phase 4: 정책 + KPI + 리스크 레지스터
- Phase 5: 전수 정합성 교차검증

차별화 포인트:
1. 아티팩트 우선 — 리포트 한 장이 아니라 실사용 파일 15종 생성
2. 정합성 체커 에이전트가 14개 규칙으로 모든 산출물 교차검증
3. 모순 발견 시 자동 수정 없이 사용자 체크 게이트 (선택지 3개 제시)
4. ADR 형식 의사결정 기록
5. ID 동결 append-only로 재실행 안전

결과물은 .plan/ 폴더에 PRD·IA 트리·기능명세·데이터모델 등 실제 개발 착수 가능한 아티팩트로 저장됩니다.
```

### 5. Category
```
productivity
```
(후보: `productivity`, `development` 중 선택 — bini-zzang은 제품 기획자용이므로 **productivity**가 적합)

### 6. Keywords / Tags
```
planning, product-management, 기획, PRD, IA, feature-spec, korean
```

### 7. Homepage / Repository URL
```
https://github.com/ansolbini-oss/bini-zzang
```

### 8. Author
```
Name: bini
Email: gf_ai@60hz.io
Organization: 60hz.io
```

### 9. License
```
MIT
```

### 10. Installation Command (사용자용 검증 안내)
```
/plugin marketplace add ansolbini-oss/bini-zzang
/plugin install bini-zzang@bini-zzang
/reload-plugins
```

### 11. Supported Languages
```
Korean (primary)
```

### 12. Required Claude Code Version
```
>= 1.0.0
```

### 13. Privacy / Data Handling
```
- 플러그인은 사용자의 현재 작업 디렉토리 아래 .plan/ 폴더에만 파일을 생성합니다.
- 외부 API 호출 없음 (서브에이전트는 Claude 표준 프롬프트로 동작).
- 개인정보·자격증명 수집 없음.
- 사용자가 입력한 기획 주제는 Claude에 전송되는 일반 프롬프트 외에는 보관되지 않습니다.
```

### 14. Known Limitations
```
- 한국어 전용 (다국어 지원은 v0.2 로드맵)
- 도메인팩은 generic.md 하나만 제공 (DR·SaaS·FinTech 등 특화팩 v0.2 예정)
- .plan/ 재개(resume) 기능 없음 (파이프라인 중단 시 처음부터 재실행 필요)
- 자동 revision 없음 (의도된 제약 — 사용자 통제권 우선)
```

### 15. Testing Evidence
```
- SMOKE_TEST_REPORT.md에 스모크 테스트 결과 기록
- Phase 0 + Phase 1-a + Phase 5 실제 실행 검증 완료
- 13명 에이전트 전원의 자기검증 체크리스트 통과
- 124K 토큰 / 8.5분 소요 (부분 스모크 기준)
- 풀 파이프라인 추정: ~500K tokens / ~40분
```

### 16. Screenshot / Demo (선택)
- 현재 없음 (텍스트 기반 파이프라인이라 스크린샷보다 산출물 예시가 유효)
- SMOKE_TEST_REPORT.md에 생성된 아티팩트 구조 예시 포함

---

## 제출 전 최종 체크리스트

- [x] GitHub 레포 Public: [github.com/ansolbini-oss/bini-zzang](https://github.com/ansolbini-oss/bini-zzang)
- [x] LICENSE 파일 존재 (MIT)
- [x] README.md 명확 (설치·사용·제거 모두 기술)
- [x] `.claude-plugin/plugin.json` 유효 JSON
- [x] `.claude-plugin/marketplace.json` 유효 JSON
- [x] 13개 agents/*.md 전원 작동 확인
- [x] 5개 commands/*.md 전원 등록 확인
- [x] 악성코드·자격증명 수집 없음
- [x] 외부 네트워크 호출 없음 (Claude 표준 외)
- [x] 로컬 설치 테스트 통과 (`/plugin marketplace add` → `install` → `reload-plugins` 3 plugins / 10 skills / 31 agents 확인)
- [x] 스모크 테스트 실행·리포트 작성 완료
- [ ] (선택) 영문 README 섹션 추가 — 국제 유저 발견성 향상
- [ ] (선택) 데모 GIF·비디오 — 마켓플레이스 페이지 매력도 향상

---

## 제출 절차

1. **[clau.de/plugin-directory-submission](https://clau.de/plugin-directory-submission)** 접속
2. 위 "폼 필드 예상 답변"에서 필요한 내용 복사해서 해당 필드에 붙여넣기
3. 폼이 요구하는 추가 항목이 있으면 이 문서 업데이트 후 제출
4. 제출 후 Anthropic 리뷰 대기 (보통 수일~수주)
5. 리뷰 통과 시 `anthropics/claude-plugins-official` 레포의 marketplace.json에 자동 추가됨
6. 전체 Claude Code 유저의 `/plugin browse` 또는 `/plugin > Discover`에서 검색 가능해짐

---

## 리뷰 피드백 대응 팁

- **"설명이 불명확하다"** → 한글+영문 병기 README 섹션 추가
- **"테스트 증거 부족"** → SMOKE_TEST_REPORT.md 링크 강조
- **"카테고리 부적합"** → development vs productivity 재선택
- **"홈페이지에 추가 정보 부족"** → README 상단에 스크린샷·GIF 추가
- **"보안 우려"** → plugin.json·agents/*.md에 외부 호출 없음을 명시

---

## 이후 업데이트 방법 (승인 후)

공식 마켓에 등록된 후 v0.2.0 배포 시:
1. GitHub 레포에 태그 생성 (`git tag v0.2.0 && git push --tags`)
2. Anthropic은 주기적으로 연결된 레포에서 최신 커밋 SHA를 sync (자동) 또는 PR 기반 업데이트
3. 큰 변경은 changelog를 명확히 업데이트
