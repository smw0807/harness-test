# harness-test

Google Drive의 이력서를 참고해 잡코리아 공고를 찾고, 근거를 붙여 지원 적합성을 검토하며, 이력서 수정본을 만드는 개인용 하네스다.

현재는 **Codex가 실행할 설정·에이전트·스킬과 구현 문서**를 제공한다. TypeScript CLI와 `package.json`은 아직 구현하지 않았다. 패키지 매니저는 pnpm을 사용한다.

## 시작하기

1. [실행 환경과 MCP 설정](docs/setup.md)을 확인하고 저장소 루트에서 `codex`를 실행한다.
2. 사용할 이력서의 Drive URL 또는 로컬 파일 경로를 지정한다. Drive 접근 도구가 없는 환경에서는 로컬 파일로 진행할 수 있다.
3. 아래처럼 스킬을 호출한다. 실제 산출물은 Git에서 제외한 `.local/`에 저장한다.

```text
$resume-intake 이 Drive 이력서를 분석해줘: <이력서 URL>
$job-search 이번에 분석한 이력서로 서울의 TypeScript 개발 공고를 찾아줘
$resume-tailor 이 로컬 이력서의 프로젝트 설명을 간결하게 수정해줘: <파일 경로>
```

`<...>`는 실제 입력으로 바꾼다. 수정할 원본이 여러 개면 사용할 파일을 먼저 확정한다. 공고 탐색에는 브라우저·웹 검색 또는 사용자가 제공한 공고 본문이 필요하다.

## 구성

| 경로 | 목적 |
| --- | --- |
| [AGENTS.md](AGENTS.md) | 프로젝트 기준과 코딩 컨벤션 |
| [.codex/config.toml](.codex/config.toml) | 최신 웹 검색, 병렬 작업 수, 선택적 Playwright MCP |
| [.codex/agents/](.codex/agents/) | 이력서 분석·공고 조사·이력서 편집·검토 역할 |
| [.agents/skills/](.agents/skills/) | `resume-intake`, `job-search`, `resume-tailor` 반복 작업 |
| [config/preferences.example.json](config/preferences.example.json) | 개인 희망 조건의 예제 |
| [docs/workflows.md](docs/workflows.md) | 작업 흐름, 역할 분배, 재개 방법 |
| [docs/data-contracts.md](docs/data-contracts.md) | 실행 상태와 산출물의 공통 형식 |
| [docs/implementation-plan.md](docs/implementation-plan.md) | TypeScript CLI 구현 순서와 완료 기준 |

## 현재 연결 범위

역할과 스킬은 저장소에서 발견되며 실제 사용 가능 여부는 실행한 호스트에 따른다. 모델·권한·인증은 사용자 환경을 상속한다. Playwright 설정은 버전을 고정한 비활성 상태로 제공하며, [설정 가이드](docs/setup.md#playwright-mcp)에 활성화 방법이 있다. Drive 인증과 잡코리아 API 이용 권한은 별도로 필요하다.
