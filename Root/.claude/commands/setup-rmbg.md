# Setup Remove Background Server

Remove Background MCP 서버를 설정합니다.

## 사전 요구사항

- Python 3.10+
- (선택) NVIDIA GPU with CUDA 12.0+

## 설정 단계

### 1. 가상환경 생성 및 활성화

```bash
cd tools/remove-bg-server
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/macOS
source venv/bin/activate
```

### 2. 의존성 설치

```bash
pip install -r requirements.txt
```

### 3. 환경 설정

```bash
cp .env.example .env
# 필요시 .env 편집
```

### 4. 서버 실행

```bash
python -m src.main
```

서버가 http://localhost:17920 에서 실행됩니다.

### 5. MCP 연동 (선택)

`.mcp.json`에 추가:

```json
{
  "mcpServers": {
    "remove-bg": {
      "type": "http",
      "url": "http://127.0.0.1:17920/mcp"
    }
  }
}
```

## Docker 실행 (대안)

```bash
cd tools/remove-bg-server

# GPU 모드
docker compose up -d

# CPU 모드
docker compose -f docker-compose.yml -f docker-compose.cpu.yml up -d
```

## 테스트

```bash
# 헬스체크
curl http://localhost:17920/health

# 모델 목록
curl http://localhost:17920/models
```

## 문제 해결

### GPU가 인식되지 않는 경우

- CUDA 드라이버 설치 확인
- `nvidia-smi` 명령으로 GPU 상태 확인
- PyTorch CUDA 버전 호환성 확인

### 모델 다운로드 실패

- 인터넷 연결 확인
- Hugging Face rate limit 시 HF_TOKEN 환경변수 설정

### 메모리 부족

- `RMBG_MAX_CACHED_MODELS=1` 로 캐시 크기 줄이기
- 작은 모델 (birefnet-general) 사용
