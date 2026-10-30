# meerkatflow

[English](README.md)

요청 파악 → 설계 → 계획 → 서브에이전트 구현 → 검증(코드, 그다음 실제 실행)으로 이어지는 단계별 개발 흐름을 담은 Claude Code 플러그인이다. 코드 변경은 작게, 답변은 짧게 유지한다.

스킬은 `meerkatflow:<스킬>` 로 부른다(예: `meerkatflow:brainstorm`).

## 스킬

| 스킬 | 이럴 때 |
|---|---|
| `meerkatflow:brainstorm` | 새 기능·동작 변경에서 사용자가 원하는 것이 아직 열려 있을 때 |
| `meerkatflow:spec` | 의도는 합의됐고 설계를 정해 승인받아야 할 때 |
| `meerkatflow:plan` | 승인된 spec(또는 짧은 조사 문서)을 태스크 단위 계획으로 만들 때 |
| `meerkatflow:execute` | 승인된 계획을 태스크마다 구현 서브에이전트 하나로 실행할 때 |
| `meerkatflow:verifying-against-criteria` | 완성된 코드를 plan 의 검증 기준으로 채점할 때(90점 이상 통과) |
| `meerkatflow:verify-runtime` | 실행 중인 프로그램에서 런타임 항목을 하나씩 증거와 함께 확인할 때 |
| `meerkatflow:review` | plan 없이 현재 diff 를 리뷰할 때 |
| `meerkatflow:debug` | 버그·오류·예상과 다른 동작이 있을 때, 고치기 전에 |
| `meerkatflow:measure` | 실행 중인 시스템(브라우저, API, DB, 잡)에서 무언가를 확인할 때 |
| `meerkatflow:issue` | 조사 결과를 이슈 초안으로 나눌 때 |
| `meerkatflow:pr` | 커밋 메시지나 PR 본문 초안이 필요할 때 |
| `meerkatflow:resume` | 작업을 다른 세션에 넘기거나 이어받을 때 |

## 하는 일

- **대신 정하지 않고 묻는다.** 아이디어 승인, spec 승인, plan 승인은 서로 다른 단계다. spec 에 남은 "A 또는 B" 는 에이전트 기본값이 아니라 사용자가 정한다.
- **커밋은 사용자가 고른 방식대로만 한다.** `execute` 를 시작할 때 태스크별 커밋(에이전트가 태스크마다 feature 브랜치에 커밋, 푸시는 안 함)과 마지막에 한 번(에이전트는 커밋하지 않고 사용자가 커밋) 중 하나를 고른다. 어느 쪽이든 전체 변경은 끝에 한 번 리뷰한다(태스크별 리뷰는 되돌리기 어려운 태스크만).
- **증거 없이 끝났다고 하지 않는다.** 코드를 plan 의 검증 기준으로 채점하고 90점이 될 때까지 다시 검증한다. 이어서 실행 중인 프로그램에서 런타임 항목을 하나씩 확인한다. 화면은 에이전트가 스크린샷을 직접 열어 판정한다.
- **짧게 답한다.** 답부터 말한다. 예/아니오 질문은 첫 줄에 답하고, 자세한 근거는 요청하면 준다.
- **코드를 적게 쓴다.** 쓰기 전에 필요한가 → 저장소에 있나 → 플랫폼에 있나 → 설치된 의존성에 있나 → 한 줄로 되나 → 그래도 안 되면 동작하는 최소 코드 순서로 확인한다.

## superpowers + caveman + ponytail 과 비교

meerkatflow 는 superpowers 의 흐름, caveman 의 짧은 답변, ponytail 의 작은 코드 규칙을 플러그인 하나에 담았다. 2026-10-30 각 프로젝트 저장소(기본 브랜치)에서 잰 값:

