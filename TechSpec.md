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
deploymentZoness: list<Zone> - Area where the player can deploy vehicles.
defenseBuildings: list<DefensiveStructure> - A list of the defensive structures. Defense structure [0] is the capital core.
buildings: list<buildings> - A list of all the non-defensive building structures
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
- changeAIBehavior(Vehicle vehicle)  
Behavior: Change the playstyle of a specific vehicle (ground or air); More information on what the play styles are in the Vehicle abstract class.
- checkGameOver()
Output: boolean - If true, and allVehicles health are = 0, end game. initializeGame() to allow user to restart level.
- upgradeVehicle (Vehicle vehicle)
Behvaior: Upgrades a type of vehicle for a specific trait. Depends on the upgradeSystem class.


2. Map
Represents the game environment, including terrain, buildings, and defensive structures.

Variables

terrainGrid: 2D Array<Point> - Represents the terrain layout. Some of the 2D array should be "walkable" by land. Some of it should be nonwalkable by land.
buildings: List<Building> - List of all buildings on the map.
defensiveStructures: List<DefensiveStructure> - Turrets, air defenses, and capital cores. defensiveStructure[0] is the capital core
deployZones: List<Zone> - Area where the player can deploy vehicles.
Methods

- loadMap(int levelNumber)
Behavior: Loads terrain, buildings, and defensive structures for the specified level.
- isPassable(Point position, Vehicle vehicle)
Output: boolean - Returns true if the vehicle type can pass through the position.
- destroyBuilding(Building building)
Behavior: "Destroys" a building from the map, meaning it can now be walked over by vehicles and updates terrainTile at that point.
- destroyDefensiveStructure(DefensiveStructure structure)
Behavior: "Destroys" a defensive structure from the map, meaning it can now be walked over by vehicles and updates terrainTile at that point.
- getNearestEnemy(Point position)
Output: Defensive structure - Returns the nearest enemy object to the given position.
- getNearestBuilding(Point position)
Output: Nearest Building; Returns the nearest enemy building to the given position. 
- getPathToCapital(Point position)
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
living: boolean - determines if vehicle still has health or not
speed: float - Movement speed.
isSelected: boolean - If the vehicle is currently under player control.
energy: float - Current energy level for weapons and shields.
energyForShield: float - Currently energy allocated for shields
energyForWeapons: float - currently energy allocated for weapons.
AIplayStyle - enum: User will choose a playstyle for ground/air vehicles. Options: 
 - "Aggressive" - move straight to the capital core to try and destroy it. 
 - "Destructive" - move to nearest building to destroy it.
 - "Offensive" - move to nearest defense tower to destroy it.
 - "Follower" - follow the user controlled 
 damage: float - Damage per shot.
fireRate: float - Shots per second.
range: float - Maximum effective range.
energyCost: float - Energy consumed per shot.
vehicleType: enum - ground Vehicle or air vehicle

Methods

- move(Vector direction, float deltaTime)
Behavior: Moves the vehicle in the specified direction, considering speed and collisions. Has to call isPassable method and determine the type of vehicle to determine if the vehicle can continue to move in a direction. Only works if isSelected is true. 
- canAttack (Point postion)
Behavior: Returns true is energyCost < energyForWeapons and if the building is within range. Returns false otherwise.
- attack(Vector direction)
Behavior: Fires the weapon in the current direction. Reduce energyForWeapons by energyCost amount. 
- selfDestruct()
Behavior: Causes the vehicle to explode, dealing area damage. Calls the destructiveVehicles class and buildings class to damage their buildings accordingly.
- redistributePower(float energyForShield, float energyForWeapons)
Behavior: Allocates energy between weapons and shields. 
- takeDamage(float amount)
Behavior: Reduces health by the damage amount after shield mitigation. If only shield takes damage, only reduce shield health.
- remove()
Behavior: If living is false, remove the vehicle from map.



4. AIVehicle (Inherits Vehicle)
Represents AI ground-based vehicles
variables:
target: Point - The target for the AI vehicle;

