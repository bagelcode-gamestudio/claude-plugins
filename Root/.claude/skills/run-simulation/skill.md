---
name: run-simulation
description: 게임 시뮬레이션 실행 및 검증 가이드. 서버 로직 수정 후 검증, RTP 확인 시 자동으로 사용됨.
trigger: 시뮬레이션 돌려, 검증해줘, RTP 확인, 서버 로직 테스트 관련 작업 시
priority: high
---

# 게임 시뮬레이션 실행

서버 작업 후 게임 로직을 시뮬레이션으로 검증합니다.

## 사전 질문 (필수)

시뮬레이션 실행 전 반드시 사용자에게 질문:

> **시뮬레이션 목적이 무엇인가요?**
> - A) RTP 검증 (정확한 수치 확인) → 1억 스핀 권장
> - B) 디버깅 (로그 보면서 동작 확인) → 적은 스핀으로 실행

## 파일 타입별 기본 스핀 횟수

| 파일 타입 | 기본 스핀 | 설명 |
|-----------|----------|------|
| `{code}_ib.ts` | 10만 | Instant Bonus 시뮬레이션 |
| `{code}_sb.ts` | 10만 | Special Bonus 시뮬레이션 |
| `{code}.ts` | 100만 | 일반 시뮬레이션 |
| RTP 정밀 검증 | **1억** | 정확한 RTP 수치 필요시 |

## 시뮬레이션 실행

### 기본 실행

```bash
cd /Users/kimminkyu/Bagelcode/Repository_Bagelcode/club-vegas-slots/games

# 일반 시뮬레이션 (100만 스핀, 10스레드)
ts-node src/slot_simulation/{game_code}.ts -- -t 10

# Instant Bonus 시뮬레이션 (10만 스핀, 10스레드)
ts-node src/slot_simulation/{game_code}_ib.ts -- -s 100000 -t 10

# RTP 정밀 검증 (1억 스핀, 10스레드)
ts-node src/slot_simulation/{game_code}.ts -- -s 100000000 -t 10
```

### 파라미터 설명

| 파라미터 | 설명 | 기본값 |
|----------|------|--------|
| `-s` | 시뮬레이션 스핀 횟수 | 1,000,000 |
| `-r` | RTP 인덱스 (0=LOW, 1=MID, 2=HIGH) | 0 |
| `-b` | 베팅 스케일 | 1 |
| `-t` | 스레드 수 | **10** (권장) |
| `-e` | Extra Bet 인덱스 | 0 |
| `-v` | Volatility 모드 | false |

### 예시 명령어

```bash
# 디버깅용 - 로그 확인하며 적은 스핀 (스레드도 적게)
ts-node src/slot_simulation/bbd.ts -- -s 10000 -t 2

# 일반 검증 - 100만 스핀, 10스레드
ts-node src/slot_simulation/bbd.ts -- -s 1000000 -t 10

# RTP 정밀 검증 - 1억 스핀, 10스레드 (시간 오래 걸림)
ts-node src/slot_simulation/bbd.ts -- -s 100000000 -t 10

# Instant Bonus 검증 - 10스레드
ts-node src/slot_simulation/bbd_ib.ts -- -s 100000 -t 10
```

## 게임 코드 찾기

```bash
ls games/src/slot_simulation/*.ts | head -20
```

---

## BBD 슈퍼보너스 시뮬레이션 (`bbd_sb.ts`)

### 실행 방법

```bash
cd /Users/kimminkyu/Bagelcode/Repository_Bagelcode/club-vegas-slots/games

# Level 1 슈퍼보너스 (기본)
ts-node src/slot_simulation/bbd_sb.ts -- -s 100000 -t 10

# Level 2 슈퍼보너스
ts-node src/slot_simulation/bbd_sb.ts -- -s 100000 -t 10 --level=2

# Level 3 슈퍼보너스
ts-node src/slot_simulation/bbd_sb.ts -- -s 100000 -t 10 --level=3

# 로그 출력 활성화 (디버깅용)
ts-node src/slot_simulation/bbd_sb.ts -- -s 1000 -t 2 --log=1
```

### 추가 파라미터

| 파라미터 | 설명 | 기본값 |
|----------|------|--------|
| `--level` | 슈퍼보너스 레벨 (1, 2, 3) | 1 |
| `--log` | 로그 출력 (0=끔, 1=켬) | 0 |

### 분석 항목

#### 1. 프리게임 통계 (Free Game Stats)

| 항목 | 설명 |
|------|------|
| Free Game Count | 슈퍼 프리게임 진입 횟수 |
| Free Game Odds | 베이스 스핀 대비 프리게임 확률 |
| Avg Win (x) | 평균 당첨 배수 |
| Std | 표준편차 |
| Min/Max | 최소/최대 당첨 배수 |
| Percentiles | P50, P90, P95, P99 백분위수 |
| Avg Total Spins | 평균 총 스핀 수 (초기 + 리트리거) |
| Super Infinity Triggered | Super Infinity 발동 횟수/비율 |

#### 2. 프리게임 당첨 분포 (Win Distribution)

```
<1    <2    <5    <10   <20   <50   <100  <200  <500  <1000 <2000 <5000 <Infinity
0.05  0.10  0.25  0.30  0.15  0.08  0.04  0.02  0.01  0.00  0.00  0.00  0.00
```

각 구간에 해당하는 당첨 비율을 표시합니다.

