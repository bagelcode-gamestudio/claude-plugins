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
