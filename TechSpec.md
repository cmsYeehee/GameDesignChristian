## Game engine:
MelonJS: No youtube videos! Simple, but there's just not enough information on this online

BabylonJS: While it does have 3d features, it looks way too difficult to actually be able to find anything.

Phaser: Seems like it can do 2.5d. It looks simple to use, and has various examples on it's website of how to use it. There are a bunch of youtube videos on it, and they wouldn't take too long to understand these videos to start coding. 

## Architecture: 
1. GameManager
Manages the overall game state, including level progression, score, and player interactions.

Variables

currentLevel: int - Tracks the current level.
score: int - Player's total score.
coins: int - Currency earned for upgrades.
isGameOver: boolean - Indicates if the game is over. If the game is over, it would be caused by the player losing the level
isGameFinished: boolean - Indicates if the user has completed the level. True if defenseBuildings[0], the capital core, has no health left. 
aiVehicles: List<Vehicle>  - List of AI-controlled vehicles (friendly). All vehicles except isSelected vehicle (check variables for vehicles) and dead vehicles
map: Map - The game map instance.
selectedVehicle: Vehicle - The vehicle currently under player's control.
allVehicles: List<Vehicle> - List of all vehicles.
deploymentZone: Zone - Area where the player can deploy vehicles.
## will have to make a class for this
defenseBuildings: list<DefensiveStructure> - A list of the defensive structures. Defense structure [0] is the capital core.
## implement the townhall style feature
gameTime: float - Timer for level objectives.
isPaused: boolean - Game pause state.
Methods

- initializeGame()
Behavior: Sets up a new game with default settings, initializes the map, vehicles, and UI elements.
- startLevel(int levelNumber)
Behavior: Loads the specified level, including map layout, enemy placements, and available vehicles. Adds all defensive buildings from the level into the defenseBuildings variable. 
- endLevel()
Behavior: If isGameFinsihed is true, calculates score (calling updateScore method + gameTime, which also has an effect on score), awards coins(equal to the score), and checks for level completion conditions. 
- updateScore(int points, list<DefenseiveStructure> defenseBuildings)
Behavior: Adds points to score based on destruction of a specific building. Has to call defenseBuildings as points are awarded based on destruction of buildings.
- deployVehicle(Vehicle vehicle, Point position)
## will I need a point class
Behavior: Places a vehicle in the deployment zone at the specified position.
- switchControl(int vehicleId)
Behavior: Changes selectedVehicle to the vehicle with the given ID.
- togglePause()
Behavior: Toggles the game's paused state.
- changeAIBehavior(String AIPlayStyle, Vehicle vehicle)  
Behavior: Change the playstyle of a specific vehicle (ground or air); More information on what the play styles are in the Vehicle abstract class.
- tickGamePlay(float deltaTime)
Behavior: Updates game logic, including vehicle movements and AI behaviors.
- handleInput(InputEvent event)
Behavior: Processes player input and directs it to the appropriate vehicle or game component.
- checkGameOver()
Output: boolean - If true, and allVehicles health are = 0, end game. initializeGame() to allow user to restart level.


2. Map
Represents the game environment, including terrain, buildings, and defensive structures.

Variables

terrainGrid: 2D Array<TerrainTile> - Represents the terrain layout. Some of the 2D array should be "walkable" by land. Some of it should be nonwalkable by land.
## would we not need a class for terrtainTile?
buildings: List<Building> - List of all buildings on the map.
defensiveStructures: List<DefensiveStructure> - Turrets, air defenses, and capital cores.
spawnPoints: List<Point> - Locations where enemy units spawn.
## don't we need a point class then?
deployZone: Zone - Area where the player can deploy vehicles.
Methods

- loadMap(int levelNumber)
Behavior: Loads terrain, buildings, and defensive structures for the specified level.
- isPassable(Point position, VehicleType type)
Output: boolean - Returns true if the vehicle type can pass through the position.
- destroyBuilding(Building building)
Behavior: "Destroys" a building from the map, meaning it can now be walked over by vehicles and updates the game state and method isPassable at that point.
- getNearestEnemy(Point position)
Output: Defensive structure - Returns the nearest enemy object to the given position.
- getNearestBuilding(Point position)
Output: Nearest Building; Returns the nearest enemy building to the given position. 
- getPathToCapital(Point position, capitcalCore core)
Output: 2D array providng directions to capital core in ground vehicle. Will have to use stack/queue to ensure that it is the quickest path to the capital.
- getPathToUser(Point postion)
Output: 2D array providng directions to user in ground vehicle. Will have to use stack/queue to ensure that it is the quickest path to the capital.

3. Vehicle (Abstract Class)
Base class for all vehicles, including ground and air units.

Variables

