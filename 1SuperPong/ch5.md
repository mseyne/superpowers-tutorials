# SUPERPOWERS TUTORIAL #1 SUPER PONG
## *Chapter 5 : Final considerations about the tutorials and the learning process of making video games*

### The first step and the last step

When you finish this tutorial, you maybe think than you still don't know how to make a Pong game,
than you just copied what the tutorial said. Well, you are not really right and you are not really wrong.
 If you understood a bit the steps, then take a deep breath, believe in yourself and
 start to remake it from scratch without looking the tutorial. Remaking it again using your own memory
 will help you to deeper the steps in your memory and have a clearer understanding of what you are doing.

Following the tutorial with little understanding is the first step of the learning process
and your adventure continue as your exploration go deeper, you need now to face alone a blank project
and start the Pong game again. If you are stuck, you can peek the tutorial for clue and continue by yourself,
it is good you use the tutorial like a documentation or a reference because what is important is you can
understand each steps and go through them by yourself as you understand better the whole process.
All the others game from next tutorial and building games in the future is just about adding complexity
and features, crafting slowly worlds than player can explore and have fun with.

Even if you have built Pong and understood the steps without looking too much at the tutorial, you maybe don't know what to do next.
If you want to go to the next step of your adventure in game development, you can start to build a new game
by your own, read code of others or follow an other tutorial and follow it until you understand it. And continue
this way until you feel comfortable enough with Superpowers and programming with TypeScript.

The purpose of this series is to propose more complex tutorials as we advance to help us to learn how to climb this big mountain
until we feel strong enough to build by ourself this fantastic, challenging and emotionally rich video games.

See you soon if you decide you want to continue the adventure of game development with Superpowers.


### Complete Game Source Reference :

**Assets files of this game :**

* background.png
* pad1.png
* pad2.png
* ball.png
* tac.mp3
* toc.mp3
* tada.mp3
* menu/title.png
* menu/starton.png
* menu/startoff.png

**Project Assets Structure :**

Menu (Startup Scene)
* MenuSprites
    * title
    * starton
    * startoff
* MenuScripts
    * button
Game
* GameSprites
    * background
    * pad1
    * pad2
    * ball
* GameScripts
    * ball
    * paddles
* GameSounds
    * tac
    * toc
    * tada
Font

**Scripts Sources :**

* ball.ts

```TypeScript
const BALLSPEED : number = 0.05 ;

class BallBehavior extends Sup.Behavior {
  // Connect the actor body to a constant
  ball = this.actor.arcadeBody2D;
  // An Array with score[0] for player 1 and score[1] for player 2
  score = [0, 0];
  // speed variable
  speed : number = BALLSPEED;
  // set positive or negative direction variables of x and y
  dx : number = 1; dy : number = 1;

  update() {
    // get the ball position of x and y
    let x : number  = this.actor.getX(); let y : number  = this.actor.getY();

    // change direction of y if ball reach up and down sides
    if(y > 2.85 || y < -2.85){
      Sup.Audio.playSound("GameSounds/toc");
      this.dy = this.dy * -1;
    }

    // We check if there is collision between the ball and the two paddles
    if(Sup.ArcadePhysics2D.collides(this.ball, Sup.ArcadePhysics2D.getAllBodies())){
       Sup.Audio.playSound("GameSounds/tac");
      /* If there is collision we check if the ball touch
      a left or right side (we change the x direction) or
      the up and down side of the paddles (y direction),
      the speed of the ball take an acceleration. */
      if(this.ball.getTouches().right || this.ball.getTouches().left){
        this.dx = this.dx * -1;
        this.speed += 0.01
        }
      else {
        this.dy = this.dy * -1;
        }
      }

    /* We check if the ball pass the paddle and go beyond
    (sides left and right of the game table) if yes, we move
    the ball to the center and change the direction of x axis,
    the speed take the default speed. */
    if(x > 4 || x < -4){
       this.ball.warpPosition(0, 0);
       this.dx = this.dx * -1;
       this.speed = BALLSPEED;
       Sup.Audio.playSound("GameSounds/tada");
       }

    //We change the score depending on which side the ball go on x axis
    if(x > 4){
      ++this.score[0];
      Sup.getActor("Player1").getChild("Score").textRenderer.setText(this.score[0]);
    }

    if(x < -4){
      ++this.score[1];
      Sup.getActor("Player2").getChild("Score").textRenderer.setText(this.score[1]);
    }

    // set ball movement velocity speed*direction for x and y axis (the ball stay in movement at any time)
    this.ball.setVelocity(this.speed*this.dx, this.speed*this.dy);

    if(this.score[0] == 10 || this.score[1] == 10){
      Sup.loadScene("Menu");
    }
  }
}
Sup.registerBehavior(BallBehavior);
```

