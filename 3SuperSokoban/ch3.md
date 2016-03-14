# SUPERPOWERS TUTORIAL #3 : SUPER SOKOBAN
## *Chapter World and player logic*

We create 3 scripts asset :

* Main (we can erase the template code as we won't make any class here)
* Player
* Level

And in the Game scene we attach a new component behavior to the Player Actor and
add the PlayerBehavior class. The same for the Level actor with the class LevelBehavior.

### Setting main script

In the Main script we first need to set some global variables and datas.


 ```TypeScript
// List the game levels
const LEVELS = {
  0:'LevelTemplate',
  1:'Level1',
  2:'Level2',
  3:'Level3'
};

// List the map layers
enum Layers{
      World = 0,
      Actors = 1
     };

// List the map tiles
enum Tiles{
      Empty = -1,
      Wall = 0,
      Floor = 1,
      Target = 2,
      Crate = 3,
      Start = 4,
      Packet = 5
     };

// Game level won flag
var isLevelWon : boolean = false;

// Current Level, Start Level 1
var levelCount : number = 1;

// Number of level, checked when game awake
var levelMax : number;

// Set new player position to map origin
var playerPosition = new Sup.Math.Vector2(0, 0);
```

From here we can start a new **Game namespace** that will contain all our main functions for the game.

Let start with a simple function **getMaxLevel** that check how many level the game have.
And let's call it directly in our Main script.

 ```TypeScript
[...]
namespace Game{
  export function getMaxLevel(){
    levelMax = 0;
    // Add one for each level in LEVELS
    for(let level in LEVELS){
      levelMax++;
    }
  }
}

// Call the getMaxLevel function when game is launched
Game.getMaxLevel();
```

*Note : if we write a function to know the number of level instead of writing directly
in the variable, 3, it is because we want the program to check automaticaly if there is
level which have been added or removed, it will come in handy when we want to add
simply levels later to the game.*

### Player position

We need a function that detect the player position when a level start, we write
a function **getPosition** that check our level for the Start tile.

We add in the Game namespace of the Main script :

```TypeScript
[...]
  export function getPosition(level){
    /*
      Scan the 16x12 level in order to :
      - Set the playerPosition vector from the Start tile position on Actor layer
      - Change the Start tile by an empty tile (the Player Sprite will come instead)
    */

// Set the variable to default
    playerPosition.x = 0, playerPosition.y = 0;

    for(let row = 0; row < 12; row++){
      for(let column = 0; column < 16; column++){

        // get the tile on Actors layer for x = column and y = row positions
        let actorTile = level.getTileAt(Layers.Actors, column, row);

        if(actorTile === Tiles.Start){

          // remove the Start tile and replace with empty tile
          level.setTileAt(Layers.Actors, column, row, Tiles.Empty);

          // set position to x, y on level map
          playerPosition.add(column, row);
        }
      }
    }
  }
[...]
```

We will know awake our behaviors, to be sure to avoid conflict in the awaking order,
there is two steps in initializing a class in Superpowers, first all the awake method
 of all behavior start, then the start methods are called (and only then, the update loop start),
 we want the level to be fully loaded before to awake the Player actor.


We use the method awake for the levelBehavior, where we will call our function getPosition :

 ```TypeScript
class LevelBehavior extends Sup.Behavior {

  level =  this.actor.tileMapRenderer;

  awake() {
    // set Level actor  to the current level map path
    this.level.setTileMap("Levels/"+LEVELS[levelCount]);

    // call the getPositions function with the current tile map as parameter
    Game.getPosition(this.level.getTileMap());
  }

  update() {

  }
}
Sup.registerBehavior(LevelBehavior);
```

Then we use the start method for the PlayerBehavior where we will set the start
position to the player Actor:

```TypeScript
class PlayerBehavior extends Sup.Behavior {

  start() {
    // set position of Player actor to the playerPosition 2D vector
    this.actor.setPosition(playerPosition);
  }

  update() {

  }
}
Sup.registerBehavior(PlayerBehavior);
```

We now have our game launching with the first level and our player to the starting
position. We now want our player to be able to move and interact in the level.


### Player movement and world interaction

We do this in two part, we first need to build a custom method than we call move
which take coordinates as parameters and is executed when a movement key is pressed.
This method will contain the condition we need to check for the player interaction with the world.

 ```TypeScript
[…]
  move(x, y){

    /*
    We update the future position of the player and start to check condition :
    - (1) if the next tile is an empty floor, the player can move, canMove is true
    - (2) else, canMove is false
    - (3) if the next tile is a box, then we check a second branch of condition :
        - (4) if the the next tile after the box tile is an empty floor, the player can push the box
        - (5) else, canMove is false
    - (6) if canMove is true, then we update the SpriteRenderer of the Player Actor to the new position
    - (7) else, we take back the previous coordinates for the player position
    */

    let canMove: boolean;

    // Set to new coordinates
    playerPosition.add(x, y);

    // We get the tiles index for each layer of the map from the new coordinates
    let level = Sup.getActor("Level").tileMapRenderer.getTileMap();
    let tileWorld = level.getTileAt(Layers.World, playerPosition.x, playerPosition.y);
    let tileActors = level.getTileAt(Layers.Actors, playerPosition.x, playerPosition.y);

    // (1)
    if(tileWorld === Tiles.Floor || tileWorld === Tiles.Target){
      canMove = true;

      // (3)
      if(tileActors == Tiles.Crate || tileActors == Tiles.Packet){

        // We get the tiles index for each layer of the map from the coordinates after the new ones
        let nextWorldTile = level.getTileAt(Layers.World, playerPosition.x + x, playerPosition.y + y);
        let nextActorsTile = level.getTileAt(Layers.Actors, playerPosition.x + x, playerPosition.y + y);

        // (4)
        if(nextWorldTile == Tiles.Floor && nextActorsTile == Tiles.Empty){

          level.setTileAt(Layers.Actors, playerPosition.x, playerPosition.y, Tiles.Empty);
          // If the next world tile is floor, the box is a crate.
          level.setTileAt(Layers.Actors, playerPosition.x + x, playerPosition.y + y, Tiles.Crate);

        }
        // (4)
        else if(nextWorldTile == Tiles.Target && nextActorsTile == Tiles.Empty){

          level.setTileAt(Layers.Actors, playerPosition.x, playerPosition.y, Tiles.Empty);
          // If the next world tile is a target tile, the box is a packet.
          level.setTileAt(Layers.Actors, playerPosition.x + x, playerPosition.y + y, Tiles.Packet);

        }
        // (5)
        else{
          canMove = false;
        }
      }
    }
    // (2)
    else{
      canMove = false;
    }

    // (6)
    if(canMove === true){
      // We update the Player Actor with the new playerPosition vector coordinate
      this.actor.setPosition(playerPosition.x, playerPosition.y);
    }
    // (7)
    else{ // blocked, return to previous value
      playerPosition.subtract(x, y);
    }
  }
[...]
 ```

Then we can build a condition structure in the update loop of the player script
to check the keyboard inputs.

 ```TypeScript
[…]
  update() {
    // if the level is NOT won, the player have control
    if(!isLevelWon){
      if(Sup.Input.wasKeyJustPressed("UP")){
        this.move(0, 1);
        this.actor.spriteRenderer.setAnimation('up');
      }
      else if(Sup.Input.wasKeyJustPressed("DOWN")){
        this.move(0, -1);
        this.actor.spriteRenderer.setAnimation('down');
      }
      else if(Sup.Input.wasKeyJustPressed("LEFT")){
        this.move(-1, 0);
        this.actor.spriteRenderer.setAnimation('left');
      }
      else if(Sup.Input.wasKeyJustPressed("RIGHT")){
        this.move(1, 0);
        this.actor.spriteRenderer.setAnimation('right');
      }
    }
  }
[...]
 ```

We have now the complete main game mechanic working, we can move the player and
push the boxes until the target destinations.

We now have to give a flow to our game, to check the level for the game situation
and have a level transition.
