# SUPERPOWERS TUTORIAL #1 SUPER PONG
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

### Summary

1. [Introduction and Plan the game](ch1.md#chapter-1--introduction-and-plan-the-game)
    * [Introduction to game development](ch1.md#chapter-1--introduction-and-plan-the-game)
    * [Plan of our game](ch1.md#plan-the-game)
2. [Preparing Superpowers](ch2.md#chapter-2--preparing-superpowers)
    * [Building the project structure](ch2.md#building-the-project-structure)
    * [Loading Game Assets](ch2.md#loading-the-game-assets)
    * [Building Game Scene](ch2.md#building-the-scene)
    * [Setting Arcade Bodies](ch2.md#setting-arcade-bodies)
3. [Programming the game logic](ch3.md#chapter-3--programming-the-game-logic)
    * [Introduction to programming](ch3.md#introduction)
    * [Behavior of the paddles](ch3.md#scripting-the-paddles-behaviors)
    * [Behavior of the ball](ch3.md#scripting-the-ball-behavior)
    * [Display score points](ch3.md#score-system)
4. [Polishing the game](ch4.md#chapter-4--polishing-the-game)
    * [Load Menu Assets](ch4.md#build-the-menu-structure-and-load-assets-files)
    * [Build Menu Scene](ch4.md#build-the-menu-scene)
    * [Scripting a Button](ch4.md#scripting-a-button)
    * [Add sound effect](ch4.md#adding-sound)
    * [Finishing the game](ch4.md#adding-an-end-to-the-game)
5. [Final recommendations about this tutorial series](ch5.md#chapter-5--final-considerations-about-the-tutorials-and-the-learning-process-of-making-video-games)
    * [Learning game development](ch5.md#the-first-step-and-the-last-step)
    * [Complete Game Source Reference](ch5.md#complete-game-source-reference)

### What we will learn in this tutorial

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
- To discover than now, we are a game developer and than we can learn anything

[1]:https://github.com/mseyne/superpowers-tutorials
