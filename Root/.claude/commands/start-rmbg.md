# Remove Background Server Start Command

배경 제거 MCP 서버를 시작합니다. BiRefNet AI 모델을 사용한 이미지 배경 제거 기능을 제공합니다.

## 사용법

```
/start-rmbg          # 서버 시작 (CPU 모드)
/start-rmbg gpu      # GPU 모드로 시작
/start-rmbg status   # 상태만 확인
```

## 실행 단계

### 1. 서버 상태 확인

```bash
curl -s http://127.0.0.1:17920/health
```

- 응답 있으면: 이미 실행 중, 상태 표시 후 종료
- 응답 없으면: 서버 시작 진행

### 2. 의존성 확인 및 설치

**Python venv 확인:**
```bash
ls tools/remove-bg-server/venv 2>/dev/null || echo "NEED_VENV"
```

**venv 없으면 생성:**
```bash
cd tools/remove-bg-server && python -m venv venv
```

**의존성 설치:**
```bash
# Windows
cd tools/remove-bg-server && .\venv\Scripts\pip.exe install -r requirements.txt

# Linux/macOS
cd tools/remove-bg-server && ./venv/bin/pip install -r requirements.txt
```

### 3. 서버 시작

**Windows (PowerShell):**
```powershell
cd tools/remove-bg-server
& .\venv\Scripts\python.exe -m uvicorn src.main:app --host 0.0.0.0 --port 17920
```

**Linux/macOS:**
```bash
cd tools/remove-bg-server && ./venv/bin/python -m uvicorn src.main:app --host 0.0.0.0 --port 17920
```

서버는 백그라운드로 실행합니다.

### 4. 서버 준비 대기

```bash
# 최대 30초 대기 (모델 로딩 시간 고려)
for i in {1..30}; do curl -s http://127.0.0.1:17920/health && break || sleep 1; done
```

### 5. 결과 보고

```markdown
## Remove-BG Server 상태

| 항목 | 값 |
|------|-----|
| 상태 | 실행 중 |
| 주소 | http://127.0.0.1:17920 |
| 디바이스 | CPU / GPU (CUDA) |
| 로드된 모델 | birefnet-general |
```

## MCP 도구

서버 시작 후 사용 가능한 도구:

| 도구 | 설명 |
|------|------|
| `rmbg.remove_background` | 이미지 배경 제거 |
| `rmbg.list_models` | 사용 가능한 모델 목록 |

## 사용 예시

```
# 배경 제거 (Claude에서)
이 이미지의 배경을 제거해줘: [이미지 경로]
```

## 리소스 사용량

| 모드 | VRAM/RAM | 첫 추론 |
|------|----------|---------|
| CPU | ~1GB RAM | ~5초 |
| GPU | ~1GB VRAM | ~1초 |

- 모델 로딩: 첫 요청 시 ~10-30초
- 이후 캐시됨: ~1초 (GPU), ~3초 (CPU)

## 트러블슈팅

### Python 버전 문제
Python 3.10+ 필요. 버전 확인:
```bash
python --version
```

### CUDA 사용 안됨
PyTorch CUDA 버전 확인:
```bash
python -c "import torch; print(torch.cuda.is_available())"
```

### 모델 다운로드 느림
Hugging Face 토큰 설정 (선택):
```bash
# tools/remove-bg-server/.env
HF_TOKEN=your_token_here
```
