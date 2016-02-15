# SUPERPOWERS TUTORIAL #2 SUPER OXO
## *Learn game development with a Tic Tac Toe game*

### Summary

1. [Introduction](ch1.md#chapter-1--introduction)
   * [Planning the game](ch1.md#planning-the-game)
   * [MVG Features](ch1.md#mvg-features)
2. [Building the Game Structure](ch2.md#chapter-2-building-the-game-structure)
   * [Loading the game assets](ch2.md#loading-game-assets)
   * [Building the game scene](ch2.md#building-the-game-scene)
3. Writing the Player actions, Game logic part 1
   * Setting the global script
   * Scripting th mouse behavior
   * A turn based game
4. Writing the Computer response, Game logic part 2
   * Algorithm of a simple computer AI
   * Scripting the computer behavior
   * Scripting the victory checking
5. Polishing the Game
   * Create a victory screen
   * Restart the game
   * Randomize the game starting
   * Slowing down the game
6. Complete Game Source Reference
   * Assets files
   * Project structure
   * Scripts sources

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
