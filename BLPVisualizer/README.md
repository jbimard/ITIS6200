# Bell-LaPadula (BLP) Access Control Visualizer

A command-line simulator of the Bell-LaPadula confidentiality model, with 18 predefined test scenarios.

## The Bell-LaPadula model

Bell-LaPadula is a formal access control model for **confidentiality**. Subjects (users or processes) hold clearances, objects (files) carry classifications, and every access is checked by comparing the two. Two rules apply:

- **No Read Up (Simple Security Property):** a subject cannot read an object classified above its clearance.
- **No Write Down (\*-Property):** a subject cannot write to an object classified below its current level, so high-level information cannot leak downward.

### How the simulator applies the rules

Each subject has a fixed **maximum level** and a **current level**.

- **Read:** allowed if the object level is at or below the current level. If it is above the current level but within the maximum, the read is allowed and the current level is raised to the object's level. If it is above the maximum, the read is denied (**No Read Up**).
- **Write:** allowed if the current level is at or below the object level. Otherwise it is denied (**No Write Down**).
- **Set level:** a subject may raise its current level up to its maximum. Lowering the level is denied, because it would bypass No Write Down.

## Security levels

| Level | Name | Rank |
|-------|------|------|
| `U`  | Unclassified | 0 (lowest) |
| `C`  | Classified   | 1 |
| `S`  | Secret       | 2 |
| `TS` | Top Secret   | 3 (highest) |

## Initial configuration

Subjects:

| Subject | Max level | Starting level |
|---------|-----------|----------------|
| `alice` | S | U |
| `bob`   | C | C |
| `eve`   | U | U |

Objects:

| Object | Level |
|--------|-------|
| `pub.txt`      | U  |
| `emails.txt`   | C  |
| `username.txt` | S  |
| `password.txt` | TS |

Every test case starts from this fresh state.

## Running the simulator

Requires Python 3 and no third-party packages.

```bash
cd BLPVisualizer
python3 Cases.py
```

At the prompt, enter a case number (`1`-`18`), `A` to run all cases in sequence, or `Q` to quit. Each case prints the actions, the ALLOW/DENY decisions, and the final state.

## Test scenarios

| # | Scenario | Result |
|---|----------|--------|
| 1 | Alice reads `emails.txt` | ALLOW (level raised to C) |
| 2 | Alice reads `password.txt` | DENY (No Read Up) |
| 3 | Eve reads `pub.txt` | ALLOW |
| 4 | Eve reads `emails.txt` | DENY (No Read Up) |
| 5 | Bob reads `password.txt` | DENY (No Read Up) |
| 6 | Alice reads `emails.txt`, writes `pub.txt` | ALLOW, DENY (No Write Down) |
| 7 | Alice reads `emails.txt`, writes `password.txt` | ALLOW, ALLOW (write up) |
| 8 | Alice reads `emails.txt`, writes `emails.txt`, reads `username.txt`, writes `emails.txt` | ALLOW, ALLOW, ALLOW, DENY (No Write Down) |
| 9 | Alice reads `username.txt`, writes `emails.txt`, reads `password.txt`, writes `password.txt` | ALLOW, DENY (No Write Down), DENY (No Read Up), ALLOW |
| 10 | Alice reads `pub.txt`, writes `emails.txt`; Bob reads `emails.txt` | ALLOW, ALLOW, ALLOW |
| 11 | Alice reads `pub.txt`, writes `username.txt`; Bob reads `username.txt` | ALLOW, ALLOW, DENY (No Read Up) |
| 12 | Alice reads `pub.txt`, writes `password.txt`; Bob reads `password.txt` | ALLOW, ALLOW, DENY (No Read Up) |
| 13 | Alice reads `pub.txt`, writes `emails.txt`; Eve reads `emails.txt` | ALLOW, ALLOW, DENY (No Read Up) |
| 14 | Alice reads `emails.txt`, writes `pub.txt`; Eve reads `pub.txt` | ALLOW, DENY (No Write Down), ALLOW |
| 15 | Alice sets level to S, reads `username.txt` | ALLOW, ALLOW |
| 16 | Alice reads `emails.txt`, sets level to U, writes `pub.txt`; Eve reads `pub.txt` | ALLOW, DENY (cannot lower), DENY (No Write Down), ALLOW |
| 17 | Alice reads `username.txt`, sets level to C, writes `emails.txt`; Eve reads `emails.txt` | ALLOW, DENY (cannot lower), DENY (No Write Down), DENY (No Read Up) |
| 18 | Eve reads `pub.txt`, then reads `emails.txt` | ALLOW, DENY (No Read Up) |

## Files

- `BLP.py`: the `BLP` class (subjects, objects, read, write, set level, state display)
- `Cases.py`: the 18 scenarios and the interactive command-line menu
