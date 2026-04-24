---
description: Phase 2만 단독 실행 — 페르소나·플로우·IA. Phase 1 산출물이 필요.
---

# `/bini-zzang:기획-ia` — Phase 2 단독 실행

Phase 2(페르소나 + 플로우 + IA)만 수행한다.

## 사전 조건

- `.plan/01_market.md` 존재 (Pre-gate에서 검증)
- `.plan/02_strategy.md` 존재

하나라도 없으면:
```
❌ Phase 2 실행 불가: Phase 1 산출물 없음
→ 먼저 /bini-zzang:기획-시장조사 를 실행하세요
```

## 실행 단계

1. `user-researcher` 호출 (순차 1)
   - 입력: 01, 02, 00_glossary
   - 산출물: `.plan/03_personas.md`
   - Mid-gate: 페르소나 검토

2. `flow-designer` + `ia-architect` **병렬 호출**
   - 입력: 00, 02, 03_personas
   - 산출물:
     - `.plan/03_flows/FL-*.md`
     - `.plan/03_ia.md`
     - `.plan/03_role-matrix.csv`

3. Post-gate: L1→L2 트리 + 시나리오 매핑 검토

## 완료 후 권장

```
/bini-zzang:기획-기능명세    # Phase 3 실행
/bini-zzang:기획-검증       # 지금까지 산출물 정합성 검증
```
