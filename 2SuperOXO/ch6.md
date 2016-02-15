# SUPERPOWERS TUTORIAL #2 SUPER OXO
## *Chapter 6, Complete Game Source Reference*

### Assets files

* board.png
* symbols.png
* screens.png
* scripts
   * global.ts
   * player.ts

### Project structure

* Game (Startup Scene)
* Sprites
   * Symbols
   * Screens
   * Board
* Scripts
   * Global
   * Player

### Scripts sources

* global.ts

```TypeScript
// Initialize globally the SQUARES Array
const SQUARES = new Array;

// Initialize globally the ray casting
var ray = new Sup.Math.Ray();

// Initialize globally the turn variable
var turn : string;

// All the victory lines position index that will be used for checking
const VICTORIES = [
  // Rows
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  // Columns
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  // Diagonals
  [0, 4, 8],
  [2, 4, 6],
];

namespace Game{

  // define and export the function setSquare
  export function setSquares(){

    /*
    Build the SQUARES array with arrays
    0 : the Square Actor of current index
    1 : default string value of situation : "unHover"
    */

    // define a local addSquare function
    function addSquare(index){
      // get the name of current square
      let name = "Square" + index.toString();

      // get the actor from the Game scene
      let square = Sup.getActor("Board").getChild(name);

      // push the square array in SQUARES array to the next index
      SQUARES.push([square, "unHover"]);
    }

    // loop calling 9 times the function addSquare with the square index as parameter
    for(let i = 0; i < 9; i++){
      addSquare(i);
    }
  }

  // function checking the board SQUARES and return [Action, Array]
  function checkBoard(){
    let crossCount : number = 0;
    let circleCount : number = 0;
    let freeSquare;
    let line; let index;
    let win;
    let block;

    // loop through all the line in VICTORIES
    for(line of VICTORIES){
      // loop through all the index position in line
      for(index of line){
        /*
        Check the situation of square from index
        - if there is a cross, increment crossCount
        - if there is a circle, increment circleCount
        - else we keep track of it as a free square
        */
        if(SQUARES[index][1] == "cross"){
          crossCount++
        }
        else if(SQUARES[index][1] == "circle"){
          circleCount++
        }
        else{
          freeSquare = SQUARES[index];
        }
      }
      // Check is there if a winning line for computer
      if(circleCount == 2 && crossCount == 0){
        win = ["Win", freeSquare];
      }
      // Check is there is a winning line for player
      if(crossCount == 2 && circleCount == 0){
        block = ["Block", freeSquare];
      }
      // Reset counts for new line check
      crossCount = 0;
      circleCount = 0;
    }

    // Return datas by order of priority
    if(win){
      return win;
    }
    else if(block){
      return block;
    }
    else{
      return ["Play", undefined];
    }
  }

  export function computerTurn(){

    // Call the function checkBoard which return [Action, Array]
    let check = checkBoard();

    /*
    The computer follow the conditions as follow :
    - if the situation is Win, then play the freeSquare
    - else if the situation is Block, then play the freeSquare
    - else if the situation is Play, then we enter to a new branch of conditions :
      - if the center is free, then take it
      - else if one corner is free, then take it
      - else if one one side is free, then take it
      - else, the game is finished
    */

    if(check[0] == "Win"){
      playSquare(check[1]);
    }

    else if(check[0] == "Block"){
      playSquare(check[1]);
    }

    else if(check[0] == "Play"){

      if(SQUARES[4][1] !== "cross" && SQUARES[4][1] !== "circle"){
        playSquare(SQUARES[4]);
      }

      else if(
              SQUARES[0][1] !== "cross" && SQUARES[0][1] !== "circle" ||
              SQUARES[2][1] !== "cross" && SQUARES[2][1] !== "circle" ||
              SQUARES[6][1] !== "cross" && SQUARES[6][1] !== "circle" ||
              SQUARES[8][1] !== "cross" && SQUARES[8][1] !== "circle"
             ){
        playSquare(getSquare([0, 2, 6, 8]));
      }

      else if(
              SQUARES[1][1] !== "cross" && SQUARES[1][1] !== "circle" ||
              SQUARES[3][1] !== "cross" && SQUARES[3][1] !== "circle" ||
              SQUARES[5][1] !== "cross" && SQUARES[5][1] !== "circle" ||
              SQUARES[7][1] !== "cross" && SQUARES[7][1] !== "circle"
             ){
        playSquare(getSquare([1, 3, 5, 7]));
      }

      else{
        Sup.log("The game is finished.")
      }
    }
  }

  // function that return a square that is free to play
  function getSquare(array){
    let index;
    let freeSquares = new Array;

    /*
    Loop that check the array index in SQUARES
    and if the square is free to take, add it to the array freeSquares
    */

    for(index of array){
      if(SQUARES[index][1] !== "cross" && SQUARES[index][1] !== "circle"){
        freeSquares.push(SQUARES[index]);
      }
    }
    // then take randomly one the square from freeSquares and return it
    let randomIndex = Math.floor(Math.random() * freeSquares.length);
    return freeSquares[randomIndex];
  }

  function playSquare(square){
    // apply change on the actor and change the situation to circle
    square[0].spriteRenderer.setAnimation("circle");
    square[1] = "circle";
  }

  export function checkVictory(){
    //set the variables
    let countCross: number = 0;
    let countCircle: number = 0;
    let countFreeSquares: number = 0;
    let line: number[];
    let index: number;

    /*
    We loop through the victory lines to check this conditions for each square:
    - if the square is hovered by a cross, increment countCross
    - else if the square is hovered by a circle, increment countCircle
    - else, count it a a free square in the countFreeSquares variable

    For each line checked, we then look for this conditions :
    - if there is 3 cross counted, then player won
    - if there is 3 circle counted, then computer won

    At the end of the loop, if countFreeSquares is still 0 and no victory is announced,
    then it is a tie.
    */

    //loop
    for(line of VICTORIES){
      for(index of line){
        if(SQUARES[index][1] == "cross"){
          countCross++
        }
        else if(SQUARES[index][1] == "circle"){
          countCircle++
        }
        else{
          countFreeSquares++
        }
      }
      if(countCross == 3){
        displayScreen();
      }
      if(countCircle == 3){
        displayScreen();
      }
      // reset count to 0 for new line to check
      countCross = 0;
      countCircle = 0;
    }
   // end of loop
   if(countFreeSquares == 0){
        turn = "tie"; // we give the value 'tie' to the turn variable for the function displayScreen()
        displayScreen();
    }
  }

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
    // load the displayFrame function after 3000 millisecondes
    Sup.setTimeout(2000, displayFrame);
  }

  export function startGame(){
    let square;

    // loop through all the square of the game and set them to default
    for(square of SQUARES){
      square[0].spriteRenderer.setAnimation("unHover");
      square[1] = "unHover";
    }

    randomStart();
  }

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
}
```

