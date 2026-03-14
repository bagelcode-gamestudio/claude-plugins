---
name: code-index
description: 코드 심볼/시맨틱 검색을 위한 인덱스 관리. 클래스 찾기, 구조 분석, 코드 검색 시 자동으로 사용됨.
---

# Code Index 스킬

코드베이스의 심볼, 참조, 타입 계층을 인덱싱하여 검색합니다.

## 언제 사용하나요?

### 자동 트리거 조건

| 트리거 | 상황 |
|--------|------|
| 심볼 검색 | "DoorController 클래스 찾아줘" |
| 참조 검색 | "ITool 어디서 사용되는지", "이 메서드 호출하는 곳" |
| 구조 분석 | "이 인터페이스 구현체가 뭐야", "상속 구조 보여줘" |
| 코드 검색 | "door animation 관련 코드" |

### 자동 트리거되는 에이전트

| 에이전트 | 활용 시점 |
|----------|----------|
| **architect** | 구조 분석, 설계 검토, 인터페이스 구현체 확인 |
| **critic** | 리스크 분석, 영향도 평가, 대안 검토 |
| **executor-code** | 구현 전 기존 코드 확인 |
| **reviewer** | 코드 리뷰 시 참조 확인 |

## 인덱스 재구축

```bash
/reindex         # 기본 (incremental)
/reindex full    # 전체 재구축
```

## MCP Tools

| Tool | 설명 | 파라미터 |
|------|------|----------|
| `code.search.symbol` | 심볼 검색 | query, kind?, limit? |
| `code.search.semantic` | 시맨틱 검색 | query, limit? |
| `code.index.status` | 인덱스 상태 | - |
| `code.get.references` | 참조 위치 검색 | symbolName, kind?, limit? |
| `code.get.hierarchy` | 타입 계층 검색 | symbolName, direction?, maxDepth? |

### 사용 예시

```json
// 심볼 검색
{"tool": "code.search.symbol", "args": {"query": "DoorController", "kind": "class"}}

// 참조 검색
{"tool": "code.get.references", "args": {"symbolName": "ITool", "limit": 20}}

// 구현체 찾기
{"tool": "code.get.hierarchy", "args": {"symbolName": "ITool", "direction": "down"}}
```

## 검색 결과 형식

### 심볼 검색

```markdown
## DoorController
- **Kind**: class
- **File**: Packages/.../DoorController.cs:10-196
- **Signature**: `public class DoorController : MonoBehaviour`
```

### 참조 검색

```markdown
## ITool
- **Kind**: interface
- **Defined at**: Packages/.../ITool.cs:7
- **Total references**: 75

### References
- [unknown] Packages/.../ToolRegistry.cs:24
- [unknown] Packages/.../PluginLoader.cs:27
```

### 타입 계층

```markdown
## ITool
- **Kind**: interface
- **Location**: Packages/.../ITool.cs:7

### Implementations
- AssetSearchTool [implements]
- EditorConsoleReadTool [implements]
...
Total: 0 parents, 56 implementations
```

## 에러 핸들링

### 서버 미실행
```
Error: Connection refused
→ cd tools/code-index-server && node dist/index.js http &
```

### 인덱스 없음
```
Error: No index found
→ /reindex full
```

## 인덱스 최신화 전략

| 상황 | 권장 액션 |
|------|----------|
| 브랜치 전환 | `/reindex full` |
| 파일 수정 | `/reindex` (incremental) |
| 인덱스 없음 | `/reindex full` |
