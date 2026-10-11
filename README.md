EMPIRE COMMANDERS

Software Architecture and Implementation Specification

ROLE

Act as a senior Python game engineer, software architect, gameplay programmer, and QA engineer.

Design and implement a complete, playable 2D real-time strategy and army-command game named Empire Commanders.

Do not merely explain how to build the game. Create the actual project files, implement the game systems, run available tests, fix discovered issues, and provide clear instructions for launching the finished application.

The game must be written in Python using Pygame.

All player-facing text, menus, descriptions, buttons, tooltips, messages, and documentation must use English.

1. PROJECT OBJECTIVES

Create a top-down 2D medieval strategy game in which the player directly controls a general while commanding an army, developing a village, recruiting troops, fighting enemy kingdoms, and capturing territory.

The game should combine:

- Direct character control.
- Tactical army commands.
- Autonomous soldier AI.
- Real-time combat.
- Village management.
- Resource gathering.
- Troop recruitment and progression.
- Enemy kingdom simulation.
- Territory expansion.
- Campaign progression.
- Save and load functionality.

The initial release must be a functional desktop single-player game.

Do not implement LAN multiplayer, internet multiplayer, or Android packaging in the initial milestone. Keep the architecture modular enough that these features can be added later.

2. TECHNOLOGY STACK

Required:

- Python 3.
- Pygame.
- Python standard library.
- JSON for campaign saves and configuration where appropriate.
- unittest or another lightweight testing framework.

Avoid unnecessary dependencies.

Do not introduce a database, backend server, ECS framework, or complex third-party architecture unless there is a demonstrated need.

Use a fixed or controlled simulation timestep where appropriate, with rendering separated from game-state updates.

Target an initial resolution of 1280 × 720 while supporting window resizing and reasonable UI scaling.

Use a target of 60 FPS on ordinary desktop hardware, but do not make gameplay simulation dependent on achieving exactly 60 FPS.

3. REQUIRED PROJECT STRUCTURE

Create a maintainable project with a structure similar to the following:

empire_commanders/
main.py
requirements.txt
README.md
settings.json

game/
    __init__.py
    game.py
    game_state.py
    game_loop.py
    camera.py
    input_manager.py
    event_bus.py

entities/
    __init__.py
    entity.py
    general.py
    soldier.py
    enemy.py
    projectile.py

systems/
    __init__.py
    movement_system.py
    pathfinding.py
    combat_system.py
    targeting_system.py
    army_command_system.py
    formation_system.py
    ai_system.py
    resource_system.py
    building_system.py
    recruitment_system.py
    progression_system.py
    world_system.py
    save_system.py

world/
    __init__.py
    battlefield.py
    world_map.py
    terrain.py
    decorations.py
    settlement.py
    kingdom.py

ui/
    __init__.py
    main_menu.py
    hud.py
    command_panel.py
    village_panel.py
    recruitment_panel.py
    pause_menu.py
    settings_menu.py
    widgets.py

data/
    troops.json
    buildings.json
    resources.json
    difficulties.json
    world_generation.json

assets/
    images/
    fonts/
    audio/

tests/
    test_combat.py
    test_army_commands.py
    test_resources.py
    test_buildings.py
    test_recruitment.py
    test_save_system.py

This is a proposed organization, not a requirement to create empty files for every component.

Create modules only when they have meaningful responsibilities. Avoid unnecessary fragmentation.

If a simpler structure is more maintainable for the first release, explain the change and implement it consistently.

4. ARCHITECTURAL PRINCIPLES

Separate game simulation, rendering, input handling, user interface, data, and persistence.

Use the following rules:

1. Entities store their state and relevant behavior.
2. Systems implement operations that affect multiple entities.
3. Rendering reads game state but does not change gameplay state.
4. UI buttons issue commands rather than directly manipulating unrelated systems.
5. Input handling translates keyboard and mouse actions into game commands.
6. Configuration and balance values must not be scattered throughout the code.
7. Game systems must communicate through clear interfaces.
8. Use dependency injection or explicit references where it improves testability.
9. Avoid circular imports and global mutable state.
10. Do not put the entire game into one enormous Python file.
11. Do not create a class for every trivial value.
12. Keep the code understandable for a developer maintaining the project.

Use type hints for important public interfaces and dataclasses where they improve clarity.

5. GAME LOOP

Implement a controlled game loop with the following responsibilities:

1. Read input events.
2. Update the current game mode.
3. Update movement and AI.
4. Resolve combat.
5. Update resources and buildings when applicable.
6. Process defeat and victory conditions.
7. Render the world.
8. Render the user interface.
9. Maintain a reasonable frame rate.

