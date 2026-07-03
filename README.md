# claude-config-starter

Claude Code 를 여러 프로젝트에서 안정적으로 쓰기 위해 시행착오 끝에 정착시킨 룰의 일반화 버전입니다. user-level `CLAUDE.md` + 5개 핵심 skill (기획, 검증, 구현, 배포, 디버깅) 을 담고 있고, 리서치 프로젝트용 `research-template/CLAUDE.md` 도 같이 들어있습니다.

## 이건 시작점입니다

본인 취향대로 자유롭게 수정하세요. CLAUDE.md 의 톤 (존댓말 / 반말), skill 의 트리거 단어, 워크플로우 룰 모두 본인 환경에 맞게 바꿔도 됩니다. 이건 시작점이지 정답이 아닙니다.

## 라이센스

MIT License. 자유 재사용·재배포·수정 가능. 자세한 사항은 [LICENSE](./LICENSE) 파일 참조.

## Setup on a new machine

Claude Code 시작 가이드 3 본문 챕터 4 의 프롬프트를 그대로 Claude 에게 던지면 자동 처리됩니다:

```
github.com/jbitize/claude-config-starter 를 내 GitHub 계정으로
fork 한 후 그 fork 한 곳을 ~/Code/claude-config-starter 로
clone 해줘. 내 사적 정보가 들어갈 가능성이 있으니 fork 는
private 으로 만들어줘. 그 다음 안의 CLAUDE.md 와 skills/ 폴더를
~/.claude/ 에 symlink 로 연결해줘.
```

Claude 가 `gh repo fork --clone=false`, private 변경, git clone, symlink, 기존 파일 backup 까지 자동 처리합니다. PC (Windows) 는 symlink 대신 PowerShell 의 `New-Item -ItemType SymbolicLink` 또는 폴더 복사로 처리 — Claude 가 OS 감지 후 분기.

⚠️ **중요**: fork 는 반드시 private 으로 만들어야 합니다. 본인이 CLAUDE.md 에 사적 메모리 (친구 비밀, 학교 자료 등) 를 추가한 후 public 으로 두면 인터넷에 공개됩니다.

## Daily sync workflow

- 본인 수정 사항 저장 — `git commit + git push` (본인 private fork 에)
- upstream 저장소 업데이트 받기 — `git pull upstream main`. Conflict 발생 시 Claude 에 "conflict 해결해줘" 한 마디.

## Skill 5개 트리거

| Skill | 주요 트리거 단어 |
|---|---|
| `plan-create` | "기획해", "계획해", "plan 작성", "설계해" |
| `plan-verify` | "검증해", "체크해", "빠진 거 없어" |
| `implement` | "구현해", "진행해", "그래 진행" |
| `commit-deploy-flow` | "commit해", "push해", "올려", "배포해" |
| `find-cause` | "왜 안 돼", "이상해", "원인 찾아", "에러 나" |

각 skill 의 상세 룰은 `skills/<name>/SKILL.md` 안에 있습니다. 각 SKILL.md 는 Claude 가 auto-invocation 판단에 쓰는 YAML frontmatter (name, description 필수) 와 skill 본문 markdown 룰로 구성됩니다.

## research-template/ 폴더

리서치 프로젝트용 project-level CLAUDE.md 템플릿. 리서치 작업 폴더에 복사해서 쓰세요. 리서치 배포 워크플로우 (Render 정적 사이트 재사용), 민감 자료 폴더 gitignore 규칙, plan-first workflow 룰 등이 포함돼 있습니다.

## 마무리

이 룰들이 본인의 Claude Code 경험을 더 안정적으로 만들어줄 거예요. 즐기시길.
