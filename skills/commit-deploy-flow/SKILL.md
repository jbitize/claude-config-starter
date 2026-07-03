---
name: commit-deploy-flow
description: Use this skill whenever the user signals they want to commit, push, or deploy code changes. TRIGGER keywords include "commit해", "push해", "push까지", "올려", "올려줘", "deploy해", "deploy까지", "배포해", "배포까지", "release해", "release", "ship it", "코드 올려". Also auto-invoke at the end of an implement skill flow before any deploy step. This skill is a BLOCKING REQUIREMENT — invoke it before running any commit/push/deploy command, to enforce the correct order and avoid hash/binary mismatch incidents.
---

# Commit + Push + Deploy 흐름

사용자가 commit, push, deploy 중 하나라도 지시하면 이 skill을 호출하고 아래 룰을
순서대로 적용한다. 절대 어기지 말 것 — 이 순서가 깨지면 git hash와 실제 배포된
binary가 mismatch되는 사고가 발생한다.

## 절대 순서 — commit → push → deploy

다음 순서를 깨면 안 된다.

1. **commit.** 모든 관련 변경을 staged 후 commit. unstaged 변경이 남아있는
   상태로 다음 단계 진입 금지. commit 메시지는 변경 의도, 영향받는 영역, 테스트
   결과를 모두 포함하고, 가능하면 commit body에 plan 또는 사고 참조 명시.
2. **push.** origin remote로 push. 인증 실패하면 즉시 사용자에게 보고하고
   사용자 직접 push 안내 (`!cd <repo> && git push origin main` 같은
   세션 내 실행 패턴). push 성공 응답에서 commit short hash를 추출.
3. **deploy.** 프로젝트별 배포 스크립트나 흐름을 호출한다. 자동 배포(예:
   Railway, Vercel, Render)는 push만으로 트리거되니까 별도 액션 불필요. 디바이스
   배포(iOS, Mac, Watch 등 네이티브 앱)는 별도 스크립트 호출 필요. 프로젝트별
   디테일은 그 프로젝트의 `CLAUDE.md`를 따른다.

## 왜 이 순서가 중요한가

배포 스크립트가 git HEAD의 short hash를 코드(예: 빌드 스크립트가 hash 를 inject
하는 버전 파일)에 inject하는 경우가 많다. 만약 working tree에 unstaged 변경이
남아있는 상태에서 deploy 스크립트를 실행하면 그 변경은 HEAD hash에 반영되지 않고
옛 hash가 binary에 박혀버린다. 결과적으로 앱은 새 코드로 동작하는데 표시되는
hash는 옛 commit을 가리키는 부정합이 발생해서 디버깅이 무력화된다. 실제 사고
경험에서 잡힌 절대 룰.

## Push 후 hash 보고

push 성공 후 응답 마지막에 commit short hash를 명시한다. 사용자가 어떤 버전이
어디에 깔렸는지 즉시 확인 가능하도록. 프로젝트마다 hash 표시 위치가 다르고
(More 탭, 사이드바, ... 메뉴 등) 그 위치들도 같이 안내. 사용자가 디바이스에서
그 hash를 확인하면 배포 완료 검증 가능.

## 인증 실패 대응

git push가 인증 실패(`fatal: could not read Username` 등)로 막히면:

- credential helper 확인 (`git config --get credential.helper`)
- SSH 키 설정 또는 personal access token 확인 안내
- 즉시 fix 어려우면 사용자에게 세션 내 직접 실행 안내 — `!cd <repo> && git push origin main`
  형식으로 프롬프트에 입력하시면 같은 shell에서 실행됨

## 외부 통신 채널 push 누락 방지

만약 다른 프로젝트나 외부 시스템에 메시지/파일을 append했다면 그 repo도 commit +
push까지 끝내야 한다. 한 작업이 여러 repo에 걸쳐있는 경우 모든 관련 repo의 push
상태를 확인하고 응답에 보고. 응답 박스에 파일 내용만 박고 외부 repo push 안 한
사고가 과거 발생한 적 있다.

## Deploy 후 활성화 대기

자동 배포 플랫폼(Railway, Vercel, Render 등)은 push 후 새 컨테이너 빌드/활성화에
시간이 걸린다. 그 사이 검증 시도하면 옛 코드가 응답해서 "fix 안 됐다"고
오해하는 사고가 발생한다. 프로젝트별 활성화 확인 방법(예: 서버 로그 DB의
startup 로그 timestamp 확인)을 그 프로젝트의 `CLAUDE.md`에서 참고해서 안내.
사용자가 기다리는 동안 `ScheduleWakeup`으로 자동 polling 도 활용 가능.

## 절대 안 되는 것

- unstaged 변경 있는 상태에서 deploy 실행
- push 안 하고 deploy 시도
- push 후 hash 응답에 명시 안 함
- 외부 repo 변경 commit/push 누락
- 배포 활성화 대기 안 하고 즉시 검증 시도
- 사용자 명시 승인 없이 push (단 implement skill 흐름의 자연스러운 마무리는 예외)