Use delta time or a fixed simulation timestep so movement and combat do not change drastically on faster computers.

Prevent large frame-time spikes from causing units to teleport or receive excessive updates.

Pause menus must stop simulation updates while keeping the interface responsive.

6. GENERAL CONTROL

The player controls one general directly on the battlefield.

Controls:

- WASD or arrow keys: movement.
- Space: melee attack.
- Mouse: select targets and destinations.
- Right-click: issue a movement order.
- F: follow order.
- H: hold-position order.
- C: charge order.
- Escape: pause menu.

The general must have:

- Health.
- Attack damage.
- Attack range.
- Attack cooldown.
- Movement speed.
- Equipment progression.
- Unlockable skills.
- Defeat and recovery rules.

The general must not attack faster merely because the computer has a higher frame rate.

Use clear attack feedback and prevent the attack input from applying unlimited damage every frame.

7. ARMY AND SOLDIER SYSTEM

A new campaign starts with:

- One general.
- Four basic soldiers.
- One village.
- Limited starting resources.
- A nearby enemy presence.

Each soldier must have:

- Unique identity within the current campaign.
- Team or faction.
- Health and maximum health.
- Attack damage.
- Attack range.
- Attack cooldown.
- Movement speed.
- Unit class.
- Current order.
- Current target.
- Current position.
- Living, defeated, or other clearly defined state.

Do not store redundant state that can be reliably derived from other data.

Implement basic separation so allied soldiers do not continuously overlap.

Avoid recalculating expensive pathfinding unnecessarily.

8. ARMY COMMAND SYSTEM

Implement the following functional commands:

- Follow General.
- Move to Position.
- Hold Position.
- Charge.
- Attack Target.
- Circle Defense.
- Surround Target.
- Flank Target.
- Retreat.
- Stand Down.
- Shield Wall.
- Archer Position.

Commands must affect soldier behavior.

Each soldier must have an explicit order state, such as:
FOLLOW, MOVE, HOLD, CHARGE, ATTACK, RETREAT, or FORMATION.

Implement order transitions carefully.

Example:

- FOLLOW causes soldiers to occupy spaced positions near the general.
- MOVE causes soldiers to travel to assigned destinations.
- HOLD keeps soldiers near their assigned positions while allowing self-defense against nearby enemies.
- CHARGE causes soldiers to advance toward enemies.
- RETREAT causes soldiers to move away from danger toward a defined rally point.
- FORMATION assigns individual formation positions.

Do not claim advanced formations are complete if they merely cause soldiers to move toward the same point.

For formations, assign separate destination positions around a center point.

Implement at least the core commands completely before implementing the more complex ones.

9. MOVEMENT AND PATHFINDING

Implement movement that works on a battlefield containing obstacles.

Use a practical navigation representation, such as a grid-based map with A* pathfinding.

Requirements:

- Distinguish walkable and blocked terrain.
- Prevent units from intentionally navigating through solid obstacles.
- Recalculate paths when necessary.
- Avoid running full pathfinding every frame.
- Allow paths to be invalidated when obstacles or destinations change.
- Use simple local avoidance for nearby units where practical.
- Prevent endless pathfinding loops.
- Provide a fallback when no path is available.

Decorative trees and rocks may be nonblocking if that makes the first version simpler, but the behavior must be consistent and documented.

The initial battlefield must remain playable even if advanced obstacle avoidance is incomplete.

10. COMBAT SYSTEM

Use a centralized combat system to validate and resolve attacks.

A successful attack must check:

- Attacker is alive.
- Target is alive.
- Attacker is permitted to attack.
- Target is in range.
- Attack cooldown has expired.

Apply damage once per valid attack.

Combat must support melee attacks initially, with the architecture allowing ranged attacks later.

Include:

- Health bars.
- Attack cooldowns.
- Damage feedback.
- Defeat handling.
- Target selection.
- Basic target prioritization.

Do not allow defeated units to continue attacking or receiving commands.

Ensure that killing a unit updates all relevant systems correctly.

11. TROOP CLASSES AND PROGRESSION

Implement the following troop classes:

1. Bolo Fighter: inexpensive basic melee unit.
2. Spearman: infantry with a longer attack range.
3. Archer: ranged unit.
4. Shield Guard: defensive infantry.
5. Cavalry: fast unit.
6. Elite Warrior: advanced infantry.
7. Spartan-inspired Elite: expensive late-game warrior.

Troop definitions must be data-driven using JSON or another simple configuration format.

Each class must have configurable:

