---
description: Run slot game simulation for RTP verification
argument-hint: <game-code>
---

# Run Simulation

Execute a slot game simulation to verify RTP and game balance.

## Instructions

1. **Locate simulation file**
   - Standard: `games/src/slot_simulation/{game_code}.ts` (1M spins default)
   - Instant Bonus: `{game_code}_ib.ts` (100K spins default)
   - Special Bonus: `{game_code}_sb.ts` (100K spins default)

2. **Execute simulation**
   ```bash
   cd games
   ts-node src/slot_simulation/{game_code}.ts -- -s 1000000 -t 10
   ```
   - `-s`: Spin count (default 1M, use 100M for precise RTP)
   - `-r`: RTP index (0=LOW, 1=MID, 2=HIGH)
   - `-t`: Thread count (recommend 10)
   - `-e`: Extra bet index

3. **Interpret results**
   ```
   RTP :
   total       base        free        bonusonbase bonusonfree
   0.960123    0.421234    0.312456    0.112345    0.114088
   ```
   - `total`: Must be within target RTP +/- 0.5%
   - Check bonus trigger rates match design intent

4. **Report findings**
   - RTP vs target comparison
   - Bonus trigger frequency
   - Any anomalies or concerns
