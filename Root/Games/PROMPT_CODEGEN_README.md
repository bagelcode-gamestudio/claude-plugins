# 프롬프트 기반 서버코드 자동생성 시스템

베이글코드 게임 스튜디오의 프롬프트 마크다운을 활용한 자동 서버코드 생성 솔루션입니다.

## 📋 개요

이 시스템은 프롬프트 마크다운(`.prompt.md`) 파일에 정의된 스펙을 기반으로 Express.js TypeScript 서버 코드를 자동으로 생성합니다.

### 주요 기능

- 🚀 **프롬프트 기반 코드 생성**: 마크다운 형태의 프롬프트로 서버 코드 자동 생성
- 🎯 **TypeScript 지원**: 타입 안전한 코드 생성
- 🏗️ **모듈화된 구조**: Controller, Service, Model 계층 분리
- 🔒 **인증/인가 시스템**: JWT 기반 인증 미들웨어 지원
- 📊 **API 문서화**: Swagger/OpenAPI 문서 자동 생성
- 🧪 **테스트 코드**: 단위 테스트 코드 자동 생성
- 🐳 **Docker 지원**: 컨테이너화된 배포 설정

## 🛠️ 설치 및 설정

### 필수 요구사항

- Node.js 16.0.0 이상
- TypeScript 5.0.0 이상
- Redis (데이터베이스)

### 의존성 설치

```bash
npm install
```

## 🚀 사용법

### 1. 프롬프트 파일 작성

`.github/prompts/` 디렉토리에 `.prompt.md` 파일을 작성합니다.

```markdown
---
mode: edit
description: "게임 API 서버 생성"
---

# 게임 관리 API

## Configuration
\`\`\`yaml
framework: express
language: typescript
database: redis
authentication: jwt
middleware: ["cors", "compression", "bodyParser"]
outputDir: ./src/generated/game-api
\`\`\`

## Requirements

게임 관리를 위한 RESTful API 서버를 생성합니다.

### API 엔드포인트 스펙
\`\`\`typescript
endpoints:
  - path: "/api/games"
    method: GET
    description: "게임 목록 조회"
    authentication: true
\`\`\`
```

### 2. 코드 생성 실행

#### CLI 사용
```bash
# 코드 생성
npx ts-node src/prompt-codegen/cli.ts generate -p .github/prompts/examples/game-api.prompt.md

# 프롬프트 목록 조회
npx ts-node src/prompt-codegen/cli.ts list -d .github/prompts

# 프롬프트 검증
npx ts-node src/prompt-codegen/cli.ts validate -p .github/prompts/examples/game-api.prompt.md
```

#### 프로그래매틱 사용
```typescript
import { PromptCodeGenManager } from './src/prompt-codegen';

const manager = new PromptCodeGenManager();
const result = await manager.generateFromPrompt('./path/to/prompt.md');

if (result.success) {
  console.log('생성 완료!', result.metadata.generatedFiles);
} else {
  console.error('생성 실패:', result.errors);
}
```

### 3. 생성된 코드 구조

```
src/generated/game-api/
├── controllers/          # API 컨트롤러
│   ├── GameController.ts
│   └── index.ts
├── services/            # 비즈니스 로직
│   ├── GameService.ts
│   └── index.ts
├── models/             # 데이터 모델
│   ├── Game.ts
│   └── index.ts
├── routes/             # 라우터 설정
│   ├── gameRoutes.ts
│   └── index.ts
├── middleware/         # 미들웨어
│   ├── auth.ts
│   ├── validation.ts
│   └── index.ts
├── types/             # TypeScript 타입
│   ├── game.types.ts
│   └── index.ts
└── tests/             # 테스트 코드
    ├── controllers/
    ├── services/
    └── routes/
```

## 📝 프롬프트 작성 가이드

### 메타데이터 설정

```yaml
---
mode: edit                    # edit, ask, agent 중 선택
description: "API 서버 생성"   # 프롬프트 설명
---
```

