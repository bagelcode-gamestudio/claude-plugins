# MCP Server Check Skill

MCP 도구 사용 시 자동으로 서버 상태를 확인하고, 필요시 시작 방법을 안내합니다.

## 트리거 조건

다음 상황에서 자동으로 트리거됩니다:

| 조건 | 대상 서버 |
|------|-----------|
| `mcp__code-index__*` 도구 호출 실패 | code-index (17910) |
| `mcp__remove-bg__*` 도구 호출 실패 | remove-bg (17920) |
| MCP 연결 에러 메시지 | 해당 서버 |

## 서버 목록

| 서버 | 포트 | 상태 확인 | 시작 명령어 |
|------|------|-----------|-------------|
| unity-bridge | 17902 | Unity Editor에서 자동 시작 | - |
| code-index | 17910 | `curl http://127.0.0.1:17910/health` | `/reindex` 또는 수동 |
| remove-bg | 17920 | `curl http://127.0.0.1:17920/health` | `/start-rmbg` |

## 동작 흐름

### 1. code-index 도구 실패 시

```
1. health 체크: curl http://127.0.0.1:17910/health
2. 실패하면:
   - "code-index 서버가 실행 중이 아닙니다."
   - "/reindex 명령어로 서버를 시작하세요."
   - 또는 자동 시작 시도
```

### 2. remove-bg 도구 실패 시

```
1. health 체크: curl http://127.0.0.1:17920/health
2. 실패하면:
   - "remove-bg 서버가 실행 중이 아닙니다."
   - "/start-rmbg 명령어로 서버를 시작하세요."
   - 또는 자동 시작 시도
```

## 자동 시작 로직

사용자가 명시적으로 요청하지 않아도, MCP 도구 사용이 필요한 상황에서:

### code-index 자동 시작

```bash
# 1. 의존성 확인
ls tools/code-index-server/node_modules || npm install

# 2. 서버 시작 (백그라운드)
cd tools/code-index-server && npm run dev -- http 17910

# 3. 준비 대기
sleep 5 && curl http://127.0.0.1:17910/health
```

### remove-bg 자동 시작

```bash
# Windows PowerShell
cd tools/remove-bg-server
& .\venv\Scripts\python.exe -m uvicorn src.main:app --host 0.0.0.0 --port 17920
```

## 에러 메시지 처리

| 에러 패턴 | 대응 |
|-----------|------|
| `ECONNREFUSED` | 서버 미실행, 시작 안내 |
| `timeout` | 서버 과부하, 재시도 안내 |
| `404 /mcp` | 잘못된 엔드포인트 |

## 사용자 알림 형식

```markdown
⚠️ **MCP 서버 연결 실패**

| 서버 | 상태 |
|------|------|
| code-index | ❌ 미실행 |
| remove-bg | ✅ 실행 중 |

**해결 방법:**
- `/reindex` - code-index 서버 시작 및 인덱싱
- `/start-rmbg` - remove-bg 서버 시작
```

## 참고

- unity-bridge는 Unity Editor 실행 시 자동 시작
- code-index는 임베딩 기능에 OPENAI_API_KEY 필요
- remove-bg는 첫 실행 시 모델 다운로드 (~500MB)