- Health.
- Damage.
- Attack range.
- Attack cooldown.
- Movement speed.
- Recruitment cost.
- Unlock requirements.
- Class-specific behavior where needed.

Use original fictional designs and do not depend on copyrighted assets.

12. ARMY CAPACITY AND RECRUITMENT

Track these separately:

- Current troop count.
- Maximum troop capacity.
- Number of available recruitment slots.

Every army-capacity upgrade increases maximum capacity by exactly two.

The cost of subsequent capacity upgrades must increase according to a configurable formula or table.

A capacity upgrade must never recruit soldiers automatically.

Recruitment must:

1. Check whether the unit type is unlocked.
2. Check available army capacity.
3. Check resource requirements.
4. Deduct resources only if recruitment succeeds.
5. Create the unit only after validation.
6. Display a useful error message if recruitment fails.

Recruitment and capacity upgrades must be tested for edge cases.

13. VILLAGE ECONOMY

Implement four resources:

- Wood.
- Stone.
- Food.
- Gold.

Create buildings:

- Town Center.
- Farm.
- Lumber Camp.
- Quarry.
- Barracks.
- Archery Range.
- Stable.
- Defensive Tower.
- Training Grounds.
- Walls.

Buildings must have configurable:

- Construction cost.
- Upgrade cost.
- Construction or upgrade time if implemented.
- Level.
- Resource output or gameplay effect.
- Unlock requirements.

Initial requirements:

- Farms produce food.
- Lumber Camps produce wood.
- Quarries produce stone.
- The Town Center unlocks village development.
- Barracks unlock infantry recruitment.
- Archery Ranges unlock archers.
- Stables unlock cavalry.
- Defensive Towers support settlement defense.
- Training Grounds improve eligible troops.

Resources must be awarded according to controlled simulation time, not rendering frame count.

Prevent resource values from becoming negative.

Prevent invalid or repeated purchases from charging resources incorrectly.

14. WORLD MAP AND TERRITORY

Implement a fictional world containing:

- The player's settlement.
- Neutral settlements.
- Enemy kingdoms.
- Roads.
- Forests.
- Resource locations.
- Strategic territory.

The player must be able to explore and capture settlements.

A captured settlement should provide meaningful benefits.

Keep world-map state separate from the tactical battlefield.

If entering a battle, preserve the relevant campaign state and return to the appropriate strategic state when the battle concludes.

For the initial implementation, a compact world map is acceptable.

Do not attempt to simulate an enormous continent before the basic campaign is stable.

15. NPC KINGDOM AI

Enemy kingdoms should make strategic decisions, including:

- Gathering resources.
- Developing settlements.
- Recruiting troops.
- Defending territory.
- Expanding.
- Attacking rival settlements.
- Responding to threats.

Use a simple, understandable decision system.

For the initial release, strategic decisions can be evaluated periodically rather than every frame.

Difficulties:

- Normal.
- Hard.
- Extreme.
- Impossible.

Difficulty should modify documented factors such as AI decision intervals, starting resources, or strategic aggressiveness.

Do not grant unlimited resources or create unbeatable enemies without clear design justification.

16. MORALE

Implement morale as a later gameplay layer after the core combat system is stable.

Possible morale influences:

- Nearby allies defeated.
- Being surrounded.
- General defeated.
- Successful attacks.
- Winning or losing a battle.

Low morale may cause reduced effectiveness or retreat.

If morale is included, its rules must be consistent and understandable to the player.

Do not let morale calculations override basic unit-state validity.

17. USER INTERFACE

Implement:

- Main menu.
- New Campaign.
- Load Campaign.
- Settings.
- Difficulty selection.
- Battlefield HUD.
- Army command panel.
- Recruitment panel.
- Village construction panel.
- Resource display.
- Pause menu.
- Victory and defeat screens.

Every visible button must perform a real action.

Use clear labels, readable fonts, consistent spacing, and visible interaction feedback.

Display relevant information such as:

- General health.
- Current army count and capacity.
- Resources.
- Current army order.
- Selected target or unit.
- Construction or upgrade costs.
- Difficulty.
- Current objectives.

Support mouse and keyboard controls.

Handle window resizing without making controls inaccessible.

18. VISUALS AND ASSETS

Use an original 2D top-down art style.

If final art assets are unavailable, create clean placeholder graphics using Pygame drawing primitives.

Required visual elements:

- Grass terrain.
- Trees and rocks.
- Player general.
- Allied soldiers.
- Enemy soldiers.
- Settlements and buildings.
- Selection indicators.
- Health bars.
- Combat feedback.
- Resource icons or readable text equivalents.

Do not require paid assets or external downloads for the project to run.

The game must still launch if optional sound files are missing.