### 설정 섹션

```yaml
framework: express           # express, fastify, koa
language: typescript        # typescript, javascript
database: redis             # redis, mongodb, mysql, postgresql
authentication: jwt         # jwt, session, oauth, none
middleware: ["cors", "compression"]
outputDir: ./src/generated
testing: true
documentation: true
```

### API 엔드포인트 정의

```typescript
endpoints:
  - path: "/api/games"
    method: GET
    description: "게임 목록 조회"
    authentication: true
    parameters:
      - name: "page"
        type: "number"
        required: false
        description: "페이지 번호"
    response:
      type: "object"
      schema:
        games: "Game[]"
        total: "number"
```

### 데이터 모델 정의

```typescript
models:
  - name: "Game"
    properties:
      id:
        type: "string"
        required: true
        description: "게임 ID"
      name:
        type: "string"
        required: true
        description: "게임 이름"
```

### 서비스 레이어 정의

```typescript
services:
  - name: "GameService"
    description: "게임 비즈니스 로직"
    methods:
      - name: "getGames"
        parameters: ["page: number", "limit: number"]
        returnType: "Promise<Game[]>"
        description: "게임 목록 조회"
        async: true
```

## 🧪 테스트

### 테스트 실행

```bash
# 전체 테스트 실행
npm test

# 프롬프트 코드 생성 테스트
npx ts-node test-prompt-codegen.ts
```

### 생성된 코드 테스트

생성된 코드는 자동으로 테스트 파일도 함께 생성됩니다:

```bash
# 생성된 테스트 실행
cd src/generated/game-api
npm test
```

## 📚 예제

### 기본 게임 API 서버

```bash
# 예제 프롬프트로 코드 생성
npx ts-node src/prompt-codegen/cli.ts generate -p .github/prompts/examples/game-api.prompt.md
```

이 예제는 다음 기능을 포함한 완전한 게임 API 서버를 생성합니다:

- 게임 CRUD 작업
- JWT 인증
- Redis 데이터 저장
- 입력 검증
- 에러 핸들링
- API 문서화

### 커스텀 프롬프트 작성

자신만의 프롬프트를 작성하려면:

1. `.github/prompts/` 디렉토리에 새 `.prompt.md` 파일 생성
2. 메타데이터, 설정, 요구사항 섹션 작성
3. API 스펙, 모델, 서비스 정의
4. CLI로 코드 생성 실행

## 🔧 고급 사용법

### 커스텀 템플릿

기본 템플릿 외에 커스텀 템플릿을 사용할 수 있습니다:

```typescript
import { CodeGenerator } from './src/prompt-codegen';

const generator = new CodeGenerator(config);
// 커스텀 템플릿 적용
```

### 플러그인 시스템

다양한 프레임워크와 데이터베이스를 지원하는 플러그인을 개발할 수 있습니다:

```typescript
// 커스텀 플러그인 예제
class FastifyPlugin {
  generateController(spec: EndpointSpec): string {
    // Fastify 전용 컨트롤러 생성
  }
}
```

## 🤝 기여하기

1. 이 저장소를 포크합니다
2. 기능 브랜치를 생성합니다 (`git checkout -b feature/amazing-feature`)
3. 변경사항을 커밋합니다 (`git commit -m 'Add amazing feature'`)
4. 브랜치에 푸시합니다 (`git push origin feature/amazing-feature`)
5. Pull Request를 열어주세요

## 📄 라이선스

이 프로젝트는 ISC 라이선스 하에 배포됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

## 🏢 베이글코드 게임 스튜디오

베이글코드 게임 스튜디오는 혁신적인 게임 개발 도구와 솔루션을 제공하는 회사입니다.

- 웹사이트: [bagelcode.com](https://bagelcode.com)
- 이메일: contact@bagelcode.com

---

**Made with ❤️ by Bagelcode Game Studio**
