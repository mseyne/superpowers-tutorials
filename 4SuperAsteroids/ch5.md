# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 5


### **Building the basic program structure**

#### Global datas

We can now start programming our game, we first need to set differents datas to be accessible globally at any moment in our scripts. 
Often when we develop a game for the first time, the data are added, modified, removed or changed as we write the rest of our program, but for the tutorial
we write the datas first and will explain them again when we met them in our code.

We will create different namespace for each of our game object, alien, ship, asteroid.

First we create in our **Global** script a main namespace for the **Game** and add differents datas that we export to be able to access them from our others scripts. 


```ts
// Main game datas and functions
namespace Game{
  
  // Game timer max value
  export const timeMax: number = 7200; //7200 2 minutes
  
  // The game index of the current played game, 0 for Asteroids, 1 for Spacewar
  export let nameIndex: number;
  
  // The screen limits in width and height + outside border
  export let bounds : {
    width:number,
    height:number
  }
  
  // set a border of 2 x 1.5 units (invisible border of 24 pixels)
  let border: number = 3;
  
  // Game points won when target shot
  export enum points {
    asteroidBig = 10,
    asteroidMedium = 15,
    asteroidSmall = 20,
    alien = 50,
    ship = 100 ,
    death = -50,
     }
  
  // Life hearts positions
  export const hearts = [
    ["empty", "empty", "empty", "empty", "empty"],
    ["full", "empty", "empty", "empty", "empty"],
    ["full", "full", "empty", "empty", "empty"],
    ["full", "full", "full", "empty", "empty"],
    ["full", "full", "full", "full", "empty"],
    ["full", "full", "full", "full", "full"]
  ]
  
  // Flags to check HUD changes
  export var checkLifeHUD: boolean;
  export var checkScoreHUD: boolean;
}
```

We now write a **Menu** namespace and add datas related:

```ts
// Menu datas
namespace Menu{
  
  // Different menu screen index
  export const screens = {
      main : "Main",
      asteroids : "Asteroids",
      spacewar : "Spacewar",
      gameover : "GameOver",
     }
  
  // Game names index
  export enum names {
        Asteroids = 0,
        Spacewar = 1,
       }
}
```

We write the ship datas :

```ts
// Ship datas
namespace Ships{
  // Default ship size
  export const size: number = 0.5;
  // Default ship collision radius
  export const amplitude: number = 1.5;
  // Starting ship score
  export const startScore: number = 0;
  // Starting ship life
  export const startLife: number = 3;
  
  // Starting spawn positions
  export const spawns: Sup.Math.Vector3[] = [
    new Sup.Math.Vector3(0, 0, 14), // ship1 for asteroids game
    new Sup.Math.Vector3(-4, -12, 2), // ship1 for spacewar game
    new Sup.Math.Vector3(4, 12, 4) // ship1 for spacewar game
  ] 
  
  // ship index
  export enum index {
    ship1 = 0,
    ship2 = 1
    };
  
  // Starting time before next shoot
  export const shootingTimer: number = 30;
  // Starting time before respawn
  export const respawnTimer: number = 180;
  // Starting time before vulnerability
  export const invincibleTimer: number = 200;
  
  // Linear speed
  export const linearAcceleration: number = 0.005;
  // Linear slowing down
  export const linearDamping: number = 0.97;
  
  // Rotation speed
  export const angularAcceleration: number = 0.02;
  // Rotation slowing down
  export const angularDamping: number = 0.75;
  
  // Commands for each ship by index
  export const commands = [
    {left:"LEFT", right:"RIGHT", forward:"UP", shoot:"CONTROL", boost:"SHIFT"}, // commands[0]
    {left:"A", right:"D", forward:"W", shoot:"SPACE", boost:"C"} // commands[1]
  ];
  
  // Missiles list for each ship by index
  export let missiles: Sup.Actor[][];
  // Starting missile life before destruction (frames)
  export const missileLife: number = 60;
  // Missile speed (unit/frame)
  export const missileSpeed: number = 0.30;
}
```

We write the datas for the alien ship :

```ts
// Alien datas
namespace Alien{
  // Flag for alien alive
  export let alive: boolean = true;
  
  // Starting alien life
  export const startLife: number = 5;
  // Current alien life
  export let lifes: number;
  
  // Alien z position
  export const zPosition: number = 12;
  
  // Different alien sizes
  export const sizes: number[] = [1.7, 1.3, 1];
  // Different collision amplitude related to size
  export const amplitudes: number[] = [2.5, 2.2, 2];
  
  // Linear and rotation speed of alien ship
  export let linearSpeed: number = 0.05;
  export let rotationSpeed: number = 0.01;
  
  // Starting time before alien ship respawn
  export const respawnTime: number = 300;
  // Current time befer alien ship respawn
  export let spawnTimer: number = 0;
  
  // Starting time before alien ship shoot again
  export const shootTime: number = 200;
  
  // Alien missile list
  export let missiles: Sup.Actor[];
  // Alien missile speed (unit/frame)
  export const missileSpeed: number = 0.05;
}
```

