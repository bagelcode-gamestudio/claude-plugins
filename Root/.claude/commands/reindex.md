# Code Index Rebuild Command

코드 인덱스를 재구축합니다. 심볼 검색, 참조 검색, 시맨틱 검색에 사용됩니다.

## 사용법

```
/reindex         # incremental (변경 파일만)
/reindex full    # 전체 재구축
```

## 실행 단계

### 1. 서버 상태 확인 및 자동 시작

```bash
curl -s http://127.0.0.1:17910/health
```

**서버가 실행 중이 아니면 자동으로 시작:**

1. 의존성 확인:
```bash
# npm 패키지 설치 여부 확인
ls tools/code-index-server/node_modules 2>/dev/null || echo "NEED_INSTALL"
```

2. 의존성 미설치 시 설치:
```bash
cd tools/code-index-server && npm install
```

3. .env 파일 확인 (OPENAI_API_KEY 필요):
```bash
ls tools/code-index-server/.env 2>/dev/null || echo "NEED_ENV"
```
- 없으면 사용자에게 `.env.example` 복사 및 API 키 설정 안내

4. 서버 시작 (백그라운드, HTTP 모드):
```bash
cd tools/code-index-server && npm run dev -- http 17910
```

5. 서버 준비 대기 (최대 10초):
```bash
for i in {1..10}; do curl -s http://127.0.0.1:17910/health && break || sleep 1; done
```

### 2. 현재 인덱스 상태 확인

```bash
curl -s http://127.0.0.1:17910/api/index/status
```

### 3. 인덱싱 실행

**기본 (Tree-sitter + SCIP + Embedding):**
```bash
cd tools/code-index-server && npm run index
```

**Full rebuild:**
```bash
cd tools/code-index-server && npm run index -- --full
```

**빠른 인덱싱 (임베딩 없이):**
```bash
cd tools/code-index-server && npm run index -- --no-embeddings
```

**심볼만 (참조/임베딩 없이):**
```bash
cd tools/code-index-server && npm run index -- --no-embeddings --no-scip
```

### 4. 결과 보고

```markdown
## 인덱싱 결과

| 항목 | 수량 |
|------|------|
| 파일 | N개 |
| 심볼 | N개 |
| 참조 | N개 |
| 타입 관계 | N개 |
| 소요 시간 | N초 |
```

## 언제 사용하나요?

| 상황 | 권장 모드 |
|------|----------|
| 브랜치 전환 후 | `/reindex full` |
| 파일 수정 후 | `/reindex` (incremental) |
| 처음 설정 시 | `/reindex full` |
| 빠른 확인만 | `--no-embeddings` 옵션 |

## SCIP 파일 생성 (참조 검색용)

SCIP 파일이 없으면 참조 검색이 작동하지 않습니다.

**기존 SCIP 파일 확인:**
```bash
ls tools/code-index-server/data/*.scip
```

**SCIP 파일이 없을 때 생성:**

1. scip-dotnet 설치 확인:
```bash
dotnet tool list -g | grep scip-dotnet || dotnet tool install --global scip-dotnet
```

2. 주요 프로젝트 SCIP 생성:
```bash
# UnityBridge
scip-dotnet index Bakery.UnityBridge.Editor.csproj \
  --output tools/code-index-server/data/scip-unitybridge.scip \
  --skip-dotnet-restore

# UITK Runtime/Editor
scip-dotnet index Bakery.UITK.Runtime.csproj \
  --output tools/code-index-server/data/scip-uitk-runtime.scip \
  --skip-dotnet-restore

scip-dotnet index Bakery.UITK.Editor.csproj \
  --output tools/code-index-server/data/scip-uitk-editor.scip \
  --skip-dotnet-restore

# Assembly-CSharp (Playground 스크립트)
scip-dotnet index Assembly-CSharp.csproj \
  --output tools/code-index-server/data/scip-assembly.scip \
  --skip-dotnet-restore
```

`data/` 폴더의 `.scip` 파일은 `/reindex` 시 자동으로 처리됩니다.

## 주의사항

- 심볼 인덱싱: ~10초 (447 파일 기준)
- SCIP 참조 처리: ~5초 (기존 .scip 파일 있을 때)
- SCIP 파일 생성: ~20-60초 (프로젝트당)
- 임베딩: ~1분 (증분), ~3분 (전체)
- 임베딩 비용: ~80원/전체, ~1원/증분
- `OPENAI_API_KEY` 환경변수 필요 (임베딩용)
