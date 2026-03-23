# Phase 1: 회의록 조회

## 진입 조건
- Phase 0 완료 (설정 파일 유효, 연동 확인됨)

## 종료 조건
- 회의록 데이터 수집 완료 (0건이면 "오늘은 회의가 없습니다 🎉" 출력 후 전체 종료)
- `--preview` 플래그 시 미리보기 확인 완료

---

## 조회

설정 파일에서 `enabled: true`인 소스만 조회한다.
활성화된 소스가 2개면 **병렬**로 조회 (Agent 도구로 서브에이전트 활용 가능):

### caret

caret MCP 도구 `caret_list_notes`를 호출하여 오늘 날짜 기준 회의록 전체 조회.

**가드레일:**
- 개별 노트 조회 시 `caret_get_note`의 파라미터명은 **`id`** (NOT `noteId`)
- 노트가 너무 길어 토큰 초과 에러 발생 시 (파일로 저장됨): 해당 파일을 Read 도구로 읽어서 내용을 파싱한 뒤, Bash로 즉시 삭제한다 (`rm -f {파일경로}`)

### Gemini 요약

Bash로 Google Drive REST API 직접 호출하여 오늘의 Gemini 회의록 검색:

```bash
curl -s -G -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  --data-urlencode "q=name contains 'Gemini가 작성한 회의록' and modifiedTime > '$(date -u +%Y-%m-%dT00:00:00Z)'" \
  --data-urlencode "fields=files(id,name,modifiedTime)" \
  --data-urlencode "orderBy=modifiedTime desc" \
  --data-urlencode "pageSize=20" \
  "https://www.googleapis.com/drive/v3/files"
```

- 파일명 패턴: `{회의명} - {YYYY/MM/DD} {HH:MM} KST - Gemini가 작성한 회의록`
  - 예: `[해외렌트카] Sync Meeting - 2026/03/19 16:00 KST - Gemini가 작성한 회의록`
- 검색된 문서의 본문 조회:
  ```bash
  curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://docs.googleapis.com/v1/documents/{문서ID}"
  ```
- 본문에서 요약, 액션 아이템, 참석자 정보 추출
- 파일명에서 회의명과 시간을 파싱하여 caret 데이터와 중복 매칭에 활용

---

## 중복 회의 처리 (두 소스 모두 활성화된 경우)

양쪽 소스의 회의를 **시간 + 제목**으로 매칭하여 중복 여부 판단:
- **caret에만 있는 회의** → caret 데이터로 분석
- **Gemini에만 있는 회의** → Gemini 요약 데이터로 분석
- **양쪽 모두 있는 회의** → 두 데이터를 교차 분석하여 더 풍부한 정리본 생성 (caret의 상세 기록 + Gemini의 AI 요약을 결합)

---

## 회의 0건 처리

양쪽 소스 모두 결과가 0건이면:
```
오늘은 회의가 없습니다 🎉
```
출력 후 종료. Phase 2 이후로 진행하지 않는다.

---

## 미리보기 (조건부)

**활성화 조건**: `--preview` 플래그만
**비활성화 조건**: 기본 모드, `--auto` 플래그

조회된 회의 목록을 출력:
```
📋 조회된 회의 {N}건:
1. {제목} ({시간}, {소스})
2. {제목} ({시간}, {소스})
...

이 내용으로 리포트를 생성하고 전송할까요?
```

- Y → Phase 2로
- N → 종료
