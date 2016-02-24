# SUPERPOWERS TUTORIAL #3 : SUPER SOKOBAN
## Chapter 4 : Level scripting


### Level checking

What we need now to complete our game is a level transition and victory checking,
to do so, we need to check the level.

We write a function **checkLevel** in our Main script, in the Game namespace :

```TypeScript
[...]
  export function checkLevel(level){

    /*
    We check all the level for the crates and targets positions.
    We then compare if the position of crates and targets match, if all crate are on target, the level is won.
    */

    // add all vector of crates and end position in the game and compare them together, if the all equal, the game is finished
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
      Sup.log("DEBUG : VICTORY"); // temporary log
    };
  }
[...]
```

To see it work, we need to add a local function in the Game namespace :

```TypeScript
[…]
  function checkVictory(level, boxesNumber, boxesPositions, targetsPositions){

    let posBox = new Sup.Math.Vector2;
    let posTarget = new Sup.Math.Vector2;
    let count : number = 0;

    for(posBox of boxesPositions){
      for(posTarget of targetsPositions){
        if(posBox.x === posTarget.x && posBox.y === posTarget.y){
          count++;
        }
      }
    }

    if(count === boxesNumber){
      return true;
    }
  }
[…]
```


We need to call the checkLevel function each time the player move, to do so we need to add a call in the move method of the PlayerBehavior, when the condition that check if the player can move is true.

```TypeScript
[…]
// (6)
    if(canMove === true){
      // We update the Player Actor with the new playerPosition vector coordinate
      this.actor.setPosition(playerPosition.x, playerPosition.y);

      Game.checkLevel(level);
    }
[…]
```

We can now play the first level and have a log when we solved the puzzle.

### Levels transition

We want now a transition and be able to go to the next level up until the end of the game.

First we create in the Main script a new function **setLevel** than we will call when we want to start a new level.

```TypeScript
[…]
  export function setLevel(){
        //reset values to default
        isLevelWon = false;
        //reload the scene
        Sup.loadScene("Game");
  }
[…]
```

Then we modify the update method of the levelBehavior.

```TypeScript
[…]
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
      }

      if(Sup.Input.wasKeyJustPressed("SPACE")){
        this.actor.getChild("Next").setVisible(false);

        this.level.setTileMap("Levels/"+LEVELS[levelCount]);

        Game.setLevel();
      }
    }
  }
[…]
```
We can now play the different level until the end, though, there will be a crash
as we have not set yet the victory scene.

### Victory Scene

We first create a new asset sprite Screen to which we attach the screen image. (grid size same as image size)

We create a new scene Victory with a new Actor Camera which we attach a new
component Camera, we set the mode to orthographic with a scale of 12.

We create a new Actor Screen to which we attach a sprite renderer, with the Screen sprite asset.

Both actor should be in position (0, 0) but the camera should have a z coordinate a
bit in front of the image like 2.


We can remove the temporary log of checkLevel and try the game. It should be complete now.

The only problem is that, right now, it is impossible to do mistake without the
necessity to restart the game. We will do a level reset option in the next chapter.
