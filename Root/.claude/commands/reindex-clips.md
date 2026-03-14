# Clip Index Rebuild Command

애니메이션 클립 인덱스를 재구축합니다. 클립 검색, 태그 기반 필터링에 사용됩니다.

## 사용법

```
/reindex-clips         # incremental (변경 파일만)
/reindex-clips full    # 전체 재구축
```

## 실행 단계

### 1. 서버 상태 확인

```bash
curl -s http://127.0.0.1:17901/health
```

서버가 실행 중이 아니면:
```bash
cd tools/clip-index-server && npm start &
```

### 2. 인덱싱 실행

**기본 (증분 인덱싱):**
```bash
cd tools/clip-index-server && npm run index
```

**Full rebuild:**
```bash
cd tools/clip-index-server && npm run index -- --full
```

**특정 패턴만:**
```bash
cd tools/clip-index-server && npm run index -- --pattern "Assets/Animations/**/*.anim"
```

### 3. 결과 보고

```markdown
## 인덱싱 결과

| 항목 | 수량 |
|------|------|
| 파일 | N개 |
| 클립 | N개 |
| 소요 시간 | N초 |
```

## 언제 사용하나요?

| 상황 | 권장 모드 |
|------|----------|
| 새 애니메이션 추가 | `/reindex-clips` (incremental) |
| FBX 임포트 후 | `/reindex-clips` (incremental) |
| 브랜치 전환 후 | `/reindex-clips full` |
| 처음 설정 시 | `/reindex-clips full` |

## 주의사항

- 인덱싱 시간: ~1초 (100개 파일 기준)
- DB 위치: `tools/clip-index-server/data/clip-index.db`
- 서버 재시작 불필요 (DB는 실시간 반영)