* player.ts

```TypeScript
class PlayerBehavior extends Sup.Behavior {
  awake() {
    // Call the function setSquares which build the SQUARES Array
    Game.setSquares();
    // Call the function startGame to randomize the new game
    Game.startGame();
  }

  // mouse method receiving parameters of the player action and the square related to this action
  mouse(action, square){
    // change the square sprite depending of action
    if(action == "isHover"){
      square.spriteRenderer.setAnimation("isHover");
    }
    else if(action == "unHover"){
      square.spriteRenderer.setAnimation("unHover");
    }
    else if(action == "click"){
      square.spriteRenderer.setAnimation("cross");
    }
  }

  gameTurn(){
    turn = "cross";
    // check if player won
    Game.checkVictory();

    // change to computer turn if game not ended
    if(turn !== "end"){
      turn = "circle";
      // Call for the computer turn

      Game.computerTurn();
      // check if computer won
      Game.checkVictory();

      // change to player turn if game not ended
      if(turn !== "end"){
       turn = "cross";
      }
    }
  }

  update() {
    // Refresh the ray casting to the mouse position inside the camera screen
    ray.setFromCamera(Sup.getActor("Camera").camera, Sup.Input.getMousePosition());

    // Create a new empty variable that will as vallue the differents array of the SQUARES constant
    let array;

    /*
    We loop through all the arrays in SQUARES and give the current array to the square variable.
    We then check differents conditions :
    - If the mouse ray intersect with a current square :
        - then if this same square was previously not hovered.
        - and then if there is the left click button pressed from the mouse.
    - Else if the mouse ray leave a square previously hovered.
    */

    // Loop which give successively to array the values of the SQUARES array.
    for(array of SQUARES){

      // Check if ray intersect with a current square (index 0 of array)
      if(ray.intersectActor(array[0], false).length > 0){

        if(array[1] == "unHover"){
          // if true, set the square new situation to isHover
          array[1] = "isHover";
          // and call the local mouse method with the action isHover and the related square actor
          this.mouse("isHover", array[0]);
        }

        // Check if the left click button of the mouse is pressed on a free square
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
      }

      // Else if ray does not intersect with a previous hovered square, the square change situation
      else if(array[1] == "isHover"){
        // if true, set the square new situation to unHover
        array[1] = "unHover";
        // and call the local mouse method with the action click and the related square actor
        this.mouse("unHover", array[0]);
      }
    }
  }
}
Sup.registerBehavior(PlayerBehavior);

class ScreenBehavior extends Sup.Behavior {

  /*
  when space key pressed :
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
