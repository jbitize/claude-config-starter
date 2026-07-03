---
name: plan-verify
description: Use this skill whenever the user asks to verify, check, review, or validate a plan. TRIGGER keywords must reference "plan" explicitly to avoid confusion with other verification contexts — "plan 검증", "plan 확인", "plan 검토", "plan 체크해", "베스트 plan 인지", "빠진 거 없어", "오버한 거 없어", "테스트 충분해", "review plan", "is this best", "anything missing". Do NOT trigger on generic verification words like "체크해", "검토해", "완벽해", "확실해" without "plan" context — those appear too often in unrelated conversations. Also auto-invoke right after plan-create finishes. This skill is a BLOCKING REQUIREMENT — invoke it before answering any verification question.
---

# Plan 검증

작성된 plan(또는 방금 제시한 plan)을 critically 검토한다. 사용자가 짧게 "확실해?" 또는 "베스트야?" 한 마디만 던져도 이 skill을 호출하고 아래 룰대로 점검한다.

## 4가지 점검 축

### 1. 베스트인가

- 다른 옵션과 비교했을 때 진짜 최선인지
- 더 단순하거나 더 견고한 대안이 있는지
- 코드 변경 최소화 + 사용자 의도 일치 두 축 모두 만족하는지

### 2. 빠진 거 없는지

- 영향받을 코드 경로 누락 없는지 (grep으로 재확인)
- 환경변수, 외부 의존성, backward compatibility 처리
- 에러 케이스, 엣지 케이스
- SSE / async / state sync 같은 흐름의 race 가능성
- 운영 데이터로 검증 가능한 케이스

### 3. 오버한 거 없는지

- over-engineering 요소 (불필요한 abstraction, premature optimization)
- 작업 scope이 사용자 요청보다 넓어진 부분
- 결정 안 된 future scope이 plan에 섞여있는지
- 안 써도 되는 추가 검증/로깅/wrap

### 4. 테스트 충분한지

- 핵심 path + 엣지케이스 + regression 다 커버하는지
- mock 기반 한계 (실제 통합 검증 필요 여부 명시)
- 네이티브 앱 테스트 setup 없으면 디바이스 실측 시나리오 명시
- 새 기능 후 회귀 영향받는 기존 test 다 통과 확인

## 작성 형식

사용자에게 답할 때 — 위 4축마다 명시적 결과 보고. "베스트인가/빠진 거/오버한 거/테스트 충분한가" 4섹션으로 정리. 풀어 쓴 단락으로(라벨 bullet 금지).

## 확신도 백분율 명시 (필수)

검증 결과 끝에 종합 확신도를 % 단위로 표현한다. 90%, 95%, 80% 같이 솔직한
수치를 박고 100% 라고 자신 못 하는 부분이 있으면 어떤 항목이 모호한지 함께
명시한다. 예시 — "90% 자신. 외부 API rate limit이 device 단위인지 endpoint
전체인지 docs에서 100% 명확하지 않아 보수적으로 endpoint 전체로 가정한 부분이
남아있음." 무근거 자신감("완벽합니다", "확실해요")은 anti-pattern이라 절대
사용 안 한다. 사용자가 의사결정 내릴 때 어디까지 신뢰 가능한지가 확신도
수치로 즉시 보여야 한다.

## 검증 결과 후

- 정정 필요한 부분 있으면 plan 업데이트 후 다시 검증
- 정정 없으면 사용자에게 "이대로 진행할까요?" 명시적 승인 요청
- 사용자 명시 승인("그래", "진행해" 등) 받기 전에 구현 시작 금지

## 절대 안 되는 것

- "완벽합니다" 같은 무근거 자신감
- 검증 없이 "괜찮을 거예요" 응답
- 4축 중 하나라도 누락 (특히 "오버한 거" 자주 누락 — 의식적 점검)
