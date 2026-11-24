# HW6 submission


## Part A - Calculate on paper

### The MDP in my own words

- **States:** the cells where the agent can stand and the game continues: (1,1), (1,2), (1,3), (2,1), (2,3), (3,1), (3,2), (3,3), (3,4). G and H are terminal cells. The wall (2,2) is not a state.
- **Actions:** up, right, down, left.
- **Transitions:** on S, ordinary floor and mud, the agent goes exactly in the chosen direction. On ice, it goes in the chosen direction with probability 0.8 and in each side direction with probability 0.1. If the agent hits a wall or the edge, it stays in place.
- **Rewards:** the agent gets the reward of the tile where the move ends: floor, S and ice = -1, mud = -3, goal = +10, hole = -10.
- **Goal of the agent:** get the biggest expected sum of discounted rewards (discount 0.9). So it is good to reach G quickly and avoid H.

### V1 (V0 = 0)

```
 -1   -1   10    0 (G)
 -1    #   -1    0 (H)
 -1   -1   -1   -1
```

Ice tile (2,3), probability-weighted sums:
- up: 0.8(-1) + 0.1(-1) + 0.1(-10) = -1.9
- right: 0.8(-10) + 0.1(-1) + 0.1(-1) = -8.2
- down: 0.8(-1) + 0.1(-1) + 0.1(-10) = -1.9
- left: 0.8(-1) + 0.1(-1) + 0.1(-1) = -1.0

The best is left, so V1(2,3) = -1.

### V2 (using only V1)

Formula: Q = sum of P x (R + 0.9 x V1(next)).

**(1,2) mud:**
- up (stay in mud): -3 + 0.9(-1) = -3.9
- right to (1,3): -1 + 0.9(10) = 8
- down (wall, stay): -3.9
- left to (1,1): -1 + 0.9(-1) = -1.9

V2(1,2) = 8. Best action: right.

**(1,3):**
- up (edge, stay): -1 + 0.9(10) = 8
- right to G: 10 + 0.9(0) = 10
- down to (2,3): -1 + 0.9(-1) = -1.9
- left to (1,2): -3 + 0.9(-1) = -3.9

V2(1,3) = 10. Best action: right.

**(2,3) ice:**
- up: 0.8(8) + 0.1(-1.9) + 0.1(-10) = 5.21
- right: 0.8(-10) + 0.1(8) + 0.1(-1.9) = -7.39
- down: 0.8(-1.9) + 0.1(-1.9) + 0.1(-10) = -2.71
- left: 0.8(-1.9) + 0.1(8) + 0.1(-1.9) = -0.91

V2(2,3) = 5.21. Best action: up.

### Best-action arrows

(1,2) right, (1,3) right, (2,3) up. There are no ties.
→ → → G
↑ # ↑ H
→ → ↑ ←

### Why two rounds are not enough

V2 only looks two steps ahead, and the values are not final yet. Information about the goal has not reached most cells. After more rounds the best actions can change, especially on ice, where the risk of the hole must be compared with the cost of a longer way. We can say the policy is optimal only after the values have converged.

---

## Part B - FrozenLake simulations

**Environment:** `FrozenLake-v1`, `map_name="4x4"`, default rewards, 100-step limit.

**Fixed policy** (one action for every non-terminal state, chosen before the runs and never changed):
`0:Right, 1:Right, 2:Down, 3:Left, 4:Up, 6:Down, 8:Right, 9:Right, 10:Down, 13:Right, 14:Right`.
The route is 0 -> 1 -> 2 -> 6 -> 10 -> 14 -> 15. It goes around the holes and has 6 steps.

**Files:** one saved episode in `outputs/episode_render.txt`, all episodes in `outputs/episodes.csv`, batch summary in `outputs/summary.csv`.

| Slippery? | Policy | Batch seed | Goals /50 | Holes /50 | Time limits /50 | Success % | Mean steps |
|---|---|---|---|---|---|---|---|
| False | random | 11 | 0 | 50 | 0 | 0 | 7.98 |
| False | random | 22 | 0 | 50 | 0 | 0 | 6.42 |
| False | random | 33 | 1 | 49 | 0 | 2 | 6.66 |
| False | fixed | 11 | 50 | 0 | 0 | 100 | 6.00 |
| False | fixed | 22 | 50 | 0 | 0 | 100 | 6.00 |
| False | fixed | 33 | 50 | 0 | 0 | 100 | 6.00 |
| True | random | 11 | 0 | 50 | 0 | 0 | 8.30 |
| True | random | 22 | 2 | 48 | 0 | 4 | 7.76 |
| True | random | 33 | 0 | 50 | 0 | 0 | 6.74 |
| True | fixed | 11 | 0 | 50 | 0 | 0 | 4.84 |
| True | fixed | 22 | 1 | 49 | 0 | 2 | 5.04 |
| True | fixed | 33 | 0 | 50 | 0 | 0 | 5.60 |

### What slipping changed

Without slipping, the fixed policy reached the goal in all 150 episodes (100%), because every move does exactly what we choose. With slipping, the agent moves in the chosen direction only with probability 1/3 and often moves sideways. My route goes next to holes, so the agent usually falls in a hole. Only 1 of 150 fixed-policy episodes succeeded with slipping. The random policy is bad in both cases (1 of 150 without slipping, 2 of 150 with slipping). There were no time-limit episodes in any condition, because every episode ended in a hole or in the goal before 100 steps.