19. SAVE AND LOAD

Implement campaign persistence using JSON or another suitable standard-library format.

Save:

- Campaign version.
- General state.
- Army state.
- Resources.
- Army capacity and upgrades.
- Village buildings and levels.
- Captured settlements.
- NPC kingdom state where appropriate.
- Current campaign progression.

Requirements:

- Validate save data before loading.
- Handle missing or malformed files.
- Use a safe fallback rather than crashing.
- Avoid silently overwriting a valid save with corrupted data.
- Include a save-version field for future compatibility.
- Provide meaningful error messages.

Separate serialization from rendering and gameplay logic.

20. TESTING REQUIREMENTS

Create automated tests for:

- Attack range and cooldown.
- Damage application.
- Defeated-unit behavior.
- Army command state changes.
- Capacity upgrades.
- Recruitment resource costs.
- Recruitment capacity restrictions.
- Building construction and upgrades.
- Resource generation.
- Save/load round trips.
- Invalid save handling.
- Difficulty configuration.

Run tests where the execution environment permits.

Also perform basic manual or automated smoke tests:

- Launching the application.
- Starting a new campaign.
- Moving the general.
- Issuing army orders.
- Defeating an enemy.
- Ending a battle.
- Saving and loading progress.

Do not claim a feature has been tested unless the test was actually executed.

21. PERFORMANCE AND RELIABILITY

The game should run smoothly on an ordinary desktop computer.

Requirements:

- Avoid excessive per-frame allocations.
- Avoid unnecessary full-world scans.
- Limit pathfinding work.
- Keep rendering separate from simulation.
- Use a controlled update rate.
- Handle empty armies and defeated generals.
- Prevent actions on invalid or destroyed entities.
- Avoid crashes when no valid target exists.
- Provide useful exceptions or error messages during development.

Do not prematurely optimize code without evidence, but keep the architecture suitable for later expansion.

22. DEVELOPMENT PHASES

Implement the game incrementally.

PHASE 1: Core battlefield

- Window and game loop.
- General movement.
- Four allied soldiers.
- Enemy squad.
- Basic combat.
- Health bars.
- Victory and defeat.

PHASE 2: Tactical systems

- Follow.
- Move.
- Hold.
- Charge.
- Target selection.
- Basic formations.
- Obstacle navigation.

PHASE 3: Village and army progression

- Resources.
- Buildings.
- Recruitment.
- Capacity upgrades.
- Troop classes.

PHASE 4: Campaign and world map

- Settlements.
- Territory capture.
- NPC kingdoms.
- Difficulty settings.
- Campaign objectives.

PHASE 5: Persistence and polish

- Save/load.
- Menus.
- Settings.
- Audio support.
- Visual improvements.
- Bug fixes.

PHASE 6: Verification

- Automated tests.
- Smoke tests.
- Performance checks.
- Installation documentation.
- Final feature and limitation report.

Each phase must produce a runnable build before the next phase begins.

23. EXPLICITLY OUT OF SCOPE FOR THIS RELEASE

Do not implement these features in the initial desktop single-player release:

- LAN multiplayer.
- Internet multiplayer.
- Account systems.
- Online matchmaking.
- Dedicated servers.
- Android APK packaging.
- In-app purchases.
- Unnecessary external services.

Keep future multiplayer and mobile requirements in mind when defining interfaces, but do not create placeholder systems that complicate the initial release.

24. DELIVERABLES

Provide:

1. Complete Python project source.
2. All required assets or procedural replacements.
3. requirements.txt.
4. README.md.
5. Setup and launch instructions.
6. Keyboard and mouse controls.
7. Tests and their actual results.
8. Known limitations.
9. A clear list of completed versus incomplete features.
10. A sensible next-step plan.

If you can access a filesystem and execute Python, create the project files directly, run the tests, launch or smoke-test the application where possible, and fix discovered problems.

If your environment cannot create downloadable project files, provide the files in clearly labeled, complete code blocks without omitting required code.

Do not claim that a complete game exists merely because the architecture is described.

Do not replace implementation with pseudocode.

Do not leave buttons connected to empty functions.

Do not claim multiplayer, Android support, or other deferred features are implemented.

FINAL ACCEPTANCE CRITERIA

The release is acceptable only when:

- The game launches using the documented commands.
- The player can control the general.
- Four allied soldiers can receive and execute core orders.
- Enemies can move, attack, take damage, and be defeated.
- Combat respects range and cooldowns.
- The village economy and recruitment work.
- Army capacity upgrades add exactly two slots.
- Resources and buildings behave consistently.
- A basic campaign can progress.
-
