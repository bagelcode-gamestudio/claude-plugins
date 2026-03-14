---
name: unity-csharp
description: Unity C# 코딩 패턴과 컨벤션 가이드. .cs 파일을 작성하거나 Unity 스크립트를 수정할 때 자동으로 사용됨.
trigger: .cs 파일 작성/수정 시 자동 트리거 (unity-compile과 함께)
priority: high
---

# Unity C# 코딩 패턴 스킬

Unity C# 스크립트 작성 시 반드시 따라야 할 패턴과 컨벤션입니다.

> **필수**: 이 스킬의 규칙은 Unity C# 코드 품질을 위해 **생략 불가**입니다.

## 트리거 조건

| 조건 | 설명 |
|------|------|
| MonoBehaviour 작성 | 컴포넌트 스크립트 |
| ScriptableObject 작성 | 데이터 에셋 |
| Editor 스크립트 작성 | 커스텀 에디터 |
| 기존 Unity C# 수정 | 리팩토링, 버그 수정 |

---

## 1. RequireComponent + OnValidate 패턴

### 규칙

`[RequireComponent]`로 필수 컴포넌트를 지정했다면, **Awake에서 매번 GetComponent를 호출하지 말고** OnValidate에서 캐싱하세요.

### 나쁜 예

```csharp
// Awake에서 매번 GetComponent - 비효율적
[RequireComponent(typeof(Camera))]
public class UICamera : MonoBehaviour
{
    private Camera _camera;

    void Awake() => _camera = GetComponent<Camera>();
}
```

### 좋은 예

```csharp
[RequireComponent(typeof(Camera))]
public class UICamera : MonoBehaviour
{
    [SerializeField, HideInInspector]
    private Camera _camera;

    void OnValidate()
    {
        if (_camera == null)
            _camera = GetComponent<Camera>();
    }

    void Awake()
    {
        // 런타임 생성 시 fallback (OnValidate 안 불림)
        if (_camera == null)
            _camera = GetComponent<Camera>();
    }
}
```

### 이유

| 방식 | Editor 시점 | Runtime 시점 | 직렬화 |
|------|-------------|--------------|--------|
| Awake만 | X | O (매번 호출) | X |
| OnValidate + Awake | O (저장됨) | O (fallback) | O |

---

## 2. Unity fake null 주의

### 규칙

Unity 오브젝트는 `==` 연산자가 오버라이드되어 **destroyed 상태**를 감지합니다.
`?.` (null-conditional) 연산자는 C# null만 체크하므로 **사용 금지**.

### 나쁜 예

```csharp
// ?. 연산자는 destroyed 상태 감지 못함
public static Camera? Camera => _instance?._camera;

// ?. 체이닝도 위험
var name = gameObject?.GetComponent<Enemy>()?.name;
```

### 좋은 예

```csharp
// != null로 명시적 체크
public static Camera? Camera => _instance != null ? _instance._camera : null;

// 명시적 null 체크
if (gameObject != null)
{
    var enemy = gameObject.GetComponent<Enemy>();
    if (enemy != null)
    {
        var name = enemy.name;
    }
}
```

### 이유

| 상태 | `== null` | `?.` (C# null) | 실제 |
|------|-----------|----------------|------|
| null | true | null 반환 | 정상 |
| destroyed | true | **값 반환** | 위험 |
| 정상 | false | 값 반환 | 정상 |

---

## 3. SerializeField 가시성

### 규칙

Inspector에서 수정할 필드는 `[SerializeField] private`를 사용하세요.
`public` 필드는 API로 노출할 의도가 있을 때만 사용합니다.

### 나쁜 예

```csharp
public class Player : MonoBehaviour
{
    public float speed = 5f;        // 불필요한 public 노출
    public GameObject target;       // 외부에서 수정 가능
}
```

### 좋은 예

```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private float _speed = 5f;
    [SerializeField] private GameObject _target;

    // 필요시 읽기 전용 프로퍼티로 노출
    public float Speed => _speed;
}
```

---

## 4. 컴포넌트 캐싱

### 규칙

자주 접근하는 컴포넌트는 캐싱하세요. Update에서 GetComponent 호출 금지.

### 나쁜 예

```csharp
void Update()
{
    // 매 프레임 GetComponent - 성능 저하
    GetComponent<Rigidbody>().AddForce(Vector3.up);
}
```

### 좋은 예

```csharp
private Rigidbody _rb;

void Awake() => _rb = GetComponent<Rigidbody>();

void Update()
{
    _rb.AddForce(Vector3.up);
}
```

---

## 5. Coroutine 관리

### 규칙

Coroutine 참조를 저장하고, 중복 실행을 방지하세요.

### 나쁜 예

```csharp
void OnEnable()
{
    StartCoroutine(DoSomething());  // 중복 실행 가능
}
```

### 좋은 예

```csharp
private Coroutine _routine;

void OnEnable()
{
    if (_routine != null) StopCoroutine(_routine);
    _routine = StartCoroutine(DoSomething());
}

void OnDisable()
{
    if (_routine != null)
    {
        StopCoroutine(_routine);
        _routine = null;
    }
}
```

---

## 체크리스트

코드 작성 완료 후 확인:

- [ ] RequireComponent 사용 시 OnValidate 패턴 적용
- [ ] Unity 오브젝트에 `?.` 대신 `!= null` 사용
- [ ] private 필드에 `[SerializeField]` 사용 (public 지양)
- [ ] Update에서 GetComponent 호출 없음
- [ ] Coroutine 참조 관리

---

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `unity-compile` | 컴파일 확인 절차 (.cs 수정 후 필수) |
| `prefab-check` | Prefab Override/Missing 확인 |
