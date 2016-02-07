# SUPERPOWERS TUTORIAL #1
## *Start to learn game development with PONG*
*Tutorial series : How to learn to make video games by using Superpowers #1.*

My plan is to provide a tutorial series of game development process as my
learning go and to share my adventures in Superpowers with the community.
Of course I am always eager to learn and this tutorials can always be improved,
this is why I stick to the open source philosophy of the community and let anyone
open issues and fix the tutorials to allow everyone to adopt them.

Two ways to help, you can address in this tutorial post your problems, propose change
and ask for tutorial clarity. Or if you feel comfortable with that, you can improve them
directly by submitting issues and propose pull requests for the markdown files in the [github repository][1].

Please, don't hesitate to say if you find mistakes, missing elements or unclear steps as it
would help to improve this tutorials aimed to the Superpowers community.

 Thank you and have fun making games !

*SUPERPOWERS TUTORIAL #2 will be a Tic Tac Toe with IA step by step tutorial*

## Summary

1. [Introduction and Plan the game](ch1.md)
    * [Introduction to game development](ch1.md#chapter-1--introduction-and-plan-the-game)
    * [Plan of our game](/ch1.md#plan-the-game)
2. Preparing Superpowers
    * Building project structure
    * Loading Game Assets
    * Building Game Scene
    * Setting Arcade Bodies
3. Scripting game behavior
    * Behavior of the paddles.
    * Behavior of the ball.
    * Display score points.
4. Polishing the game
    * Load Menu Assets
    * Build Menu Scene
    * Add sound effect.
    * Finishing the game.
5. Final recommendations about this tutorials series

## What we will learn in this tutorial

- To discover how easy it is to become a game developer (it is just about doing it)
- How to setup a first game project
- How to create a simple structure for a game and load assets
- How to build a scene with Actors and add components (camera, sprite renderer, text renderer, behavior)
- Code a game with the superpowers API and TypeScript
    - Sup.Actor
    - Sup.Input
    - Sup.ArcadePhysics2D.Body (Sup.ArcadeBody2D)
    - Sup.getActor( string ).getChild( string )
    - Sup.Math.Ray
    - Audio.playsound
    - Sup.loadScene

[1]:https://github.com/mseyne/superpowers-tutorials
