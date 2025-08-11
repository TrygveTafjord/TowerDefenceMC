# UppaalTD: A Formal Tower Defense Game

## Project Overview

This project is an assignment for the "Formal Methods for Concurrent and Real-Time Systems" course (A.Y. 2024 - 2025). The goal is to model a Tower Defense game as a Network of Timed Automata using the Uppaal model checker. The primary objective is to formally verify the correctness of the game's specifications, analyze game properties, and determine winning or losing conditions for specific turret configurations.

This README focuses on the **"Vanilla Version"** of the game, which involves a single wave of enemies with deterministic timing for movement and attacks.

## Game Description

The game is played on a 16x8 grid map. The player must defend a **Main Tower** from a wave of enemies by placing defensive **Turrets** on designated locations.

### Core Components:

* **Main Tower**:
    * **Health**: 10 HP.
    * **Goal**: The player loses if the Main Tower's health drops to 0 or below.

* **Enemies**: A wave consists of **3 Circles** and **3 Squares**.
    * They spawn at position (0,0) and move along a predefined path towards the Main Tower.
    * At branching points, their path choice is non-deterministic.
    * Upon reaching the Main Tower, they attack once and then leave the map.
    * **Stats**:

| Type   | Speed (delay) | Health | Damage | Spawn Delay |
| :----- | :------------ | :----- | :----- | :---------- |
| Circle | 1             | 10     | 2      | 2           |
| Square | 3             | 20     | 4      | 3           |

* **Turrets**: Can be placed on designated green cells.
    * They target the closest enemy within their range.
    * **Targeting Priority**: Turrets always prioritize attacking **Squares** over **Circles** if both are in range.
    * **Stats**:

| Type   | Range | Fire Speed (delay) | Damage |
| :----- | :---- | :----------------- | :----- |
| Basic  | 2     | 2                  | 2      |
| Cannon | 1     | 7                  | 5      |
| Sniper | 4     | 20                 | 8      |

## Model Implementation in Uppaal

The system is modeled as a network of three concurrent timed automata templates. Global variables and constants in the `declaration` section define the game parameters, map path, and entity stats.

### Automata Templates:

1.  **`GameExecution`**: Manages the global game state. It initializes the game, sets up enemy properties, and transitions to a `Victory` or `Loss` state based on the outcome (all enemies defeated or Main Tower destroyed).

2.  **`EnemyManager`**: Controls the behavior of all enemies.
    * **Spawning**: Spawns Circles and Squares at their specified time intervals.
    * **Movement**: Updates enemy positions along the predefined `path` array according to their speed.
    * **Path Choice**: Handles non-deterministic choices at crossroads.
    * **Attacking**: Manages enemy attacks on the Main Tower.

3.  **`TurretManager`**: Controls the behavior of all turrets.
    * **Targeting Logic**: Identifies which enemies are in range and selects a target based on proximity and the Square-priority rule.
    * **Shooting**: Inflicts damage on the selected target enemy.
    * **Cooldown**: Enforces the `FireSpeed` delay between shots for each turret.

## Verification and Analysis (Vanilla Version)

The `ModelCheckerTD.xml` file includes a set of TCTL queries used to verify the game's properties.

### Key Properties Verified:

* **Deadlock Freedom**: `A[] not deadlock` ensures the game never gets stuck in a state where time cannot progress.
* **Enemy Path Correctness**: `A[] forall (...) enemy_pos[i] < nCells` verifies that living enemies always remain on the defined map path.
* **Reachability (No Turrets)**: Queries like `E<> (path[enemy_pos[0]][0] == 15 and ...)` confirm that it is possible for each enemy to reach the Main Tower when no turrets are present.
* **Timing Constraints (No Turrets)**: Queries like `A[] ((... imply glob_clock <= 28))` verify the maximum time it takes for each enemy to reach the Main Tower.
* **Winning/Losing Conditions**: The query `E<> game.Victory` is used to check if there is at least one execution path that leads to a win for the player with a given turret configuration. The model is pre-configured with the winning setup from the homework assignment.

## How to Run the Model

### Prerequisites:

* The **Uppaal IDE** must be installed.

### Instructions:

1.  Open the `ModelCheckerTD.xml` file in Uppaal.
2.  The model is loaded with the "Vanilla Version" parameters. The default turret configuration is the winning one specified in property (VI) of the assignment.
3.  **To change the turret configuration**:
    * Navigate to the `declarations` section of the **`TurretManager`** template.
    * Comment out the active `turret_type` and `turret_pos` arrays.
    * Uncomment the desired configuration from the provided examples or create a new one.
4.  Switch to the **Verifier** tab in Uppaal.
5.  The predefined queries for the vanilla version analysis are listed. Select a query and click **Check** to run the verification.