id: int - Unique identifier for the vehicle.
position: Point - Current position.
direction: Vector - Current movement direction.
health: float - Current health points.
maxHealth: float - Maximum health points.
living: boolean - determines if vehicle still has health or not
speed: float - Movement speed.
isSelected: boolean - If the vehicle is currently under player control.
energy: float - Current energy level for weapons and shields.
maxEnergy: float - Maximum energy capacity.
energyForShield: float - Currently energy allocated for shields
energyForWeapons: float - currently energy allocated for weapons.
weapon: Weapon - The equipped weapon.
shield: Shield - Shield component.
type: String - type of vehicle (air or ground)
AIplayStyle - "String": User will choose a playstyle for ground/air vehicles. Options: 
 - "Aggressive" - move straight to the capital core to try and destroy it. 
 - "Destructive" - move to nearest building to destroy it.
 - "Offensive" - move to nearest defense tower to destroy it.
 - "Follower" - follow the user controlled 

Methods

- move(Vector direction, float deltaTime)
Behavior: Moves the vehicle in the specified direction, considering speed and collisions. Has to call isPassable class to determine if the vehicle can continue to move in a direction.
- canAttack (energyCost)
Behavior: Returns true is energyCost < energyForWeapons. Returns false otherwise.
- attack(Vector direction)
Behavior: Fires the weapon in the current direction. Reduce energyForWeapons by energyCost amount. 
- selfDestruct()
Behavior: Causes the vehicle to explode, dealing area damage. Calls the destructiveVehicles class and buildings class to damage their buildings accordingly.
- redistributePower(float energyForShield, float energyForWeapons)
Behavior: Allocates energy between weapons and shields. I
- takeDamage(float amount)
Behavior: Reduces health by the damage amount after shield mitigation. If only shield takes damage, only reduce shield health.
- update(float deltaTime)
Behavior: Updates the vehicle's state each frame.
- remove()
Behavior: If living is false, remove the vehicle from map.



4. GroundVehicle (Inherits Vehicle)
Represents ground-based vehicles.



Methods

playerMove(Vector direction, float deltaTime, boolean isSelected)
- Behavior: Overrides base move to include terrain modifiers. If the isSelected is false, will call the AIMove method in AIGroundVehicle

5. AirVehicle (Inherits Vehicle)

Methods

playerMove(Vector direction, float deltaTime, boolean isSelected)
- Behavior: Overrides base move to include terrain modifiers. If the isSelected is false, will call the AIMove method in AIAirVehicle


6. AIGroundVehicle (Inherits Vehicle)
Represents AI ground-based vehicles
variables:
AIplayStyle - "String": User will choose a playstyle for ground/air vehicles. Options: 
 - "Aggressive" - move straight to the capital core to try and destroy it. 
 - "Destructive" - move to nearest building to destroy it.
 - "Offensive" - move to nearest defense tower to destroy it.
 - "Follower" - follow the user controlled 

Methods:
- AIMove(Vector direction, float deltaTime, boolean isSelected)
Behavior: Has to call the AIplayStyle method. If Aggressive, call getPathToCapital. If destructive, call getNearestBuilding. If Offensive call getNearestenemy. If follower, follow movement of user. If they are in an air vehicle and user isn't, call: getPathToUser. 

7. AIAirVehicle (Inherits Vehicle)
AIplayStyle - "String": User will choose a playstyle for ground/air vehicles. Options: 
 - "Aggressive" - move straight to the capital core to try and destroy it. 
 - "Destructive" - move to nearest building to destroy it.
 - "Offensive" - move to nearest defense tower to destroy it.
 - "Follower" - follow the user controlled 

Methods:
- AIMove(Vector direction, float deltaTime, boolean isSelected)
Behavior: Has to call the AIplayStyle method. If Aggressive, call getPathToCapital. If destructive, call getNearestBuilding. If Offensive call getNearestenemy. If follower, follow movement of user. If they are in an air vehicle and user isn't, call: getPathToUser. 
8. Weapon
Represents weapons equipped by vehicles.

Variables

damage: float - Damage per shot.
fireRate: float - Shots per second.
range: float - Maximum effective range.
energyCost: float - Energy consumed per shot.
projectileType: Projectile - Type of projectile fired.



9. DefensiveStructure
Represents defensive buildings like turrets and air defenses.

Variables

position: Point - Location on the map.
health: float - Current health points.
weapon: Weapon - Equipped weapon.
targetingRange: float - Detection range for enemies.
isActive: boolean - Indicates if the structure is operational.
Methods

scanForTargets()
Output: boolean - determine whether or not there is a target in range
Behavior: Detects enemy vehicles within range.
attackTarget()
Behavior: Fires at the closest target within range. 
takeDamage(float amount)
Behavior: Reduces health and checks for destruction.
10. Building
Represents non-defensive structures that can be destroyed.

Variables

position: Point
health: float
isDestroyed: boolean
Methods

takeDamage(float amount)
Behavior: Reduces health and sets isDestroyed if health drops to zero.
11. CapitalCore (Inherits DefensiveStructure)
Represents the main objective with stronger defenses.

Variables

shield: Shield - Enhanced shielding.
Methods