| | superpowers + caveman + ponytail | meerkatflow |
|---|---|---|
| 매 세션 목록에 올라가는 스킬 | 43개 | 12개 |
| 하는 일이 겹치는 스킬(리뷰 ×4, 디버깅·조사 ×2, "완료 전 검증" ×2, 도움말 ×2) | 10개 | 0개 |
| 세션마다 고정으로 드는 맥락: 스킬 설명 + 세션 시작 규칙 | 약 18,700자(약 4,700 토큰) | 약 11,400자(약 2,850 토큰), **−39%** |
| 서브에이전트마다 주입되는 규칙 | 서브에이전트당 약 7,500자(약 1,900 토큰) | 0(구현자는 필요한 규칙만 태스크 프롬프트로 받는다) |
| 커밋 | superpowers 구현자가 태스크마다 커밋 | 실행마다 선택: 태스크별, 또는 없음(마지막에 사용자가 커밋) |
| 완성된 변경을 실행해서 항목별 증거와 함께 확인 | 없음 | `verify-runtime` |

측정 방법: 스킬 설명은 모든 `skills/*/SKILL.md` 의 description 길이, 세션 시작 규칙은 각 훅이 주입하는 내용(superpowers 는 `using-superpowers`, caveman·ponytail 은 훅이 불러오는 대표 스킬 본문으로 근사), 토큰은 글자 수 ÷ 4. 재지 않은 것: 매 프롬프트 리마인더, 결과 품질(실제 과제에서의 diff 크기·토큰·재작업). 같은 과제를 두 구성으로 돌린 비교는 아직 하지 않았다.

## 기본값과 덮어쓰기

meerkatflow 는 프로젝트를 가정하지 않는다. 아래는 기본값이고, 프로젝트의 `CLAUDE.md` 나 다른 플러그인의 세션 규칙이 이긴다.

| 항목 | 기본값 |
|---|---|
| 문서 언어 | 사용자의 언어 |
| 에이전트 문서 위치 `<docs>` | `.meerkatflow/` (git 에서 제외) |
| 검증 상태·피드백 | `.claude/verify/` (git 에서 제외) |
| 브라우저 도구 | 세션에 있는 브라우저 자동화 도구. 어느 도구를 어떻게 쓸지는 개인·프로젝트 플러그인의 `measure` 노트가 정한다 |
| 워크트리·브랜치 | `feature-<key>` / `feature/<key>` |
| `execute` 커밋 방식 | 실행마다 한 번 묻는다. `CLAUDE.md` 규칙으로 고정 가능. 답이 없으면 마지막에 사용자가 |
| 커밋·PR 양식 | 저장소의 최근 이력과 PR 템플릿 |
| 주석·테스트 파일 정책 | 없음. 규칙이 있으면 plan 의 Global Constraints 로 구현자에게 전달 |

덮어쓰려면 `CLAUDE.md` 에 규칙을 적거나, 세션 시작 훅을 가진 작은 개인 플러그인을 만든다(브라우저 설정이 있으면 도구 노트를 담은 `measure` 스킬도 함께).

## 요구 사항

- `node` 가 PATH 에 있어야 한다(훅이 node 로 규칙 파일을 출력한다).
- `git`. 스킬의 셸 예시는 bash 기준이다(Windows 는 Git Bash).

## 설치

```
claude plugin marketplace add <이 저장소 경로 또는 git URL>
claude plugin install meerkatflow@dev_meerkat
```

## 수정

이 저장소만 고친다. `~/.claude/plugins/cache` 는 고치지 않는다.

1. 파일 수정
2. `meerkatflow/.claude-plugin/plugin.json` 의 `version` 올리기
3. `CHANGELOG.md` 에 한 줄
4. `claude plugin marketplace update dev_meerkat` 후 `claude plugin update meerkatflow@dev_meerkat`

## 넣는 기준

- 어느 프로젝트에서나 맞는 규칙만 넣는다. 개인 취향과 프로젝트 사정은 각자의 플러그인이나 `CLAUDE.md` 에 둔다.
- 스킬은 다른 플러그인의 스킬 이름을 참조하지 않는다.
- 회사 코드, 내부 경로·API·컴포넌트 이름, 실제 커밋 제목을 예시로 쓰지 않는다.
- 금방 바뀌는 사실(도구 동작, 도구 이름)에는 확인한 연월을 붙인다.
- 한 분기 동안 안 쓴 스킬은 지운다.

## 출처

meerkatflow 는 [superpowers](https://github.com/obra/superpowers), [caveman](https://github.com/JuliusBrussee/caveman), [ponytail](https://github.com/DietrichGebert/ponytail) 의 아이디어에서 영감을 받아 새로 썼다. [NOTICE.md](NOTICE.md) 참고.
