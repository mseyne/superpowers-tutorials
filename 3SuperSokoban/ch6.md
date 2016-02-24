# SUPERPOWERS TUTORIAL #3 : SUPER SOKOBAN
## *Chapter 6 : Complete Game Source Reference*


### Assets files

* character.png
* next.png
* reset.png
* screen.png
* tiles.png
* scripts/player.ts
* scripts/level.ts
* scripts/main.ts



### Scripts source

main.ts

```TypeScript
[...]
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


// Original map pattern saved
var mapSaved : number[][];

// Set new player position to map origin
var playerPosition = new Sup.Math.Vector2(0, 0);

namespace Game{
  export function getMaxLevel(){
    levelMax = 0;
    // Add one for each level in LEVELS
    for(let level in LEVELS){
      levelMax++;
    }
  }

  export function getPosition(level){
    /*
      Scan the 16x12 level in order to :
      - Save the tile pattern for each layer in the mapSaved array
      - Set the playerPosition vector from the Start tile position on Actor layer
      - Change the Start tile by an empty tile (the Player Sprite will come instead)
    */

    // Set the variables to default, erase previous level
    mapSaved = [];
    playerPosition.x = 0, playerPosition.y = 0;

    for(let row = 0; row < 12; row++){
      for(let column = 0; column < 16; column++){

        // get the tile for x = column and y = row positions
        let actorTile = level.getTileAt(Layers.Actors, column, row);

        // Add the tile to the array
        mapSaved.push(actorTile);

        if(actorTile === Tiles.Start){
          // remove the Start tile and replace with empty tile
          level.setTileAt(Layers.Actors, column, row, Tiles.Empty);

          // set position to x, y on level map
          playerPosition.add(column, row);
        }
      }
    }
  }

  export function checkLevel(level){

    /*
    We check all the level for the crates and targets positions.
    We then compare if the position of crates and targets match,
    if all crate are on target, the level is won.
    */

    let boxesNumber : number = 0;
    let boxesPositions = [];
    let targetsPositions = [];

    for(let row = 0; row < 12; row++){
      for(let column = 0; column < 16; column++){

        // Take the tiles from two layers to coordinates column and row
        let actorTile = level.getTileAt(Layers.Actors, column, row);
        let worldTile = level.getTileAt(Layers.World, column, row);

        // If the actor tile is a box, keep the position
        if(actorTile === Tiles.Crate || actorTile === Tiles.Packet){
          let position = new Sup.Math.Vector2(column, row);
          boxesPositions.push(position);

          // we count the total number of crate
          boxesNumber++;
        }

        // If the world tile is a target, keep the position
        if(worldTile === Tiles.Target){
          let position = new Sup.Math.Vector2(column, row);
          targetsPositions.push(position);
        }
      }
    }

    // Check if all boxes
    if(checkVictory(level, boxesNumber, boxesPositions, targetsPositions)){
      isLevelWon = true;
      levelCount++;
    };
  }

  function checkVictory(level, boxesNumber, boxesPositions, targetsPositions){

    /*
    Check all the positions and find if the coordinate match together.
    If there is as much match than there is boxes, the game is finished.
    */

    let count : number = 0;

    for(let posBox of boxesPositions){
      for(let posTarget of targetsPositions){
        if(posBox.x === posTarget.x && posBox.y === posTarget.y){
          count++;
        }
      }
    }
    if(count === boxesNumber){
      return true;
    }
  }

  export function setLevel(){
        //reset values to default
        isLevelWon = false;
        //reload the scene
        Sup.loadScene("Game");
  }

  export function resetLevel(level){
    let index : number = 0;

    // set all the actor tiles of the current level to the savedMap tile
    for(let row = 0; row < 12; row++){
      for(let column = 0; column < 16; column++){
        level.setTileAt(Layers.Actors, column, row, mapSaved[index]);
        index++
      }
    }
    // call the setLevel function to prepare a new level
    setLevel();
  }
}

// Call the getMaxLevel function when game is launched
Game.getMaxLevel();
[...]
```

player.ts

```TypeScript
[...]
class PlayerBehavior extends Sup.Behavior {

  start() {
    // set position of Player actor to the playerPosition 2D vector
    this.actor.setPosition(playerPosition);
  }

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

      Game.checkLevel(level);
    }
    // (7)
    else{ // blocked, return to previous value
      playerPosition.subtract(x, y);
    }
  }

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
}
Sup.registerBehavior(PlayerBehavior);

[...]
```

level.ts

```TypeScript
[...]
class LevelBehavior extends Sup.Behavior {

  level =  this.actor.tileMapRenderer;

  awake() {
    // set Level actor  to the current level map path
    this.level.setTileMap("Levels/"+LEVELS[levelCount]);

    // call the getPositions function with the current tile map as parameter
    Game.getPosition(this.level.getTileMap());
  }

  update() {

    /*
    If the level is won we check
      - if it was the last level
        - if yes, we go to the victory screen
        - else we change the transition text visibility to true
      - if the key space is pressed
        - change the transition text visibility to false
        - change the new level tile map
        - call the setLevel function that will prepare the scene before to reload it
    */

    if(isLevelWon){
      if(levelCount == levelMax){
        Sup.loadScene("Victory/Scene");
      }
      else{
        this.actor.getChild("Next").setVisible(true);
        this.actor.getChild("Reset").setVisible(false);
      }

      if(Sup.Input.wasKeyJustPressed("SPACE")){
        this.actor.getChild("Next").setVisible(false);
        this.actor.getChild("Reset").setVisible(true);

        this.level.setTileMap("Levels/"+LEVELS[levelCount]);

        Game.setLevel();
      }
    }
    // if R key is pressed and the level is NOT won, then reset the whole level
    if(Sup.Input.wasKeyJustPressed("R") && !isLevelWon){
      Game.resetLevel(this.level.getTileMap());
    }
  }
}
Sup.registerBehavior(LevelBehavior);

[...]
```
