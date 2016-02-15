# SUPERPOWERS TUTORIAL #2 SUPER OXO
## *Chapter 4 : Writing the Computer response, Game logic part 2*

### Algorithm of a simple computer AI

What do we want from the computer and what does it mean in our case to create an
artificial intelligence ? Don't worry, it is not as difficult than it appear, in
fact we call here AI the fact that our program will check the behavior of the player
and the global situation before to take any action instead of just automatically
giving the same response to any situation.

To be more precise, each time the computer will have a turn, it will check the board
and the situation of each square and act accordingly to assure it's own victory.

The script consist of a little algorithm. It is differents conditions like step
than the computer will follow and check until it find the most appropriate response.
Here what the algorithm looks like in english :

By checking the game board, the computer should be able to know :

* If there is a way to win the game this turn.
   * And if yes, just play to win.
* Else if there is no way to win, but the player is going to win the next turn.
   * If yes, then play to block the player.
* Else if there is no way to win or no danger from the player then let's play.
   * If there is still a free square on the board
      * If yes, let's play the center square, I like it the best :)
      * If there is no center free, let's play one of the corner square, That sound nice :)
      * If there is no corner free either, let's play a side square, I take what left :(
      * If there is no free square on the board, I guess the game is finished


### Scripting the computer behavior

There is one important data to give to the computer in a way to help the checking
process is what it mean to be victorious. We have to give to the computer what to
achieve in the game to use it like a reference. For this, we need to write a constant
that will contain all the victory lines in a Tic Tac Toe game.

Simply put, here the **VICTORIES** constant than we add in our global script.

```TypeScript
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
[…]
```

*Note : It is an Array of arrays, each array is a victory line and give the three
index position of each square to get to be victorious.*

Now we can start to write in code our algorithm. What we want first is a function
that check the board and return datas that could be used to take a decision.

We add a new function **checkBoard** in the Game namespace of the Global script.

```TypeScript
[…]
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
[…]
```


We then write a second function **computerTurn** (exported to access it from outside
the namespace) which is the steps conditions that will return a response from the computer.  
This function will call first the checkBoard as a support to get informations needed before
to check what is the response to give back to the player accordingly to the datas.

Here the basic structure of the function :

```TypeScript
[…]
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
      Sup.log("I play to win.");
    }

    else if(check[0] == "Block"){
      Sup.log("I play to block.")
    }

    else if(check[0] == "Play"){

      if(false){
        Sup.log("I play the center.");
      }

      else if(false){
        Sup.log("I play a corner.");
      }

      else if(true){
        Sup.log("I play a side.");
      }

      else{
        Sup.log("The game is finished.")
      }
    }
  }
[…]
```

To be able to check if the computerTurn and checkBoard functions work fine we need
to make some change inside the Player script when the player play by clicking on a
square and initialize the computer turn. We replace the Sup.log in the gameTurn()
method by a call for the function **Game.computerTurn()**.

Here the method once we changed it :

```TypeScript
[…]
  gameTurn(){
    // change to computer turn
    turn = "circle";
    // Call for the computer turn
    Game.computerTurn();
    // change to player turn
     turn = "cross";
  }
[…]
```

We have now the complete structure of the computer turn minus the part when the
situation is Play for the computer and there is no change in the SQUARES board,
which make the game incomplete and illogical.

Right now, it is already possible to test the computer turn in the console log
but not completely, because the computer only say 'I play to block' if the player
is about to win or say 'I play a side' because for demonstration we set it up this
condition true by default.

We need to write more to complete the computer turn condition checking, let's
focus now on the part where the game situation is 'Play'.

We have three conditions, let see them one by one.

1. if the center is free we take  it (we know than the index of the center is 4)

```TypeScript
[…]
    else if(check[0] == "Play"){

      if(SQUARES[4][1] !== "cross" && SQUARES[4][1] !== "circle"){
        Sup.log("I play the center.");
      }
[…]
```

2. else if there is free corner (we know than corners are index 0, 2, 6 and 8)

```TypeScript
[…]
      else if(
              SQUARES[0][1] !== "cross" && SQUARES[0][1] !== "circle" ||
              SQUARES[2][1] !== "cross" && SQUARES[2][1] !== "circle" ||
              SQUARES[6][1] !== "cross" && SQUARES[6][1] !== "circle" ||
              SQUARES[8][1] !== "cross" && SQUARES[8][1] !== "circle"
              ){
        Sup.log("I play a corner.");
      }
[…]
```

3. else if there is free side (we know than sides are index 1, 3, 5 and 7)

```TypeScript
[…]
      else if(
              SQUARES[1][1] !== "cross" && SQUARES[1][1] !== "circle" ||
              SQUARES[3][1] !== "cross" && SQUARES[3][1] !== "circle" ||
              SQUARES[5][1] !== "cross" && SQUARES[5][1] !== "circle" ||
              SQUARES[7][1] !== "cross" && SQUARES[7][1] !== "circle"
              ){
        Sup.log("I play a side.");
      }
[…]
```

We have now all our conditions ready. We can now finish the function computerTurn
by returning the correct square and apply the change on it.

We first need to define a new function getSquare in the namespace Game, we will
use it to get a free square from an array of index when we want specifically check
for a free corner or a free side square.

```TypeScript
[…]
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
[…]
```

We also define a new function **playSquare** which take a square as a parameter. It's purpose
is simply to modify the situation and the sprite rendering when the computer call for it.

```TypeScript
[…]
  function playSquare(square){
    // apply change on the actor and change the situation to circle
    square[0].spriteRenderer.setAnimation("circle");
    square[1] = "circle";
  }
[…]
```

We can now call the function playSquare with the related square each time our condition
is true in the computerTurn function.

Here the (nearly) completed computerTurn function :

*Note : we replaced all the Sup.log by the function calls*

```TypeScript
[…]
  export function computerTurn(){

    // Call the function checkBoard which return an array [Situation, freeSquare]
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
    // end of computer turn, change to player turn
    turn = "cross";
  }
[…]
```

The computer now play against the player, it use the checkBoard to understand the
situation and respond accordingly. The AI is now complete.

The game right now have still one issue, it is never ending, it just continue when
there is a winner or the board is filled. We need now to write a function that will check the end of game.


## Scripting the victory checking

At this point, the game work, we can play with the mouse and the computer can respond.
We want now than the game detect and tell us if the player or the computer won the game and stop the game turns.

We now write the **checkVictory()** function (exported) as follow (see comments for explanation) :

```TypeScript
[…]
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
        Sup.log("Player won !");
        turn = "end";
      }
      if(countCircle == 3){
        Sup.log("Computer won !");
        turn = "end";
      }
      // reset count to 0 for new line to check
      countCross = 0;
      countCircle = 0;
    }
   // end of loop
   if(countFreeSquares == 0){
        Sup.log("It is a tie !");
        turn = "end";
    }
  }
[…]
```

We need to change one more time the gameTurn method in the Player Script to call
our function checkVictory to each turn and stop or continue the turns according to the results of the function.

```TypeScript
[…]
  gameTurn(){
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
[…]
```

The game is finished, we can play the full game, the player can choose where to play,
the computer can respond and one can win the game or have together a tie.

Sometimes, it is good to polish the overall game feeling by adding some enhancing features.
It is what we will do in the last chapter.
