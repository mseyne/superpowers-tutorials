# SUPERPOWERS TUTORIAL #3 : SUPER SOKOBAN
## Chapter 5 : Polishing the game


### Restarting a level

To be able to restart a level, we need to same the Tile pattern before we make
change to it. We will keep the original map in an Array that we will use if the player ask to reset the level.

First in the main script, we create a new global variable.


```TypeScript
[...]
// Original map pattern saved
var mapSaved : number[][];
[...]
```

And we will use the getPosition function which check all the tile to find the
start player, to also save all our tiles from the Actors layers inside the mapSaved variable.

Here the updated function :

```TypeScript
[...]
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

    for(let column = 0; column < 12; column++){
      for(let row = 0; row < 16; row++){

        // get the tile for x = row and y = column positions
        let actorTile = level.getTileAt(Layers.Actors, row, column);

        // Add the tile to the array
        mapSaved.push(actorTile);

        if(actorTile === Tiles.Start){
          // remove the Start tile and replace with empty tile
          level.setTileAt(Layers.Actors, row, column, Tiles.Empty);

          // set position to x, y on level map
          playerPosition.add(row, column);
        }
      }
    }
  }
[...]
```

Once a level start, the actors layer of the level is saved.


Now, inside the Game namespace of the main Script, we add a resetLevel function.

```TypeScript
[...]
  export function resetLevel(level){
    let index : number = 0;

    // set all the actor tiles of the current level to the savedMap tile
    for(let column = 0; column < 12; column++){
      for(let row = 0; row < 16; row++){
        level.setTileAt(Layers.Actors, row, column, mapSaved[index]);
        index++
      }
    }
    // call the setLevel function to prepare a new level
    setLevel();
  }
[...]
```

Now we add an Input condition in the update loop of Levelbehavior.
 in the level script make it invisible and visible again in the transition step.


```TypeScript
[...]
  update() {

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

[...]
```

Finally, we can make visible the Reset text in the Game scene and launch the game. It is finished.


### Adding sound and music to the game

I will update this part later when I finish the music.


### Add new levels to the game

It is easy to add a level to the game, we just need to duplicate the levelTemplate,
change it name with a number like Level4 and don't forget to add as much boxes than target.

Also it is important to add a player start position.

Then we just need to add it to the LEVELS list in our main script like so :

```TypeScript
[...]
// List the game levels
const LEVELS = {
  0:'LevelTemplate',
  1:'Level1',
  2:'Level2',
  3:'Level3',
  4:'Level4'
};
[...]
```

Note : If you need idea of sokoban puzzle, you can [check here](http://sokoban.info/).
