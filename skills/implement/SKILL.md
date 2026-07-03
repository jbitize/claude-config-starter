---
name: implement
description: Use this skill ONLY when the user explicitly signals they want implementation to proceed. TRIGGER keywords include "구현해", "구현 진행", "진행해", "implement", "make it", "go ahead", "그래 진행해", "ok 진행", "코딩해", "코드 작성해", "build it". Do NOT auto-invoke without explicit user signal — implementation requires user approval per plan-verify outcome. This skill is a BLOCKING REQUIREMENT once triggered — invoke before writing any code.
---

# 구현

사용자가 명시 승인한 plan을 끝까지 정확히 구현한다.

## 핵심 룰

### 1. 작성된 plan 그대로 진행

- plan-create로 만든 plan의 모든 항목을 빠짐없이 구현
- plan에 없는 추가 변경 금지 (스코프 확장 금지)
- 구현 중 plan과 다른 path가 더 좋다고 판단되면 멈추고 사용자에게 확인

### 2. 중간에 멈추지 말 것

- 사용자 개입이 필요하거나(외부 액션, env 등록 등) 다른 차단 요인이 있는 경우만 멈춤
- 코드 작성 → 테스트 update → pytest 실행 → 빌드 검증 → commit 까지 한 흐름
- "1단계 끝났습니다 진행할까요" 같은 중간 확인은 plan 안에 명시한 단계가 아니면 안 함

### 3. 완료 항목은 plan에 ✅ 표시

- plan.md 파일이 있으면 완료 항목마다 ✅ 추가
- 모든 항목 완료되면 plan.md 파일 삭제 (`docs/plans/`에 미완료만 남김)
- 단 spec 성격(영구 보존)은 별도 — spec_<feature>.md 패턴

### 4. 테스트는 필수

- 새 기능/수정에 대한 테스트 케이스 빠짐없이 추가
- `pytest tests/ -v`로 전체 회귀 통과 확인
- 통과 못 하면 절대 commit/push 금지
- 네이티브 앱은 테스트 setup 없으면 빌드 통과 + 디바이스 실측 시나리오 명시

### 5. 구현 완료 후 — code review

자체 code review 한 번:
- 변경된 파일들을 다시 훑어보면서 누락/실수 점검
- side effect, race condition, error handling 빠진 곳 확인
- 글로벌 CLAUDE.md 룰(markdown italic 금지, 풀어쓰기) 적용된 결과인지

### 6. Commit + Push 순서 엄수 + 배포

배포 흐름은 `commit-deploy-flow` skill을 호출해서 그 룰을 따른다. 핵심
원칙만 요약하면 다음과 같다.

- 순서: **commit → push → deploy.** 절대 깨지 말 것. 배포 스크립트가 git HEAD
  hash를 사용하는 경우 unstaged 변경 있으면 옛 hash가 박혀 버려서 코드/hash
  부정합이 발생한다.
- commit 메시지는 변경 의도, 영향 범위, 테스트 결과를 모두 포함.
- push 후 commit short hash를 응답 마지막에 명시. 사용자가 어떤 버전이
  어디에 깔렸는지 즉시 확인 가능하도록.
- 프로젝트별 구체적 배포 스크립트, 활성화 대기 방식, 디바이스 강제 종료
  안내 같은 디테일은 각 프로젝트의 `CLAUDE.md`를 따른다.

## 절대 안 되는 것

- plan 없이 즉시 구현 (plan-create 먼저 호출)
- 사용자 명시 승인 없이 구현 시작
- 테스트 안 돌리고 commit
- commit 안 하고 deploy
- 스코프 확장 (plan에 없는 변경 추가)
