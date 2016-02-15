# SUPERPOWERS TUTORIAL #2 SUPER OXO
## *Chapter 5, Polishing the game*

### Create a victory screen

One first thing we would like to add in the game is the possibility to restart the
game after we finish a first round. We can do that by simply allowing to reset to
default the SQUARES board when the player do an action after the game is ended.

We are going to mix this feature with the addition of an end screen which announce
the victory or tie of the game. When the player click once more on this screen,
the game then restart.

We write first a new function **displayScreen** that will create and set our Screen Actor.

```TypeScript
[...]
  function displayScreen(){
    // Create a new Screen actor
    let Screen = new Sup.Actor("Screen");
    // Create a new SpriteRenderer and attach it to the Screen Actor
    new Sup.SpriteRenderer(Screen, "Sprites/Screens");
    // Set the frame of the sprite to the current turn, cross, circle or tie
    Screen.spriteRenderer.setAnimation(turn);
    // Set the frame position to the center (0, 0) and 4 on z axis to fit in the camera view
    Screen.setPosition(0, 0, 4);
    // stop the game turns
    turn = "end";
  }
[...]
```

We add the call of the victory screen by replacing the Sup.log and the repetitions
of turn = "end" from the checkVictory conditions.

```TypeScript
[...]
  export function checkVictory(){
    [...]
      if(countCross == 3){
        displayScreen();
      }
      if(countCircle == 3){
        displayScreen();
      }

[...]
   if(countFreeSquares == 0){
        turn = "tie"; // we give the value 'tie' to the turn variable for the function displayScreen()
        displayScreen();
    }
  }
[...]
```

We have now the screens displaying, but if we want to restart and reset the game
we now need to write a new behavior for the player.

### Restart the game

In the Player script we create the new class **ScreenBehavior** and we say simply than
we want to survey in a loop if the player will use the space key from the keyboard.
To do that, we use the update method of a class. Here our new class:

```TypeScript
class ScreenBehavior extends Sup.Behavior {

  /*
  When space key pressed :
  - Destroy the victory screen
  - Start a new game
  */

  update() {
    if(Sup.Input.wasKeyJustPressed("SPACE")){
      Sup.getActor("Screen").destroy();
      Game.startGame();
      }
    }
 }
Sup.registerBehavior(ScreenBehavior);
```

In the global script namespace we create a function **startGame()** that we export.
It will be the function that will be called when the player use SPACE key while on a victory screen.

```TypeScript
[...]
  export function startGame(){
    let square;

    // loop through all the square of the game and set them to default
    for(square of SQUARES){
      square[0].spriteRenderer.setAnimation("unHover");
      square[1] = "unHover";
    }

    turn = "cross"; // give back player control
  }
[...]
```

Finally, nothing will change if we don't attach the ScreenBehavior class to our
victory screen. We do that by adding with the Screen.addBehavior() method in the
displayScreen function of the Global script.

```TypeScript
[...]
  function displayScreen(){
    […]
    // Attach the behavior ScreenBehavior the the screen Actor
    Screen.addBehavior(ScreenBehavior);
  }
[...]
```

We can now restart our game from the end screen and play as many times we wish to.

## Randomize the game starting

One feature we can add to give more variety to the game is to randomize the start
of the game, if it is the player or the computer who can choose the first square.
We already had to use the random method in our game, we can use it again

We create a new function **randomStart()** in Game namespace of global script, it will
decide if yes or no the computer start the game by playing randomly one square.

```TypeScript
[…]
  function randomStart(){
    //Call a random number between 0 and 1 to see if we start with computer
    if(Math.floor(Math.random() * 2)){
      // Take a random index between the 9 squares and play this square
      let randomIndex = Math.floor(Math.random() * 9);
      playSquare(SQUARES[randomIndex]);
    }
    // then give back control to player
    turn = "cross";
  }
[...]
```

And we call now this function from our startGame function to replace the turn = 'cross',
this will allow to randomize a new game each time we call the gameStart function.

```TypeScript
[…]
  export function startGame(){
    let square;

    // loop through all the square of the game and set them to default
    for(square of SQUARES){
      square[0].spriteRenderer.setAnimation("unHover");
      square[1] = "unHover";
    }

    randomStart() ;
  }
[...]
```

We can also replace in the awake method of playerBehavior the turn = 'cross' by
the gameStart function, like this, the randomize will work from the moment we
launch the game for the first time.

```TypeScript
class PlayerBehavior extends Sup.Behavior {
  awake() {
    // Call the function setSquares which build the SQUARES Array
    Game.setSquares();
    // Call the function startGame to randomize the new game
    Game.startGame();
  }
[...]
```

### Slowing down the game

The game go really fast, the computer play directly after the player and at the end,
the victory screen appear without the possibility to see the final position of the
game squares. We want to slow down all this. To give time for the player to see
what is happening on the board and why the game is going to end soon.

We need to add a speed to the game. To achieve that we will use a javascript method
that have been exported to Superpowers API : Sup.setTimeout()

We will change the code to different place, to slow down the gameTurn, we can change
the simple call this.gameTurn() from our update method from the playerBehavior class
in the Player script :

```TypeScript
[…]
if(Sup.Input.wasMouseButtonJustPressed(0) && array[1] == "isHover"){
          // Check if it is the player turn
          if(turn == "cross"){
            // if true, set the square new situation to cross
            array[1] = "cross";
            // and call the local mouse method with the action click and the related square actor
            this.mouse("click", array[0]);
            // call a game turn
            turn = "break"; // take control away from player
            Sup.setTimeout(600, this.gameTurn);
          }
        }
[...]
```

*Note : We change the turn variable before the timeout break because the player
would then be able to click on the other squares before the computer got the chance to play.*

Like we changed the turn variable before, we need to give the cross value back to
the turn variable after the break but before the checkVictory function in the
ameTurn Method from the playerBehavior class.


```TypeScript
[...]
gameTurn(){
    turn = "cross";
    // check if player won
    Game.checkVictory();
[…]
```

That is slowing down our game a little bit before each computer turn.

For the last move, to have time before the end screen appear, we need to play a
trick and simply change the screen position from behind the background to front
after the timer is out. We need to add a little function inside our function to
be able to call with the setTimeout method. We change our displayScreen function
from the global script like this :

```TypeScript
[…]
  function displayScreen(){
    // Create a new Screen actor
    let Screen = new Sup.Actor("Screen");
    // Create a new SpriteRenderer and attach it to the Screen Actor
    new Sup.SpriteRenderer(Screen, "Sprites/Screens");
    // Set the frame of the sprite to the current turn, cross, circle or tie
    Screen.spriteRenderer.setAnimation(turn);
    // Attach the behavior ScreenBehavior the the screen Actor
    Screen.addBehavior(ScreenBehavior);

    // Set the frame position to the center (0, 0) and -2 on z axis to be behind the board backgroung
    Screen.setPosition(0, 0, -2);
    turn = "end";

    function displayFrame(){
      // Set the frame position to the center (0, 0) and 4 on z axis to be in front of the board backgroung
      Screen.setPosition(0, 0, 4);
    }
    // load the displayFrame function after 2000 millisecondes
    Sup.setTimeout(2000, displayFrame);
  }
[…]
```

Yes ! this time our game is ready for release. :)