We write the asteroids datas :

```ts
// Asteroids datas
namespace Asteroids{
  // List of all current asteroids
  export let list: Sup.Actor[];
  
  // Differents size of asteroids
  export const sizes = {"big":1.5, "medium":1, "small":0.5};
  // Different collision amplitude related to size
  export const amplitudes = {"big":2.5, "medium":2, "small":1}
  
  // Range for Z positions of asteroids
  export enum zPositions {min = -28, max = 10};
  
  // Starting asteroids number
  export const startCount: number = 5;
  // Current asteroids number
  export let currentCount: number;
  // Maximum asteroids number
  export const maxCount: number = 10;
  
  // Range of linear and rotation speed of asteroids
  export enum linearSpeed {min = -0.05, max = 0.05};
  export enum rotationSpeed {min = -0.01, max = 0.01};
  
  // Starting time before asteroids spawning
  export const respawnTime: number = 180;
  // Current time before asteroids spawning
  export let spawnTimer: number = 0;
}
```
We now have all our datas ready to use in our program.


#### Init game behavior

The GameBehavior class in ou **Game** script will be used as the main process running when our game start after we launch it from the menu. There will be differents branch
in this behavior depending which game is running. *By example, if the game is Game.nameIndex is 0 (Asteroids), then we can spawn the alien and the asteroids, else we don't.*

Let's start to write some process of the game. (we use the start method and not the awake method to init the behavior)

```ts
class GameBehavior extends Sup.Behavior {
  timer: number;

  start() {
    // When GameBehavior awake, if game is Asteroids then spawn asteroids and alien
    if(Game.nameIndex === 0){
      // Give to this game instance, the starting timer
      this.timer = Game.timeMax;
      // And update the HUD timer
      Game.updateTimer(this.timer);
      
      // Spawn an Alien ship
      Game.spawnAlien();
      
      // Spawn as much asteroids than we have set as a Starting number
      for(let i = 0; i < Asteroids.startCount; i++){
        // Spawn an asteroid
        Game.spawnAsteroid();
      }
    }
  }

  update() {
    // If the game is Asteroids then spawn alien and asteroids
    if(Game.nameIndex === 0){
      //If alien destroyed, spawn a new alien after a certain time
      if(!Alien.alive){
        // If spawnTimer not finished, decrease by one
        if(Alien.spawnTimer > 0){
          Alien.spawnTimer--;
        // If spawnTimer finished, spawn an alien ship
        }
        else {
          Game.spawnAlien();
        }
      }

      // Spawn a new asteroid after a certain time until the overall number reach maximum
      // If asteroid current count is less than the max number
      if (Asteroids.currentCount < Asteroids.maxCount) {
        // If spawnTimer is not finished, decrease by one
        if (Asteroids.spawnTimer > 0) {
          Asteroids.spawnTimer--;
        }
        // If spawnTimer is finished, create a big asteroid
        else {
          Game.spawnAsteroid();
          // Reset timer for next asteroid
          Asteroids.spawnTimer = Asteroids.respawnTime; 
        }
      }
      
      // Timer and game over screen decrease to each frame
      if (this.timer > 0) {
        this.timer--
        // If 60 frames passed (which mean one second when converted), update
        if(this.timer % 60 === 0) {
          Game.updateTimer(this.timer);
        }
      // If timer at 0, then the game is finished.
      } 
      else {
        // The game is over, return ship1 score
        Game.gameOver("ship1");
      }
    }
  }
}
Sup.registerBehavior(GameBehavior);
```

We now have four new functions than we haven't wrote yet :

* Game.spawnAsteroid() which create an asteroid in the game
* Game.spawnAlien() which create an alien ship in the game
* Game.updateTimer() which update the game HUD timer
* Game.gameOver() which display the game over screen

We return in the **Global** script to write them now.


#### Global first functions

We also need to start an important function, the Game.start() that will be used to initialize a new game.

In the Game namespace we add the five functions than we export globally.

```ts
namespace Game{
  [...]
  // Start the game
  export function start(){
  
  }
  
  // Create an alien in the Game scene              
  export function spawnAlien(){
    
  }
  
  // Create an asteroid in the Game scene
  export function spawnAsteroid(size: string){
    
  }
  
  // Update HUD timer in the Game scene
  export function updateTimer(time: number){
    
  }
  
  // Close game scene and return menu game over screen
  export function gameOver(winner?: string){
      
  }
}
```

##### Game.start

For now, we just need to set with this function the bounds of our game screen.


