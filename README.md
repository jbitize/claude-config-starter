# claude-config-starter

Claude Code 를 여러 프로젝트에서 안정적으로 쓰기 위해 시행착오 끝에 정착시킨 룰의 일반화 버전입니다. user-level `CLAUDE.md` + 5개 핵심 skill (기획, 검증, 구현, 배포, 문제 해결) 을 담고 있고, 리서치 프로젝트용 `research-template/CLAUDE.md` 도 같이 들어있습니다.

## 이건 시작점입니다

본인 취향대로 자유롭게 수정하세요. CLAUDE.md 의 톤 (존댓말 / 반말), skill 의 트리거 단어, 워크플로우 룰 모두 본인 환경에 맞게 바꿔도 됩니다. 이건 시작점이지 정답이 아닙니다.

## 라이센스

MIT License. 자유 재사용·재배포·수정 가능. 자세한 사항은 [LICENSE](./LICENSE) 파일 참조.

## Setup on a new machine

Claude Code 시작 가이드 3 본문 챕터 4 의 프롬프트를 그대로 Claude 에게 던지면 자동 처리됩니다:

```
github.com/jbitize/claude-config-starter 저장소에 있는
CLAUDE.md 파일과 skills/ 폴더 안 파일들을 내 컴퓨터의
~/.claude/ 위치로 복사해줘. 기존 ~/.claude/CLAUDE.md 랑
skills/ 폴더가 있으면 backup 해두고 진행. 다 되면 VS Code
를 재시작하는 방법도 알려줘.
```

Claude 가 GitHub 저장소에서 파일 다운로드 → `~/.claude/` 에 복사 → 기존 파일 backup → VS Code 재시작 안내까지 자동 처리합니다.

PC (Windows) 환경도 파일 복사만 하니 그대로 진행 가능. 관리자 권한 필요 없어요.

## 업데이트가 필요하면

이 저장소가 나중에 개선되면 같은 프롬프트를 한 번 더 던지시면 됩니다. Claude 가 최신 파일로 다시 복사해줘요. 본인이 수정한 CLAUDE.md 는 backup 폴더에 남으니 필요한 부분을 새 CLAUDE.md 에 다시 합칠 수 있습니다.

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
