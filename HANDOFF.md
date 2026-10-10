# HANDOFF

## 지금까지 한 일 (2026-10-07, 2차)

- **할 일과 분리된 독립 "다이어리" 탭**을 추가했습니다.
  - 상단에 "할 일"/"다이어리" 탭 버튼(`#view-tabs`)을 추가해 `#view-todo`(기존 캘린더+체크리스트)와 `#view-diary`(신규)를 토글합니다. 날짜는 `selectedDate`를 그대로 공유하므로 할 일 탭 캘린더에서 고른 날짜가 다이어리 탭에도 그대로 반영됩니다.
  - 기존 "오늘의 한 줄"(`#memo-input`, `notes-v1`)은 삭제하고, `migrateNotesToDiary()`로 기존 데이터를 새 `diary-v1`에 1회성·비파괴적으로 이전했습니다(`notes-v1`은 원본 그대로 남겨둠, `escapeHTML` 재사용).
  - 다이어리 본문은 `contenteditable` + `document.execCommand`(굵게/목록)로 서식 입력 가능, `diary-v1`(`{ [isoDate]: HTML }`)에 날짜당 하나씩 저장됩니다.
  - 기분 이모지 스탬프(`moods-v1`, `{ [isoDate]: emoji }`)를 추가해 날짜 제목 옆에 도장처럼 회전시켜 표시, 같은 이모지 재클릭 시 해제됩니다.
  - 연속 작성일(streak)과 글자 수를 다이어리 하단에 표시(`computeStreak()`가 오늘부터 거슬러 올라가며 계산).
  - Claude in Chrome으로 직접 확인: 마이그레이션(`notes-v1`→`diary-v1` 자동 변환, 원본 보존), 탭 전환, 날짜 공유, 굵게/목록 서식, 기분 스탬프 토글, streak/글자 수 갱신, 빈 내용일 때 placeholder 복귀, 새로고침 후 전부 유지, 라이트/다크 모드 대비, 기존 할 일 기능(체크/삭제/TOP3/캘린더) 무회귀 확인. 콘솔 에러 없음.
  - 이번 범위에서 제외: 캘린더 화면 변경, 검색 기능, 날씨 자동 태그, 사진/스티커 첨부, 계정 동기화.

## 지금까지 한 일 (2026-10-07)

- **마감일 지난 항목 빨간 표시** 기능을 구현했습니다.
  - `renderChecklist()`에서 각 todo의 `isOverdue = todo.deadline && todo.deadline < todayStr && !todo.done`를 계산해 `li.todo-item`에 `overdue` 클래스를 조건부로 추가했습니다.
  - `renderTop3()`에도 동일한 로직(`isOverdueTop3`)을 적용해 TOP3에 뜬 항목도 같은 방식으로 강조됩니다.
  - CSS: `.todo-item.overdue .label`과 `.top3-item.overdue .top3-text`에 기존 `--stamp`(빨간 계열) 색상을 재사용해 텍스트 색을 바꿨습니다. 새 색상 변수는 추가하지 않았습니다.
  - 마감일 입력칸(`.deadline-input`)도 지난 날짜면 `border-color`/`color`를 `--stamp`로 바꿔 함께 강조하도록 했습니다.
  - 완료(`done`)된 항목은 `isOverdue` 조건에서 제외되므로 `overdue`와 `done` 스타일이 동시에 적용되지 않습니다.
  - 구문 검증: inline script를 추출해 `node --check`로 문법 오류 없음을 확인했습니다.
  - 브라우저 동작 확인: 로컬 HTTP 서버(`python -m http.server`)로 `index.html`을 열어 Claude in Chrome으로 직접 확인했습니다. 마감일을 지난 날짜(2026-01-01)로 바꾼 미완료 항목("회의 자료 준비")이 체크리스트/TOP3/마감일 입력창 모두에서 빨간색으로 표시되었고, 완료된 항목("이메일 확인하기")은 마감일이 지나도 빨간색으로 표시되지 않음(의도대로)을 확인했습니다. 라이트/다크 모드 둘 다 대비가 잘 보였습니다.

## 다음에 할 일

- 특별히 예정된 작업 없음.

## 참고
- 작업 범위 규칙(`CLAUDE.md` 참고): 명시적으로 요청받은 작업만 수행하고, 범위 밖 개선은 제안 후 승인받기로 되어 있습니다.