#### 3. Super Infinity 통계

| 항목 | 설명 |
|------|------|
| Trigger Count | Super Infinity 발동 횟수 |
| Avg Final Multiplier | 평균 최종 배수 |
| Avg Slashes | 평균 슬래시(스핀) 횟수 |
| Avg Lives Used | 평균 라이프 소모량 |
| Avg Lives Gained | 평균 라이프 획득량 |

#### 4. 심볼 타입 분포 (Symbol Type Distribution)

Super Infinity 중 등장하는 심볼 비중:

| 심볼 타입 | 설명 |
|-----------|------|
| `blank` | 빈 칸 (당첨 없음) |
| `cash` | 현금 심볼 |
| `wheel` | 휠 심볼 (잭팟/배수 결정) |
| `life` | 라이프 심볼 (+1 라이프) |
| `golden_cash` | 골든 현금 (고배율) |
| `golden_wheel` | 골든 휠 (고배율 잭팟) |

**출력 예시:**
```
Symbol Type Distribution:
  Total Symbols: 125000
  blank: 50000 (40.0000%)
  cash: 35000 (28.0000%)
  wheel: 20000 (16.0000%)
  life: 8000 (6.4000%)
  golden_cash: 7000 (5.6000%)
  golden_wheel: 5000 (4.0000%)
  Golden Total: 12000 (20.0000% of non-blank)
```

#### 5. Level 3 클러스터 사이즈 분포

Level 3에서만 등장하는 대형 심볼(2xN) 통계:

| 사이즈 | 설명 |
|--------|------|
| 2x1 | 2열 1행 클러스터 |
| 2x2 | 2열 2행 클러스터 |
| 3x3 | 3열 3행 클러스터 |

**출력 예시:**
```
Level 3 Cluster Size Distribution (2xN):
  2x1: 5000 (50.0000%), Golden: 500 (10.0000%)
  2x2: 3500 (35.0000%), Golden: 350 (10.0000%)
  3x3: 1500 (15.0000%), Golden: 300 (20.0000%)
```

#### 6. Super Infinity 배수 분포

최종 배수의 구간별 분포:

```
<1    <2    <5    <10   <20   <50   <100  <200  <500  <1000 <2000 <5000 <Infinity
0.10  0.20  0.30  0.20  0.10  0.05  0.03  0.01  0.005 0.002 0.001 0.001 0.001
```

#### 7. 레벨별 상세 통계 (Level Breakdown)

각 레벨(1, 2, 3)별로 분리된 통계:

```
Level 1:
  Free Games: 10000, Avg Win (x): 25.123456
  Super Infinity: 8000, Avg Mult: 15.500, Avg Slashes: 12.300
  Lives - Used: 5.200, Gained: 2.100
  Symbols - Golden: 10.50%, Life: 8.20% (of non-blank)

Level 2:
  ...

Level 3:
  ...
```

#### 8. 잭팟 통계

슈퍼 프리게임 중 잭팟 발생 통계:

| 항목 | 설명 |
|------|------|
| Jackpot Count | 잭팟 발생 횟수 |
| Credit | 총 잭팟 크레딧 |
| Odds (per FS) | 프리스핀당 잭팟 확률 |

### 분석 활용 가이드

#### 심볼 밸런스 확인

```
Golden 비율이 너무 높으면 → 고배율 당첨 빈도 증가
Life 비율이 너무 낮으면 → Super Infinity 지속시간 감소
Blank 비율이 너무 높으면 → 전체 배수 하락
```

#### 휠 결과 분석

- `wheel` + `golden_wheel` 비율 = 잭팟/배수 보너스 빈도
- Golden wheel 비율이 높을수록 고배율 잭팟 확률 상승

#### 레벨별 기대값 비교

| 레벨 | 특징 |
|------|------|
| Level 1 | 기본 심볼, 낮은 배수, 안정적 |
| Level 2 | 중간 배수, 골든 심볼 증가 |
| Level 3 | 클러스터 심볼, 고배율, 변동성 높음 |

---

## 결과 해석

### RTP 출력 예시

```
===============
RTP :
total       base        free        bonusonbase bonusonfree
0.960123    0.421234    0.312456    0.112345    0.114088
===============
```

| 항목 | 의미 |
|------|------|
| total | 전체 RTP (목표: 0.96) |
| base | 베이스 게임 RTP |
| free | 프리스핀 RTP |
| bonusonbase | 베이스에서 보너스 RTP |
| bonusonfree | 프리스핀에서 보너스 RTP |

### 스핀 횟수별 RTP 정확도

| 스핀 횟수 | 정확도 | 용도 |
|-----------|--------|------|
| 1만 | 낮음 | 디버깅, 로그 확인 |
| 100만 | 중간 | 일반 검증, 대략적 확인 |
| 1억 | 높음 | RTP 정밀 검증 |

## 트러블슈팅

### 메모리 부족

스레드 수를 줄여서 실행 (10 → 4 또는 2):

```bash
# 메모리 여유 있으면 4스레드
ts-node src/slot_simulation/bbd.ts -- -s 1000000 -t 4

# 그래도 부족하면 2스레드
ts-node src/slot_simulation/bbd.ts -- -s 1000000 -t 2
```

### 타입 에러

ts-node가 타입 에러를 보여주면 해당 파일 수정 후 재실행.

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `game-mapping` | 게임 ID/코드 매핑 |
