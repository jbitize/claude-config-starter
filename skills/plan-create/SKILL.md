---
name: plan-create
description: Use this skill whenever the user asks to write a plan, design, or specification — especially in Korean. TRIGGER keywords include "plan 작성", "plan 만들어", "기획해", "기획 작성", "계획해", "계획 작성", "설계해", "스펙 작성", "spec 만들어", "design", "create a plan", "draft a plan". Also use when the user describes a new feature or change that requires non-trivial implementation work, even without explicit trigger words. This skill is a BLOCKING REQUIREMENT — invoke it before generating any plan content.
---

# Plan 작성

사용자가 새 기능, 변경, 통합 등을 요청하면 바로 코드 쓰지 말고 plan부터 작성한다. 이 skill의 룰을 매번 그대로 따른다.

## 핵심 룰

### 1. 관련 코드와 로직을 100% 완벽히 파악

plan 작성 전에 영향받을 코드와 흐름을 끝까지 파악한다.
- 기존 코드 path를 grep + read로 모두 확인
- 의존성, 호출 관계, side effect를 명확히 추적
- "확실히 모르겠다"고 느끼는 부분이 있으면 plan 작성 전에 추가 조사 — 절대 추정으로 plan 채우지 말 것

### 2. 베스트 plan을 md 파일로 작성

plan은 채팅 응답이 아니라 **`docs/plans/plan_<feature>.md`** 파일로 작성한다(또는 프로젝트별 plan 디렉토리 위치).

작성 내용:
- 변경 범위(파일/모듈/엔드포인트 명확히)
- 단계별 작업 순서
- 영향받는 기존 동작과 backward compatibility 검토
- 테스트 plan(pytest, 디바이스 실측, 회귀 검증 단계)
- 위험 / 엣지케이스와 대응
- self-review 섹션(베스트인지/빠진 거/오버한 거)
- 사용자 결정 필요 항목

### 3. UI/UX 설계 (필요 시)

UI/UX가 포함된 작업이면 베스트 UX 흐름도 같이 설계.
- 사용자 flow를 step별로 정리
- 화면 상태 변화, 토스트/에러 메시지 문구
- 기존 디자인 시스템과 일관성

### 4. 인터뷰 (사용자 결정 필요 시)

plan 작성 중에 사용자가 결정해야 하는 사항이 나오면 **인터뷰 룰**을 따른다.

#### 4-1. 질문 list 작성 직후 자문 단계 (필수)

질문 list를 만들고 사용자에게 보내기 **직전에**, 각 항목마다 다음 자문을 의무로
실행한다.

> "이 항목은 사용자의 UX/실측/취향/도메인 결정인가? 아니면 시스템 아키텍처와
> 코드를 알고 있는 내가 그냥 베스트 판단으로 결정 가능한 항목인가?"

후자라면 그 항목은 **인터뷰에서 빼고**, 내가 결정한 결과를 plan 본문에 박는다
(결정 근거 한 줄과 함께). 인터뷰에 남기는 건 진짜 사용자 영역 항목만이다.

자문을 건너뛰면 "추천이 명확한데도 객관식 5개로 던지는" 안티패턴이 발생한다.
"네가 잘 결정할 수 있는 것은 절대 물어보지 말고 네가 알아서 결정해라" 원칙을
재강조 — 룰 자체는 4-2 마지막 줄에도 있었지만 자문 단계가 없어서 실제 작성 중에
빠지는 사고가 반복됐다. 그 사고 방지가 4-1의 본질.

사용자 영역 vs 내 영역 가르는 기준:
- **사용자 영역** — UX 선호 (CLI vs 웹 vs iOS), 실측이 필요한 사용성, 도메인 지식
  이 필요한 비즈니스 로직, 사실 확인이 필요한 매핑, trade-off가 동등한 양자택일에서
  사용자 취향
- **내 영역** — 책임 경계 어디에 둘지, payload 필드 명명, 데이터 구조, 마이그
  레이션 순서, 백워드 호환성 처리, 코드 위치/모듈 분리, 테스트 전략, 추천이
  명확한 아키텍처 결정

#### 4-2. 인터뷰 포맷

자문 통과한 항목만 다음 포맷으로 던진다.

- 질문은 뻔하지 않고 심도 있게
- 객관식 4~10개 선택지 + 추천(1번 배치)
- 전체 질문 수를 처음에 알려준다
- 하나씩 묻고 답 받은 후 다음 질문
- 각 선택지는 축약 금지 — 1~3문장으로 풀어 설명 + trade-off 명시
- 사용자 결정 불필요한 사항(내가 결정 가능)은 묻지 말고 결정 (4-1 자문으로 이미
  걸러진 상태여야 함)

### 5. 외부 API/서비스 통합 시 공식 문서 직접 확인

외부 시스템 통합(외부 API, 결제, 인증, DB 등)을 plan에 포함할 때는 추정으로
setup 안내를 박지 말고 **공식 문서를 직접 확인**한 후 plan을 채운다. WebFetch나
WebSearch로 endpoint, 인증 방식, rate limit, 파라미터 spec 같은 핵심 정보를 잡고
그 결과를 plan 본문에 인용. anti-pattern은 "이 메뉴에 있을 것"이나 "보통 이 형식"
같은 추정 안내 — 사용자가 그대로 따라가다 막히면 사고가 발생한다.

### 6. 메이저 기능은 spec 문서로 분리 보존

작은 변경은 `plan.md`(또는 `plan_<feature>.md`)로 작성 후 구현 완료 시 삭제.
메이저 기능(여러 컴포넌트 걸친 통합, 큰 아키텍처 결정 등)은 `spec_<feature>.md`
형식으로 영구 보존한다. plan은 작업 진행용 임시 문서, spec은 미래 참고용
reference 문서로 역할이 다르다. plan 작성 시점에 어느 카테고리인지 판단해서
파일명과 보존 정책을 다르게 가져간다.

### 7. plan 파일은 구현 코드와 같은 commit에 push

plan/spec 문서는 구현 코드와 **같은 commit에 묶어** push한다. plan 따로 commit
하고 코드 따로 commit하면 history에서 plan과 코드 매칭이 어려워져 미래 리뷰
부담이 커진다. plan 파일 삭제는 follow-up commit으로 별도 처리해서 plan history
자체는 git log에 남게 한다.

### 8. Plan 작성 완료 후

작성이 끝나면 `plan-verify` skill을 호출하라고 사용자에게 안내하거나, 사용자가 "검증해" / "체크해" 등 trigger 표현 시 즉시 plan-verify skill로 전환한다.

## 절대 안 되는 것

- plan 없이 바로 코드 작성
- 추정으로 plan 채우기 (관련 코드 안 보고)
- 사용자가 결정해야 할 사항을 plan에 그대로 박기 (인터뷰로 분리)
- 짧은 라벨 bullet만 나열 (글로벌 CLAUDE.md 룰 — 풀어 쓴 완성 문장)
