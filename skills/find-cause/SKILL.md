---
name: find-cause
description: Use this skill whenever the user reports a problem, error, unexpected behavior, or asks to find the cause of something. TRIGGER keywords include "디버깅", "디버그해", "원인 파악", "원인 찾아", "왜 안 돼", "왜 안 나와", "에러 나", "에러야", "버그 찾아", "버그야", "이상해", "잘못됐어", "fail", "broken", "not working", "why is this", "what's wrong". Also use when user shares an error screenshot, log snippet, or unexpected output. Do NOT trigger on generic phrases like "안 됨" or "이상하게" alone — those appear in unrelated contexts (decision statements, adverbial usage) and cause false positives. (옛 이름 `debug`는 Claude Code reserved name과 충돌해 Skill 도구 호출이 차단됐음 — issue #14864. 그래서 `find-cause`로 변경. Skill 이름 지을 때 참고.) This skill is a BLOCKING REQUIREMENT — invoke it before generating any diagnosis or fix.
---

# 디버깅

문제 보고를 받으면 추정 금지. 확실한 증거 기반으로만 원인 파악.

## 핵심 룰

### 1. 추정 절대 금지 — 증거부터

문제 원인을 추측으로 답하지 말 것. 다음 순서로 증거 확보:

**a. 로그 인프라 먼저 조회** (프로젝트의 logging 인프라 — PostgreSQL 로그 테이블, CloudWatch, ELK stack, 서비스별 로그 대시보드 등 프로젝트마다 다름).
- 사고 시각 기준 ±N분 윈도우로 관련 로그 추출
- error/warning level 우선
- 키워드(이벤트 ID, 사용자 ID 등)로 필터링
- 배포 플랫폼 CLI 로그는 retention 있으니 영구 로그 저장소 먼저

**b. 관련 DB 테이블 조회**
- conversations / events / state 테이블 직접 조회
- 사고 시점의 record가 실제 어떻게 박혔는지 확인

**c. 코드 분석**
- 로그/DB로 가설 좁힌 후 관련 코드 path를 read + grep
- "이렇게 동작했을 것"이 아니라 "이 코드가 어떻게 동작하는지" 확정

### 2. 정확한 원인 못 찾으면 — 추가 진단 제안

로그가 부족해서 원인 못 좁히면 **절대 추정으로 fix 제시 금지**. 대신:
- 필요한 추가 로그 위치 명시
- 진단 코드 추가 plan 제시 (예: "이 함수에 entry/exit log 박으면 다음 재현 시 정확히 잡힘")
- 사용자에게 진단 강화 → 재현 → 분석 순서 안내

### 3. 원인 확정 후 — plan-create로 전환

정확한 원인이 잡히면:
- 그 원인을 fix하는 plan을 작성
- `plan-create` skill 룰 따라 plan 작성
- 사용자 검증 + 승인 후 implement

즉 디버깅 결과 = "원인 + fix plan" 두 부분.

## 보고 형식

사용자에게 답할 때 풀어 쓴 단락으로:
1. **확정된 사실** (로그/DB 인용)
2. **원인 분석** (어떤 코드가 어떻게 동작해서 문제 발생)
3. **fix 방향** (단순하면 바로 plan-create로 전환, 복잡하면 사용자 결정 필요 항목 인터뷰)

확신도 명시 — 100% 확정인지, 90% 정도 의심인지 솔직히 보고.

## 사용자 영향 즉시 차단 안내 (선행 액션)

문제가 사용자에게 손해를 누적시키고 있는 상황(외부 액션 폭주, 비용 발생, 데이터
손실 위험 등)이면 진단 시작 전에 **사용자가 즉시 손해를 차단할 수 있는 manual
경로를 답변 첫 부분에 안내**한다. 예를 들면 외부 서비스의 직접 정지 버튼,
컨트롤러 차단, 앱 강제 종료 등 우회 액션. 진단 끝날 때까지 손해가 계속되는
케이스를 차단하는 안전망. 영향 없는 단순 버그 진단에는 적용 안 함.

## 추가 진단 로그 박는 plan

로그가 부족해서 원인을 못 좁히면 **절대 추정으로 fix 제시 금지**. 대신 다음
재현 시 정확히 잡힐 수 있게 로그를 어디에 어떤 형태로 박을지 plan을 제시한다.
함수 entry/exit 로그, 핵심 변수 dump, 시점 timestamp 같은 구체적 진단 코드를
제안하고, plan-create skill 흐름으로 넘어가 그 plan을 사용자가 승인 후 구현. 추측
fix로 코드 수정하면 시간 낭비뿐 아니라 진짜 원인은 못 찾아 같은 사고가 재발한다.

## 프로젝트별 logging 인프라 활용

각 프로젝트마다 logging 인프라가 다르다. 디버깅 시 해당 프로젝트의 `CLAUDE.md`
에서 logging 인프라 설명을 먼저 확인하고 그 흐름으로 진단한다. 예를 들면 서버
로그가 PostgreSQL 테이블에 영구 저장되는 프로젝트, CloudWatch에 저장되는
프로젝트, 로컬 파일에 저장되는 프로젝트 등 다양하다. 배포 플랫폼 CLI 로그는
retention 이 있으므로 영구 로그 저장소를 먼저 확인하는 게 정석.

## 절대 안 되는 것

- "아마 이런 거 같아요" / "이게 원인일 거예요" — 증거 없이 추정
- 로그 안 보고 코드만 보고 진단
- 사용자가 "왜 안 돼?"라고 짧게 물었을 때 즉시 추정 답변 (반드시 증거부터 수집)
- 원인 미확정인데 fix 제시
- 추측 원인으로 코드 수정 (수정해도 진짜 원인 아닐 수 있음)
- "destructive 조치로 문제 회피" (예: --no-verify, force push, 데이터 삭제 등 — 진짜 원인 fix 아님)