### How consistent the batches were

Without slipping, the fixed policy gave exactly the same result in all three batches (50 goals, 6 steps), because there is no randomness. In the other conditions, the results changed from batch to batch (for example, random policy with slipping gave 0, 2 and 0 goals). The numbers of successes are very small, so a change of one or two episodes changes the result a lot.

### Why one successful episode is not enough

With slipping, the result is random. One success can be just luck, and one failure can be just bad luck. We need many episodes and several batches to see how often a policy really succeeds.

### Example traces

Successful episode (fixed policy, no slipping): `[paste from outputs/example_traces.txt]`

Failed episode (fixed policy, with slipping): `[paste from outputs/example_traces.txt]`

### Why this is not training

The policies are fixed before the runs and they are never changed using the results. The simulations only measure how good a policy is. The agent does not learn anything from the episodes.

---

## Part C - Solver and extra floors

The floor rules are stored in `floors.py` as a table (reward, terminal, passable, slip). The value iteration code in `gridworld.py` does not mention any floor type. To add a new floor, I only add one new line to the table.

### Checks (saved in `outputs/part_c_log.txt`)

All four checks passed:
- Walls and map edges leave the agent in the same place.
- Ice probabilities are 0.8, 0.1, 0.1 and the sum is 1.
- Mud = -3, sand = -2, water = -5.
- There are no transitions or rewards after G or H.

### Original map: comparison with my paper work

My code gives V1(2,3) = -1 and V1(1,3) = 10. It gives V2(1,2) = 8, V2(1,3) = 10, V2(2,3) = 5.21. This is the same as my paper work, so there were no discrepancies to correct.

### Convergence

- Original map: 12 iterations, final change 2.17e-07.
- Extended map: 11 iterations, final change 2.41e-07.
- The stopping limit was 0.000001 and the safeguard of 10,000 iterations was not reached.

### Tie-breaking

If two actions have the same value (difference smaller than 1e-9), I choose the first one in this order: up, right, down, left. No ties were found on either map.

### Final values and policies

Original map (`.M.G / .#IH / S...`):

```
Values:
  4.200   8.000  10.000   0.000
  2.780    #      5.824   0.000
  1.536   2.818   4.242   2.818

Policy:
→ → → G
↑ # ↑ H
→ → ↑ ←
```

Extended map (`.M.G / .#IH / SD.W`):

```
Values:
  4.200   8.000  10.000   0.000
  2.780    #      5.824   0.000
  1.502   2.818   4.242   2.818

Policy:
→ → → G
↑ # ↑ H
↑ → ↑ ←
```

### Simulation of the computed policies

3 batches of 50 episodes, 100-step limit, same seeds as Part B (1000 x batch seed + i). Discounted return = r0 + 0.9 r1 + 0.9^2 r2 + ...

| Map | Batch seed | Goals | Holes | Time limits | Mean steps | Mean discounted return |
|---|---|---|---|---|---|---|
| original | 11 | 42 | 8 | 0 | 4.98 | 0.891 |
| original | 22 | 46 | 4 | 0 | 5.04 | 1.915 |
| original | 33 | 48 | 2 | 0 | 5.04 | 2.492 |
| extended | 11 | 50 | 0 | 0 | 5.00 | 1.502 |
| extended | 22 | 50 | 0 | 0 | 5.00 | 1.502 |
| extended | 33 | 50 | 0 | 0 | 5.00 | 1.502 |

### Simulation compared with the infinite-horizon values

V(start) is the expected discounted return when the agent can play for an unlimited number of steps. A simulation is different: it has a limited number of episodes and a 100-step limit, so it is only an estimate.

- Extended map: there is no randomness (no ice on the path), so every episode has the same return, 1.502. This is equal to V(start) = 1.502.
- Original map: V(start) = 1.536. The average return of the three batches is about 1.77. It is close but not equal, because the result depends on luck at the ice tile, and 50 episodes is a small sample. The batch averages are quite different (0.891, 1.915, 2.492).

### Comparison of the two maps

- **Values:** only one final value changed: V(3,1) went from 1.536 to 1.502. All other values are the same on both maps.
- **Actions:** only one action changed. At the start cell (3,1), the policy goes right on the original map and up on the extended map.
- **Where the floor costs mattered:** on the extended map, cell (3,2) is sand (-2). Going right from the start now has value -2 + 0.9 x 2.818 = 0.536, which is worse than going up (-1 + 0.9 x 2.78 = 1.502). So the policy goes up through (2,1), (1,1), the mud cell (1,2) and (1,3) to the goal. The mud costs -3, but this way is still better. The water cell (3,4) is never visited by the policy.
- **How the policy handles ice:** on the original map, the policy goes through the ice cell (2,3) and moves up. From there, there is a 10% chance to slip right into the hole each try. This gives about 0.1 / 0.9 = 11% chance of ending in the hole. In the simulation, 14 of 150 episodes ended in the hole (9.3%), which is close to this number. On the extended map, the policy avoids the ice completely, so there were 0 holes in 150 episodes.

