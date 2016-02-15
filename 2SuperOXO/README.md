# SUPERPOWERS TUTORIAL #2 SUPER OXO
## *Learn game development with a Tic Tac Toe game*

### Summary

1. [Introduction](ch1.md#chapter-1--introduction)
   * [Planning the game](ch1.md#planning-the-game)
   * [MVG Features](ch1.md#mvg-features)
2. [Building the Game Structure](ch2.md#chapter-2-building-the-game-structure)
   * [Loading the game assets](ch2.md#loading-game-assets)
   * [Building the game scene](ch2.md#building-the-game-scene)
3. [Writing the Player actions, Game logic part 1](ch3.md#chapter-3--writing-the-player-actions-game-logic-part-1)
   * [Setting the global script](ch3.md#setting-the-global-script)
   * [Scripting th mouse behavior](ch3.md#scripting-the-mouse-behavior)
   * [A turn based game](ch3.md#a-turn-based-game)
4. [Writing the Computer response, Game logic part 2](ch4.md#chapter-4--writing-the-computer-response-game-logic-part-2)
   * [Algorithm of a simple computer AI](ch4.md#algorithm-of-a-simple-computer-ai)
   * [Scripting the computer behavior](ch4.md#scripting-the-computer-behavior)
   * [Scripting the victory checking](ch4.md#scripting-the-victory-checking)
5. [Polishing the Game](ch5.md#chapter-5-polishing-the-game)
   * [Create a victory screen](ch5.md#create-a-victory-screen)
   * [Restart the game](ch5.md#restart-the-game)
   * [Randomize the game starting](ch5.md#randomize-the-game-starting)
   * [Slowing down the game](ch5.md#slowing-down-the-game)
6. [Complete Game Source Reference](ch6.md#chapter-6-complete-game-source-reference)
   * [Assets files](ch6.md#assets-files)
   * [Project structure](ch6.md#project-structure)
   * [Scripts sources](ch6.md#scripts-sources)

### What we will learn in this tutorial

- Set up assets and build a scene for a Tic Tac Toe in Superpowers
- Write a mouse behavior for the player to choose how to play by clicking on an object
- Write functions inside a namespace to access it later from our others scripts
- Write a simple AI that check a game situation to make responds accordingly
- Superpowers and TypeScript API methods used:
   - Sup.Math.Ray()
   - Math.floor and Math.random
   - Sup.Actor()
   - Actor.addBehavior()
   - Sup.getActor().destroy()
   - Sup.setTimeout()
