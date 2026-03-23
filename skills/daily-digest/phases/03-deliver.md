# Phase 3: 전송

## 진입 조건
- Phase 2 완료 (포맷 생성됨)

## 종료 조건
- 활성화된 모든 채널로 전송 완료 (또는 실패 처리 완료)

---

활성화된 채널만 **병렬** 전송:

## 슬랙 AgentDeck 봇 전송

Bash로 Slack Web API를 직접 호출하여 DM을 전송한다.
Bot Token은 환경변수 `$AGENTDECK_BOT_TOKEN` 사용.
Slack mrkdwn 문법을 사용하며, `*bold*`/`_italic_` 대신 backtick(`` ` ``)으로 강조한다.

### 전송 순서 (`references/slack-format.md` 참조):

1. **내 할 일 먼저** — 모든 회의에서 추출한 내 담당 액션 아이템을 DM 1건으로 전송
2. **회의별 정리본** — 각 회의마다 핵심 임팩트를 제목으로 한 DM + 스레드 정리본
3. **종합** — 마지막에 오늘의 종합 DM 1건

### API 호출

각 DM 전송:
```bash
curl -s -X POST 'https://slack.com/api/chat.postMessage' \
  -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"channel": "{USER_ID}", "text": "{메시지}"}'
```

스레드 답글 전송 (응답의 `ts`를 `thread_ts`로):
```bash
curl -s -X POST 'https://slack.com/api/chat.postMessage' \
  -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"channel": "{USER_ID}", "text": "{정리본}", "thread_ts": "{ts}"}'
```

- 스레드 본문 5,000자 초과 시 여러 스레드 답글로 분할
- 전송 완료 후 Slack 메시지 링크를 사용자에게 안내

### 인증 에러

`invalid_auth` 또는 `token_revoked` 시:
"슬랙 Bot Token이 만료되었습니다. https://api.slack.com/apps 에서 Token을 재발급하고 `/digest-setup`으로 업데이트해주세요." 한 줄만 안내.

---

## poke 전송

Bash로 poke REST API 호출:
```bash
curl -s -X POST 'https://poke.com/api/v1/inbound-sms/webhook' \
  -H 'Authorization: Bearer {API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{"message": "{poke 요약 메시지}"}'
```
- API 키는 설정 파일의 `channels.poke.api_key` 사용
- 본문은 `references/poke-format.md` 참조

---

## 전송 결과

```
✅ 슬랙 전송 완료 (스레드 링크: {url})
✅ poke 전송 완료
```

일부 채널 전송 실패 시:
```
✅ 슬랙 전송 완료
❌ poke 전송 실패: {에러 메시지}
재시도하시겠어요?
```