```ts
namespace Game{
  [...]  
  // Start the game
  export function start(){
    // get the Camera actor the the game Scene
    let screen = Sup.getActor("Camera").camera;
    // We get the game screen bounds and add the invisible border
    bounds = {
      width: screen.getOrthographicScale() + border,
      height: screen.getOrthographicScale() + border
    }
  }
  [...]
}
[...]
```

Because this function will change many time and will be called from the menu, we still need to launch our game and do tests before that.
We then call the function at the begining of the awake method of our Game script.

```ts
  [...]
  awake() {
    Game.start() // Temporary Function Call
[...]
```

##### Game.spawnAlien

```ts
  // Create an alien in the Game scene              
  export function spawnAlien(){
    // Load the alien prefab and get the Alien actor (index 0) as a new alien variable
    let alien = Sup.appendScene("Alien/Prefab")[0];
    
    // Choose a random position somewhere at the vertical edges of the game screen
    // Create the three axis positions x, y, z
    let x: number = 0, y: number = 0, z: number = 0;
    
     // Choose randomly a position on one of the vertical edges
    x = Sup.Math.Random.sample([-bounds.width / 2, bounds.width / 2]);
    // Get the height without the invisible border
    let height = bounds.height - border
    // Choose randomly a position on y axis
    y = Sup.Math.Random.integer(-height / 2 , height / 2);
    // Get alien default Z position
    z = Alien.zPosition;
    
    // Set the randomly chosen position to the Alien actor
    alien.setLocalPosition(x, y, z);
  }
```


##### Game.spawnAsteroid

```ts
  // Create an asteroid in the Game scene
  export function spawnAsteroid(){
    // Load the asteroid prefab and get the Asteroid actor (index 0) as a new asteroid variable
    let asteroid = Sup.appendScene("Asteroid/Prefab")[0];
    
    // Choose a random position somewhere at the edges of the game screen
    // Create the three axis positions x, y, z
    let x: number = 0, y: number = 0, z: number = 0;
    
    // Choose randomly if the asteroids come from the horizontal or vertical edges
    if(Sup.Math.Random.sample([0, 1]) === 0){
      x = Sup.Math.Random.sample([-bounds.width / 2, bounds.width / 2]);
      y = Sup.Math.Random.integer(-bounds.height / 2, bounds.height / 2);
    }
    else {
      x = Sup.Math.Random.integer(-bounds.height / 2, bounds.height / 2);
      y = Sup.Math.Random.sample([-bounds.width / 2, bounds.width / 2]);
    }
    // Choose randomly the z position on the scene in the range of Asteroids.zPoistions
    z = Sup.Math.Random.integer(Asteroids.zPositions.min, Asteroids.zPositions.max);
      
    // Set the randomly chosen position to the Asteroid actor
    asteroid.setLocalPosition(x, y, z);
    
    // Report the asteroid size
    asteroid.getBehavior(AsteroidBehavior).sizeClass = "big";
  }
```

To make it work without bug we need to init a variable in the Asteroid script, inside the class AsteroidBehavior :

```ts
class AsteroidBehavior extends Sup.Behavior {
  sizeClass: string;
[...]
```

##### Game.updateTimer

The updateTimer function use a certain amount of time in frames and convert it to second and minutes. We kept the default setting of Sueprpowers, 60 frames per seconds.

```ts    
  // Update HUD timer in the Game scene
  export function updateTimer(time: number){
    Sup.log("one second (60 frames) of", time, "frame left."); // Debug log
    // We convert frames in minutes and seconds
    let minutes = Math.floor((time / 60) / 60);
    let seconds = Math.floor((time / 60) % 60);
    // We get The Timer actor from the Game scene than we store in a timer variable
    let timer = Sup.getActor("HUD").getChild("Timer");
    // Update the text Renderer with the new time converted automatically as a string
    // For the last 10 seconds we need to add a 0 to keep 2 numbers in the timer
    if (seconds < 10) {
      timer.textRenderer.setText(minutes + ":0" + seconds);
    }
    else {
      timer.textRenderer.setText(minutes + ":" + seconds);
    }
  }
```

*Note : To try to launch the game and see the behavior of the timer and the HUD, we need to set the game nameIndex to 0.*

```ts  
  export let nameIndex: number = 0;  // Temporary allocation
```

##### Game.gameOver

We will comme back to this function when we will work on the menu, for now we can just log a string that will loop in the console to say the timer is out.

Note : the parameter with ? mean that it is an optional parameter.

```ts
  // Close game scene and return menu game over screen
  export function gameOver(winner?: string){
    Sup.log("Game Over");
  }
```

Ok, we now have a basic game structure, if we do test and launch the game we have a timer working and the asteroids and ship on the game, but apart for the timer and the displaying of alien ship
and asteroids nothing happen yet, for animation, we need to write individual behavior of this objects. We do that in the next chapter.

[<-- back to chapter 4](ch4.md) -- [go to chapter 6 -->](ch6.md)