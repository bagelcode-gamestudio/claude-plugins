---
name: prefab-check
description: Unity Prefab 수정 시 Override와 Missing Reference를 확인하는 가이드. .prefab 파일을 수정할 때 자동으로 사용됨.
---

# Prefab Check 스킬

Unity Prefab 작업 시 Override와 Missing Reference를 확인합니다.

## 언제 사용하나요?

- Prefab을 수정했을 때
- Prefab Instance를 수정했을 때
- 씬에서 Prefab 관련 작업 후

---

## 1. Prefab Override 확인

### 수정 후 확인 사항

| # | 상황 | 필요 동작 |
|---|------|-----------|
| 1 | Prefab Asset 직접 수정 | 저장 확인 |
| 2 | Instance에서 수정 | Apply 또는 Revert 결정 |
| 3 | 중첩 Prefab 수정 | 어느 레벨에 적용할지 결정 |

### Apply vs Revert 가이드

```markdown
## Prefab Override 발견

수정된 속성:
- Transform.position
- BoxCollider.size

선택:
A) Apply to Prefab - 모든 인스턴스에 적용
B) Revert - 원래 Prefab 값으로 복구
C) Keep Override - 이 인스턴스만 유지

권장: [상황에 따라]
```

---

## 2. Missing Reference 검사

### 검사 대상

| # | 항목 | 위험도 |
|---|------|--------|
| 1 | Script Reference | 🔴 높음 |
| 2 | Material Reference | 🟡 중간 |
| 3 | Texture Reference | 🟡 중간 |
| 4 | Audio Clip | 🟢 낮음 |

### 발견 시 동작

```markdown
⚠️ Missing Reference 발견

Prefab: `Assets/Prefabs/Player.prefab`

Missing 항목:
| # | 컴포넌트 | 필드 | 상태 |
|---|----------|------|------|
| 1 | PlayerController | targetMaterial | Missing |
| 2 | AudioSource | clip | Missing |

해결 방법:
1. 해당 에셋이 삭제되었는지 확인
2. 경로가 변경되었으면 재연결
3. 의도적 제거면 필드 정리
```

---

## 3. Prefab 저장 확인

### 저장 명령

```
unity.project.saveAll {}
```

### 저장 확인 체크리스트

- [ ] Prefab Asset 저장됨
- [ ] 씬 저장됨 (Instance 변경 시)
- [ ] Override 상태 정리됨

---

## 4. 자동 Retro 트리거

### Missing Reference 발견 시

```markdown
💡 RETRO: Missing Reference
- 문제: [어떤 Reference가 Missing인지]
- 원인: [에셋 삭제/이동/이름 변경]
- 방지책: [에셋 이동 시 확인 절차]
```

### Override 충돌 발생 시

```markdown
💡 RETRO: Prefab Override 충돌
- 문제: [여러 인스턴스에서 다른 Override]
- 원인: [일관성 없는 수정]
- 방지책: [Prefab 수정 정책 명확화]
```

---

## 5. 리포트 형식

작업 완료 후:

```markdown
## ✅ Prefab 검증 완료

| 항목 | 상태 |
|------|------|
| Prefab | `Assets/Prefabs/Player.prefab` |
| Override | 정리됨 |
| Missing Ref | 0건 |
| 저장 | ✅ 완료 |

변경 사항:
- BoxCollider.size 수정
- 새 컴포넌트 추가
```
