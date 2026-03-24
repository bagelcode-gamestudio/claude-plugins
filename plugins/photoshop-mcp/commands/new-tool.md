---
description: Request a new Photoshop MCP tool — interview, generate, test, merge, notify
argument-hint: <what you want to do in Photoshop, e.g. "Inner Glow 넣고 싶어">
---

# New Photoshop Tool Request

아트직군이 자연어로 Photoshop 도구를 요청하면, 인터뷰 → 실현 가능성 판단 → 코드 생성 → QA → 머지 → Slack 알림까지 자동으로 수행합니다.

## Prerequisites

- photoshop-mcp 레포가 로컬에 클론되어 있어야 함
- 레포 경로를 모르면 `find ~ -name "photoshop-mcp" -type d -maxdepth 3 2>/dev/null` 로 찾기
- 레포가 없으면: `git clone https://github.com/bagelcode-gamestudio/photoshop-mcp.git`

## Flow

### Phase 1: 자연어 인터뷰

사용자의 시각적 의도를 파악한다. **기술 용어를 쓰지 않는다.**

1. 사용자가 인자로 요청을 넘겼으면 바로 시작. 아니면 "어떤 효과를 원하세요?" 질문
2. 최대 3개 질문으로 의도 파악:
   - **무엇을**: 어떤 효과? (예: "빛나는 느낌", "안쪽 그림자")
   - **어떤 느낌으로**: 강도/색상 등을 자연어로 (예: "은은하게", "노란빛")
   - **확인**: "이런 느낌으로 만들게요, 괜찮아요?"

질문할 때 반드시 선택지를 제공 (강하게 / 보통 / 은은하게). 기술 파라미터는 묻지 않는다.

### Phase 2: 도구 존재 여부 확인

photoshop-mcp 레포의 `registry/tools.json`을 읽어 스캔.

- **이미 있음** → 바로 실행. 종료.
- **유사한 도구** → "이미 있는 [도구명]으로 비슷하게 할 수 있는데, 써볼까요?"
- **없음** → Phase 3 진입

### Phase 3: 실현 가능성 검토 (사용자에게 비노출)

Agent가 내부적으로 판단:

1. context7으로 Photoshop batchPlay API 문서 조회
2. UXP sandbox 제약 확인
3. 기존 `plugin/commands/` 유사 코드 참조
4. 외부 의존성 범위 확인

판정:
- ✅ 가능 → Phase 4
- ⚠️ 제한적 → 사용자에게 제약 설명, 진행 여부 확인
- ❌ 불가 → "이건 Photoshop API로 자동화하기 어려워요. 수동 방법을 안내할게요."

### Phase 4: 의도 → 파라미터 매핑

Agent가 자연어를 기술 파라미터로 변환:
- "은은한 발광" → opacity:40, spread:20, size:30 등
- 합리적 디폴트 세팅
- 사용자에게 결과 설명 (기술 파라미터 숨김): "부드러운 노란빛 Inner Glow를 적용할게요"

### Phase 5: 코드 생성

photoshop-mcp 레포에서 worktree를 생성하고:

1. **Python MCP 함수** 작성 (TDD: 테스트 먼저 → 구현)
   - `mcp/tools/{category}.py`에 추가 또는 새 파일 생성
   - 함수명: snake_case, 모든 파라미터에 디폴트값
2. **JS UXP 핸들러** 작성
   - `plugin/commands/{category}.js`에 추가
   - batchPlay descriptor 포함
   - `plugin/commands/index.js`에 import 추가
3. **registry/tools.json** 엔트리 추가
   - description은 한국어
   - enabled: true

### Phase 6: 4 Gate QA

**Gate 1 — 코드 품질:**
```bash
cd <photoshop-mcp-repo>
ruff check mcp/
pytest tests/unit/ -v
```

**Gate 2 — 실제 동작 검증 (Photoshop 필요):**
- Pre-flight: Proxy 실행 중? Photoshop 실행 중? Plugin 연결됨?
- 미충족 시 사용자에게 안내
- 디폴트 파라미터로 도구 호출 → 성공 확인

**Gate 3 — 회귀 테스트:**
```bash
pytest tests/regression/ -v
python scripts/check_registry.py
```

**Gate 4 — 비개발자 안전성:**
- layer_id만으로 호출 가능한지
- registry description이 한국어인지
- 에러 메시지가 이해 가능한지

### Phase 7: 머지 + 알림

- ALL PASS → worktree를 main에 머지 → push
- GitHub Actions가 `#mcp-photoshop` 채널에 자동 알림
- 사용자에게: "도구 준비됐어요, 써볼까요?"

실패 시 (최대 3회 재시도):
- 3회 초과 → worktree 보존, Slack에 에러 요약, draft PR 생성

## 코드 생성 규칙

| 항목 | 규칙 |
|------|------|
| Python 함수명 | snake_case (`add_inner_glow_layer_style`) |
| JS 핸들러명 | camelCase (`addInnerGlowLayerStyle`) |
| action 이름 | camelCase (Python ↔ JS 통신용) |
| 디폴트값 | 모든 파라미터 필수 (layer_id 제외) |
| docstring | 영문 (Claude에 노출) |
| registry description | 한국어 (비개발자용) |
| mutable dict 디폴트 | `None` + 함수 내부 초기화 |