* paddles.ts

```TypeScript
class Paddle1Behavior extends Sup.Behavior {
  // connect the paddle body to a variable which will be used many times in the script
  pad = this.actor.arcadeBody2D;

  // set the speed of the paddle
  speed : number = 0.1;

  update() {
    // get Y position of paddle in a variable
    let y : number = this.actor.getY();

    // if the key is pressed and y < to max, the velocity of the body is set in motion with speed
    if(Sup.Input.isKeyDown("W") && y < 2.35){
      this.pad.setVelocityY(this.speed);
    }
    // if the key is pressedand y > to min, the velocity of the body is set in motion with negative speed
    else if(Sup.Input.isKeyDown("S") && y > -2.35){
      this.pad.setVelocityY(-this.speed);
    }
    // in other situations the velocity of the body is set to 0
    else{
      this.pad.setVelocityY(0);
    }
  }
}

class Paddle2Behavior extends Sup.Behavior {
  // connect the paddle body to a variable which will be used many times in the script
  pad = this.actor.arcadeBody2D;

  // set the speed of the paddle
  speed : number = 0.1;

  update() {
    // get Y position of paddle in a variable
    let y : number = this.actor.getY();

    // if the key is pressed and y < to max, the velocity of the body is set in motion with speed
    if(Sup.Input.isKeyDown("UP") && y < 2.35){
      this.pad.setVelocityY(this.speed);
    }
    // if the key is pressedand y > to min, the velocity of the body is set in motion with negative speed
    else if(Sup.Input.isKeyDown("DOWN") && y > -2.35){
      this.pad.setVelocityY(-this.speed);
    }
    // in other situations the velocity of the body is set to 0
    else{
      this.pad.setVelocityY(0);
    }
  }
}

Sup.registerBehavior(Paddle1Behavior);
Sup.registerBehavior(Paddle2Behavior);
```

* button.ts

```TypeScript
// We initialize a variable ray which is of type Math.Ray
var ray : Sup.Math.Ray;

class ButtonBehavior extends Sup.Behavior {
  // flag to tell when the mouse hover the button
  isHover : boolean = false;

  awake() {
    ray = new Sup.Math.Ray(this.actor.getPosition(), new Sup.Math.Vector3(0, 0, -1));
  }

  /* We define different possible actions of the mouse,
  click action load the game scene, hovering and unhovering
  make the button to change the sprite.*/

  mouse(action) {
    if(action == "click"){
      Sup.loadScene("Game");
      Sup.Audio.playSound("GameSounds/toc");
    }
    else if(action == "hover"){
      Sup.getActor("Button").spriteRenderer.setSprite("MenuSprites/starton");
    }
    else if(action == "unhover"){
      Sup.getActor("Button").spriteRenderer.setSprite("MenuSprites/startoff");
    }
  }

  update() {
    // Refresh position of the mouse in the camera
    ray.setFromCamera(Sup.getActor("Camera").camera, Sup.Input.getMousePosition());

    /* Condition to check if yes or no, the mouse hover
    the button, and if yes, check if the mouse click.
    We call the mouse function with the action related. */

    if(ray.intersectActor(this.actor, false).length > 0){
      if(!this.isHover){
        this.mouse("hover");
        this.isHover = true;
      }
      if(Sup.Input.wasMouseButtonJustPressed(0)){
        this.mouse("click")
      }
    }
    else if(this.isHover){
      this.isHover = false;
      this.mouse("unhover")
    }

  }
}
Sup.registerBehavior(ButtonBehavior);
```