Methods:
- findTarget()
Behavior: finds the next target based on the AIplayStyle variable. Returns a target point.
AIMoveAndAttack(Vector direction, float deltaTime)
Behavior: Has to call the AIplayStyle method. If Aggressive, call getPathToCapital. findTarget() should also produce the capital point, and the AI bot will attack once in range. If destructive, call getNearestBuilding. findTarget() should also produce the nearest building, and the AI bot will attack once in range. If Offensive call getNearestenemy. findTarget() should also produce the nearest defensive structure, and the AI bot will attack once in range. If follower, follow movement of user. If they are in an air vehicle and user isn't, call: getPathToUser. Once user has stopped for >= 1 second, find where user is attacking and attack the same building. 

5. DefensiveStructure
Represents defensive buildings like turrets and air defenses.

Variables

position: Point - Location on the map.
health: float - Current health points.
targetingRange: float - Detection range for enemies.
isActive: boolean - Indicates if the structure is operational.
 damage: float - Damage per shot.
fireRate: float - Shots per second.
Methods



scanForTargets()
Output: boolean - determine whether or not there is a target in range
Behavior: Detects enemy vehicles within range.
attackTarget()
Behavior: Fires at the closest target within range. 
takeDamage(float amount)
Behavior: Reduces health and checks for destruction.
6. Building
Represents non-defensive structures that can be destroyed.

Variables

position: Point - location on map
health: float - current health points
isDestroyed: boolean - indicates if structure is destroyed
Methods

takeDamage(float amount)
Behavior: Reduces health and sets isDestroyed if health drops to zero.


7. UpgradeSystem
Handles vehicle upgrades and progression.

Variables

availableUpgrades: List<Upgrade> - Upgrades the player can purchase.
purchasedUpgrades: List<Upgrade> - Upgrades the player has acquired.
Methods

- purchaseUpgrade(Upgrade upgrade)
Behavior: Buys an upgrade if the player has enough coins. 
- applyUpgrade(Vehicle vehicle, Upgrade upgrade)
Behavior: Applies the upgrade effects to the specified vehicle. Will update a variable in the vehicle class like rate of fire, shield cost, etc. (listed in revisedDesign.md) based on the upgradeType
getAvailableUpgrades()
Output: List<Upgrade> - Returns a list of upgrades that can be purchased taking into account the cost of the upgrades.
8. Upgrade
Represents an individual upgrade.

Variables

name: String - name of upgrade
description: String - description of upgrade
cost: int - cost of upgrade
upgradeType: enum { Offensive, Defensive, Utility } - type of upgrade(check what each upgrade does in the revisedDesign.md. Depending on which option they select, they will get a small boost to those factors)
Methods


9. Point
Represents a coordinate in 2D space.

Variables

x: float - The X-coordinate.
y: float - The Y-coordinate.
isPassable: boolean. Is the point passable by a ground vehicle?
Methods

- distanceTo(Point other): float
Behavior: Calculates the Euclidean distance to another point.
- add(Vector vector): Point
Behavior: Returns a new Point by adding a Vector to the current point.
- equals(Point other): boolean
Behavior: Checks if two points have the same coordinates.

10. Vector
Represents a direction and magnitude in 2D space.

Variables

dx: float - Change in the X-coordinate.
dy: float - Change in the Y-coordinate.

Methods

- magnitude(): float
Behavior: Calculates the magnitude (length) of the vector.
- scale(float scalar): Vector
Input: scalar - The factor by which to scale the vector.
Behavior: Returns a new Vector scaled by the given factor.


11. Zone
Defines a rectangular area on the map, such as the deployment zone.

Variables

topLeft: Point - The top-left corner of the zone.
bottomRight: Point - The bottom-right corner of the zone.
isPassable: boolean. Zone is passable if true by a ground vehicle. Always passable by air vehicle.
Methods

- contains(Point point): boolean
Behavior: Determines if a given point lies within the zone.
- getAllPoints(): List<Point>
Behavior: Returns a list of all points within the zone (useful for certain calculations).
- determineIfPassable(): determine if all points in a zone are passable. If true, set isPassable to true. 


12. Projectile
Projectile Represents projectiles fired by weapons.

Variables:
currentPosition: Point - where the projectile is coming from 
endPosition: Point - where the projectile is going to

Methods:

move(float deltaTime) Behavior: Moves the projectile forward from currentPosition to endPosition. 
checkCollision()
 Behavior: Checks for when projectile hits endPosition.