takeDamage(float amount)
Behavior: Reduces health and checks for destruction.
12. Projectile
Represents projectiles fired by weapons.

Variables

position: Point
direction: Vector
speed: float
damage: float
originTeam: Team
Methods

move(float deltaTime)
Behavior: Moves the projectile forward.
checkCollision()
Behavior: Checks for collisions with vehicles or structures, and checks if collision is with the other team.
13. UpgradeSystem
Handles vehicle upgrades and progression.

Variables

availableUpgrades: List<Upgrade> - Upgrades the player can purchase.
purchasedUpgrades: List<Upgrade> - Upgrades the player has acquired.
Methods

- purchaseUpgrade(Upgrade upgrade)
Behavior: Buys an upgrade if the player has enough coins. 
- applyUpgrade(Vehicle vehicle, Upgrade upgrade)
Behavior: Applies the upgrade effects to the specified vehicle. Will update a variable in the vehicle class like rate of fire, shield cost, etc. (listed in revisedDesign.md). 
getAvailableUpgrades()
Output: List<Upgrade> - Returns a list of upgrades that can be purchased.
14. Upgrade
Represents an individual upgrade.

Variables

name: String
description: String
cost: int
upgradeType: enum { Offensive, Defensive, Utility }
effect: UpgradeEffect
Methods

15. UserInterface
Manages all UI elements, including HUD, menus, and in-game notifications.

Variables

hud: HUD - Displays health, energy, and selected vehicle info.
mainMenu: Menu
settingsMenu: Menu
scoreScreen: Screen
Methods

- updateHUD()
Behavior: Refreshes HUD elements based on game state.
- showMenu(Menu menu)
Behavior: Displays the specified menu.
- displayNotification(String message)
Behavior: Shows in-game messages to the player.
16. SoundManager
Handles all audio in the game.

Variables

backgroundMusic: AudioTrack
soundEffects: Dictionary<String, AudioClip>
Methods

- playMusic(String trackName)
Behavior: Starts playing background music.
- playSoundEffect(String effectName)
Behavior: Plays a sound effect.
- stopMusic()
Behavior: Stops the background music.
17. InputManager
Processes player input and sends commands to the GameManager.

Methods
- processInput()
Behavior: Reads input devices and triggers corresponding actions.
- bindKey(String action, KeyCode key)
Behavior: Allows customization of key bindings.

18. Team (Enum)
Defines the team affiliations.

Values:
Player
Enemy

20. Point
Represents a coordinate in 2D space.

Variables

x: float - The X-coordinate.
y: float - The Y-coordinate.
Methods

- distanceTo(Point other): float
Behavior: Calculates the Euclidean distance to another point.
- add(Vector vector): Point
Behavior: Returns a new Point by adding a Vector to the current point.
- equals(Point other): boolean
Behavior: Checks if two points have the same coordinates.
21. Vector
Represents a direction and magnitude in 2D space.

Variables

dx: float - Change in the X-coordinate.
dy: float - Change in the Y-coordinate.
Methods

- magnitude(): float
Behavior: Calculates the magnitude (length) of the vector.
- normalize(): Vector
Behavior: Returns a unit vector (vector of length 1) in the same direction.
- scale(float scalar): Vector
Input: scalar - The factor by which to scale the vector.
Behavior: Returns a new Vector scaled by the given factor.
22. Zone
Defines a rectangular area on the map, such as the deployment zone.

Variables

topLeft: Point - The top-left corner of the zone.
bottomRight: Point - The bottom-right corner of the zone.
Methods

- contains(Point point): boolean
Behavior: Determines if a given point lies within the zone.
- getAllPoints(): List<Point>
Behavior: Returns a list of all points within the zone (useful for certain calculations).
23. TerrainType (Enum)
Categorizes different types of terrain on the map.

Values
GRASS - Standard terrain, passable by ground vehicles.
WATER - Impassable terrain for ground vehicles.
MOUNTAIN - Impassable terrain.
ROAD - Terrain that may provide speed bonuses to ground vehicles.
BUILDING - Occupied space by buildings; impassable until destroyed.
DEPLOYMENT_ZONE - Area where players can deploy vehicles.
UNWALKABLE - General impassable terrain.
24. TerrainTile
Represents an individual tile in the terrain grid.

Variables

type: TerrainType - The type of terrain for this tile.
isPassable: boolean - Indicates if the tile can be traversed by ground vehicles.
Methods

- determinePassability(Vehicle vehicle): boolean
Input: vehicle - Within the vehicle you would need to know the type of vehicle, so is it a ground or an air vehicle. this will change which variables can cross the path. 
Behavior: Determines if the tile is passable by the given vehicle type.
- setTerrainType(TerrainType newType)
Behavior: Changes the terrain type of the tile and updates isPassable accordingly.
- getTerrainEffects(Vehicle vehicle): TerrainEffects
Behavior: Returns any effects the terrain has on the vehicle (e.g., speed modifiers).
