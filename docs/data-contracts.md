# 실행 데이터 계약

이 문서는 스킬·역할·향후 TypeScript CLI가 공유할 형식이다. 현재는 문서상의 계약이며 런타임 스키마 검증기는 아직 없다. JSON 필드는 `camelCase`, 시간은 UTC ISO 8601을 사용한다. 선택 값을 알 수 없으면 `null` 또는 명시된 빈 배열로 남기고 추정 사실로 채우지 않는다.

## 저장 위치

```text
.local/preferences.json
.local/resumes/<resume-id>/original/<원본 파일>
.local/resumes/<resume-id>/revised/<수정본 파일>
.local/runs/<run-id>/run.json
.local/runs/<run-id>/profile.json
.local/runs/<run-id>/jobs.json
.local/runs/<run-id>/matches.json
.local/runs/<run-id>/report.md
.local/runs/<run-id>/edits.md
```

새 실행의 `run-id`는 `YYYYMMDDTHHMMSSZ-<고유 접미사>`로 생성하고 기존 디렉터리와 충돌하지 않게 한다. 기존 실행을 재개할 때는 같은 `runId`와 디렉터리를 사용한다. `resume-id`는 실행 간 같은 원본을 추적하는 로컬 식별자이며 이름·연락처를 넣지 않는다. 파일 경로는 저장소 루트 기준이다. 파일 이름에 들어갈 회사명·직무는 경로 구분자를 제거해 정규화한다. 외부 입력으로 `.local/` 밖에 저장하지 않는다.

## 실행 기록: `run.json`

부모 에이전트가 작성한다. `stage`는 `resume_intake`, `job_search`, `resume_tailor`, `review` 중 하나다. 단계·실행의 `status`는 `pending`, `running`, `completed`, `partial`, `blocked`, `failed` 중 하나다. 요청하지 않은 단계는 `stages`에서 생략한다.

```json
{
  "schemaVersion": 1,
  "runId": "20261116T010000Z-example",
  "status": "running",
  "createdAt": "2026-11-16T01:00:00Z",
  "updatedAt": "2026-11-16T01:00:00Z",
  "resumeId": "resume-example",
  "stages": [
    {"stage": "resume_intake", "status": "running", "artifacts": []}
  ],
  "issues": []
}
```

`artifacts`는 해당 단계가 실제 생성한 파일 경로의 배열이다. `issues`는 `{ "stage": "job_search", "code": "tool_unavailable", "message": "브라우저 도구 없음" }` 형태로 민감정보 없이 기록한다. 전체 상태는 요청한 모든 단계가 완료된 경우에만 `completed`다. 유효한 부분 결과가 있으면 `partial`, 선행 조건만 부족하면 `blocked`, 처리 실패로 결과가 없으면 `failed`로 통합한다.

## 이력서 프로필: `profile.json`

필수 필드: `schemaVersion`, `resumeId`, `source`, `facts`, `unknowns`.

- `source`: `kind`(`drive`/`local`), `fileId`, `url`, `mimeType`, `modifiedAt`, `localPath`, `sha256`, `extractedAt`. Drive 전용 값은 로컬 입력일 때 `null`이다. 해시를 계산하지 못하면 `null`과 이유를 기록한다.
- `facts`: `{ "id": "fact-1", "category": "skill", "text": "TypeScript 사용 경험", "evidence": [{"locator": "프로젝트 A / 기술 스택", "excerpt": "TypeScript"}] }` 형태다. `category`는 `skill`, `experience`, `project`, `education`, `certification` 중 하나다.
- `unknowns`: 읽기 실패·불명확한 기간 등 확인할 사항의 문자열 배열이다. 근거 위치는 페이지·제목·표 등 재확인 가능한 단위로 남긴다.

연락처·주민등록번호·사진 등 추천에 불필요한 항목은 `facts`로 옮기지 않는다. 이력서 원문과 추출 사실은 모두 개인 데이터로 취급한다.

## 채용공고: `jobs.json`

최상위 형식은 `{ "schemaVersion": 1, "jobs": [] }`다. 각 공고는 다음 필드를 사용한다.

| 필드 | 내용 |
| --- | --- |
| `id`, `source`, `sourceJobId`, `url` | 로컬 공고 ID, `jobkorea`, 원문 ID(미확인 시 `null`), 확인한 상세 URL |
| `company`, `title`, `location`, `employmentType`, `salaryText` | 원문 사실. 미기재 값은 `null` |
| `required`, `preferred` | `{ "id": "req-1", "text": "TypeScript 경험", "evidence": "자격요건 항목" }` 배열 |
| `deadlineText`, `deadlineAt` | 원문 마감 표현, 명확히 해석할 수 있는 경우의 시각. 상시채용·불명확 시 `deadlineAt`은 `null` |
| `status` | `open`, `closed`, `unknown` |
| `checkedAt`, `verification` | 확인 시각, `detail_page`/`official_api`/`user_provided`/`search_only` |
| `unknowns` | 미확인·접근 실패 항목의 배열 |

검색 결과만 읽었거나 오래된 사용자 본문만 있으면 현재 상태를 `open`으로 확정하지 않는다. `status`와 마감 해석은 실제 근거에 따라 구분한다. 외부 쓰기에 사용할 URL·파일 ID는 도구로 관찰한 값을 사용한다.

## 매칭 결과: `matches.json`

최상위 형식은 `{ "schemaVersion": 1, "matches": [] }`다. 항목은 `jobId`, `assessments`, `strengths`, `gaps`, `questions`, `recommendation`을 가진다.

`assessments`는 `{ "requirementId": "req-1", "kind": "required", "status": "met", "factIds": ["fact-1"], "reason": "기술 경험의 원본 근거 확인" }` 배열이다. `kind`는 `required`/`preferred`, `status`는 `met`/`not_met`/`unknown`이다. `factIds`는 같은 실행의 `profile.json`을 참조한다.

`strengths`, `gaps`, `questions`는 문자열 배열이고 `recommendation`은 `recommend`, `consider`, `skip`, `unknown` 중 하나다. 이력서에 근거가 없으면 기본적으로 `unknown`이며 명시적 불일치 근거가 있을 때 `not_met`으로 판단한다. 합격 확률은 만들지 않는다.

## 사람에게 제공할 결과

- `report.md`: 입력 프로필·조건·확인 시각, 추천 순서와 이유, 원문 링크, 필수·우대 조건별 근거, 부족한 점, 미확인·제외 공고와 이유.
- `edits.md`: 원본·수정본 경로, 변경 전후 요약, 변경 이유와 원본 근거, 새로 확인할 사실, 렌더링 검증 결과 또는 검증 한계.
- 검토자는 지적의 심각도·경로·근거·영향·수정 제안을 부모에게 반환한다. 부모가 실행 기록에 반영한다.

예제의 `example` 값은 모두 합성 데이터다. 실제 개인 산출물은 커밋하지 않는다.
