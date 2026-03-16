# Dev Plugin Conventions

## Skill

```yaml
---
description: "한 줄 설명 (영문)"
user-invocable: false
globs:
  - "트리거할 파일 패턴"
---
```

- `user-invocable: false` 필수 — 슬래시 메뉴에 노출되지 않도록
- `globs`는 최소한으로 — 불필요한 파일에서 트리거되지 않도록
- 파일 위치: `skills/{skill-name}/SKILL.md`

## Command

```yaml
---
description: 한 줄 설명
argument-hint: <인자 힌트>
---
```

- **Skill Set 섹션** 포함 — 세션에서 참조할 스킬을 명시
- **Self-review checklist** 포함
- 파일 위치: `commands/{command-name}.md`

## Versioning

커밋 시 아래 두 파일을 함께 업데이트할 것:

1. **`plugin.json`** — `version` 필드 업데이트 (semver)
   - patch: 버그 수정, 오타, 소소한 내용 변경
   - minor: 스킬/커맨드 추가·삭제, 기능 변경
   - major: 구조 변경, 호환성 깨지는 변경
2. **`CHANGELOG.md`** — 버전 헤더 추가 후 변경 내용 기록
   ```markdown
   ## x.y.z — YYYY-MM-DD
   변경 내용
   ```
