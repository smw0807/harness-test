# 실행 환경과 MCP 설정

## 적용 범위

설정 형식 확인 기준은 **Codex CLI 0.167.0, 2026-11-16**이다. 저장소 루트에서 실행하고, 호스트에서 이 프로젝트를 신뢰하도록 설정해야 프로젝트 `.codex/config.toml`이 적용된다. 전역 설정이나 인증 파일은 저장소에 복사하지 않는다.

```sh
codex --version
node --version
pnpm --version
codex
```

현재 `package.json`이 없으므로 프로젝트용 `pnpm install`, `pnpm dev`, `pnpm test`는 아직 제공하지 않는다. Node.js는 Active LTS를 권장한다. 모델·추론 수준·승인·샌드박스 설정은 사용자 환경을 상속하며 프로젝트에서 고정하지 않는다.

## 에이전트와 스킬 발견

- `.codex/agents/*.toml`: `name`, `description`, `developer_instructions`로 역할을 정의한다. 파일의 `name`이 호출 이름이다.
- `.agents/skills/<이름>/SKILL.md`: 작업 이름과 설명으로 선택된다. `$resume-intake`처럼 명시적으로 호출할 수도 있다.
- CLI의 `/skills`에서 스킬을 확인한다. 새 파일이 나타나지 않으면 세션을 다시 시작한다.
- 부모 에이전트가 `resume_analyst`, `job_researcher`, `resume_editor`, `reviewer`를 작업에 맞게 사용한다. 역할 파일은 별도 실행 프로그램이 아니다.

하위 에이전트는 부모의 도구·권한을 상속한다. `reviewer`의 편집 금지는 역할 지침이며 독립적인 읽기 전용 샌드박스를 만들지는 않는다. 병렬 실행은 서로 다른 산출물에만 배정한다.

공식 기준: [프로젝트 설정](https://learn.chatgpt.com/docs/config-file/config-basic), [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents), [Skills](https://learn.chatgpt.com/docs/build-skills).

## Google Drive/Docs

1. 현재 호스트에서 Google Drive 커넥터 또는 검증된 Drive MCP의 검색·메타데이터·다운로드 도구가 제공되는지 확인한다.
2. 계정의 파일 접근 권한을 확인하고 사용할 이력서 URL/ID를 지정한다. 호스트의 로그인 절차를 사용한다.
3. Google Docs는 필요한 형식으로 내보내고 저장된 DOCX/PDF는 원본 파일을 다운로드한다. 사본을 `.local/resumes/<resume-id>/original/`에 보존한다.

현재 세션에 있는 커넥터가 다른 CLI나 새 호스트에 자동으로 연결되지는 않는다. 이 저장소는 확인되지 않은 Drive MCP 주소를 설정하지 않는다. 접근 실패는 `resume_intake` 단계의 이슈로 기록한다. 로컬 대체 파일로 요청을 충족했다면 단계를 완료할 수 있고, 사용자가 요구한 Drive 원본을 확보하지 못했다면 실제 완료 범위에 따라 `blocked` 또는 `partial`로 남긴다. 독립 Drive 클라이언트와 OAuth 구현은 [구현 계획](implementation-plan.md)에서 다룬다.

## Playwright MCP

`.codex/config.toml`에는 `pnpm dlx @playwright/mcp@0.0.85` 설정이 있다. 서버 자동 실행·패키지 다운로드는 처음부터 켜지 않으며, 브라우저를 사용할 세션에서 다음처럼 활성화한다.

```sh
codex -c mcp_servers.playwright.enabled=true
```

계속 사용하려면 프로젝트 설정의 `enabled = false`를 `true`로 바꾼다. `pnpm dlx`는 첫 실행 시 서버 패키지를 내려받을 수 있다. 서버 연결은 브라우저 설치·페이지 접근까지 보장하지 않으므로 사용 가능한 브라우저를 확인하고 필요한 설치는 공식 안내를 따른다.

- 저장소 루트에서 실행한다. `.local/browser` 출력 경로는 서버 작업 디렉터리를 기준으로 하므로 다른 디렉터리에서 실행할 때는 절대 경로로 조정한다.
- 기본 설정은 `--isolated`로 브라우저 프로필을 메모리에 유지한다. 기존 개인 브라우저의 로그인 세션을 자동 재사용하지 않는다.
- 로그인 상태 유지가 필요하면 공식 `--storage-state` 또는 전용 `--user-data-dir` 설정을 사용하고 상태 파일은 `.local/`에 둔다. 병렬 작업은 각각 다른 프로필을 사용한다.
- 로그인·CAPTCHA·접근 차단 시 미확인 상태를 기록한다. 사용자가 제공한 URL·공고 본문이나 사용자 브라우저 조작으로 이어간다.
- 패키지 버전 변경 시 공식 변경 사항과 옵션을 확인하고 연결·페이지 읽기를 다시 점검한다.

참고: [Codex MCP 설정](https://learn.chatgpt.com/docs/extend/mcp), [Playwright MCP](https://github.com/microsoft/playwright-mcp).

## 희망 조건과 개인 데이터

```sh
mkdir -p .local
cp config/preferences.example.json .local/preferences.json
```

`.local/preferences.json`은 스킬이 읽는 개인 조건이다. 아직 CLI가 자동 로드하는 설정은 아니다. 배열이 비어 있으면 해당 조건을 제한하지 않고 `null`은 미지정을 뜻한다. `minimumAnnualSalaryKrw`는 원 단위 연봉이며, 급여 미기재 공고를 자동 탈락시키지 않는다. `maxResults`는 최종 보고서의 목표 건수다.

`.local/`과 `.env*`는 Git에서 제외한다. 이력서·쿠키·토큰을 문서나 공개 검색어에 넣지 않는다. 저장 위치를 바꾸면 파일을 만들기 전에 제외 규칙부터 적용한다.

## 연결 점검

1. `/skills`에서 스킬 3개를 확인하고 역할 이름을 지정한 작업이 인식되는지 확인한다.
2. Drive는 메타데이터 조회, 브라우저는 공개 페이지 읽기부터 실제 동작을 확인한다.
3. 연결 실패 시 도구 이름, 실패 단계, 민감정보를 제거한 오류 요약을 `run.json`에 남긴다. 확인하지 않은 연결을 완료로 표시하지 않는다.

이 문서 구성 작업에서는 실제 Drive 파일 접근, 브라우저 로그인, 채용공고 탐색을 수행하지 않는다. 동작 확인은 각 작업의 실제 실행 기록에 남긴다.
