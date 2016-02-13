# SUPERPOWERS TUTORIAL #1 SUPER PONG
## *Chapter 3 : programming the game logic*

### Introduction

If you don't know nothing about Programming in Javascript (which TypeScript is based on)
you will maybe need to look at a tutorial to have a first idea (like the track JavaScript on Khan Academy).
Don't worry, coding/programming is not as difficult as we could imagine, it is about
solving real problems with logic and not much about abstraction, it is important to don't
go fast and take the time to understand basic concepts. You could feel it is really
slow and abstract to get a really simple thing, but it will go faster and more concrete
as you understand by practicing this steps. (The frustration come often when we try
  to do something we wasn't prepared to do)

However you can continue the tutorial and try to understand the basics by jumping
right now and copying the code below in Superpowers, by repetition and practice alone,
exploring and testing, making mistakes and repairing them (which we call debugging,
a fundamental concept in programming, where errors and mistakes are not to be avoid
but understood) slowly but confidently, you will do amazing progress.

### Scripting the paddles behaviors

We can now start to code the logic of our game, we will start to implement the movement of the paddles.

In our folder GameScripts, we will start to code in the script Paddles. You will have a default script,
we remove all and we write instead this template :

```TypeScript
class Paddle1Behavior extends Sup.Behavior {

  update() {

  }
}

class Paddle2Behavior extends Sup.Behavior {

  update() {

  }
}

Sup.registerBehavior(Paddle1Behavior);
Sup.registerBehavior(Paddle2Behavior);
```

We will work in the first Class Paddle1Behavior, the second Class Paddle2Behavior will be almost the same.

We define variables for the Class, **pad** which will be a connection to the body of our
scene and a number **speed**. We write under the Class line :

```TypeScript
class Paddle1Behavior extends Sup.Behavior {
    // connect the paddle body to a variable which will be used many times in the script
    pad = this.actor.arcadeBody2D;
    // set the speed of the paddle
    speed : number = 0.1;
[...]
```

We will use a condition **if** and the command **Sup.Input** which is important in Superpowers
to be able to get user input, which is the way player can have control in the game with
the mouse, the keyboard, the gamepad or his fingers on mobile.

We need a way to say to the sprite body to move if we press a keyboard key. We use the command
setVelocityY (because for the paddle we will move only on the y axis).

We also need to check if the paddle don't go out of the screen and we need to limit
maximum and minimun until which the paddle can go up and down. To do so, we will
keep track of the Y position of our paddle and add in our condition the possibility
to move the paddle only if the y is under a maximum y and above a minimum y.

In the **update** method we write our script as follow.

```TypeScript
[...]
update() {

    // get Y position of paddle in a variable
    let y : number = this.actor.getY() ;

    // if the key W is pressed and y < to max, the velocity of the body is set in motion with speed
    if(Sup.Input.isKeyDown("W") && y < 2.35){
      this.pad.setVelocityY(this.speed);
    }
    // if the key S is pressed and y > to min, the velocity of the body is set in motion with negative speed
    else if(Sup.Input.isKeyDown("S") && y > -2.35){
      this.pad.setVelocityY(-this.speed);
    }
    // in other situations the velocity of the body is set to 0
    else{
      this.pad.setVelocityY(0);
    }

  }
[…]
```

To try our script we need to attach it to our Actor in the Scene. In the paddle of
player1 with create a new component **Behavior** and we choose the class Paddle1Behavior.

![behavior.png](img/behavior.png)

We can now start our program and we should be able to see the paddle moving up and
down with the key W and S.

We can copy our code for the second Paddle in the Class Paddle2Behavior, but instead
of W and S we change the Sup.Input.isKeyDown to 'UP' and 'DOWN'.

In the same way than before, we attach the script in the Scene Game to the paddle of player 2.
(new component>Behavior), we choose the class Paddle2Behavior.


### Scripting the ball behavior

We can now start to code the movements of the ball in the asset script Ball in the folder
GameScripts. This Script will be contain more than the script paddles because we will
write the collisions and the scores systems inside the same Class.

We can remove from the default template the start method than we won't use in
this game, we should have now as a starter script :

```TypeScript
class BallBehavior extends Sup.Behavior {

  update() {

  }
}
Sup.registerBehavior(BallBehavior);
```
Because our ball will have accelerations, we can add now the default speed, which
will be a constant than we will refer to. We put our constant out of the main Class,
in won't matter in this game, but in other game, this constant could be then used in
other Class of the game, it is a good habit to get used to.

```TypeScript
const BALLSPEED : number = 0.05 ;
class BallBehavior extends Sup.Behavior {
[…]
```

We had now in our call BallBehavior our variables, one for the speed of the ball
which will accelerate, starting from our constant BALLSPEED, one Array which contain
both scores of player1 and player2 and two variables that will be flags positive or
negative which will give us the direction of the ball, by example if the ball go up
and touch the up side of the game table, the variable will take the oposite value,
telling us than the ball take the oposite direction.


```TypeScript
[…]
class BallBehavior extends Sup.Behavior {
    // speed variable
    speed : number = BALLSPEED;
    // Connect the actor body to a constant
    ball = this.actor.arcadeBody2D;
    // An Array with score[0] for player 1 and score[1] for player 2
    score = [0, 0];
    // set positive or negative direction variables of x and y
    dx : number = 1; dy : number = 1;
[…]
```

We write different conditions for different cases :

1. If the ball touch the up and down sides of the table, the ball change of direction
on y axis and continue to move on the same x axis. We will just check a condition and if true, change the variable dy.

2. If the ball collide with the paddles, the ball take a little acceleration and go in
the opposite direction on x axis while stay the same on y axis (if the ball collide with left and right side of the paddle).
We will use the **Sup.ArcadePhysics2D.collides** method to check if the Physics Bodies
of the ball and the paddles collides together. And a second check of the side of the ball collision with the method **getTouches**.

3. If the ball touch the right and left side (when the paddle miss the ball), a
score is made for the player who made the goal (we will see this point a bit later),
 the ball return to the center and take back the default speed.

Ok, now we write the behavior of the ball inside the loop method update.

```TypeScript
[…]
  update() {

    // get the ball position of x and y

    let x : number  = this.actor.getX(); let y : number  = this.actor.getY();

    // change direction of y if ball reach up and down sides

    if(y > 2.85 || y < -2.85){
      this.dy = this.dy * -1;
    }

    // We check if there is collision between the ball and the two paddles

    if(Sup.ArcadePhysics2D.collides(this.ball, Sup.ArcadePhysics2D.getAllBodies())){

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
       }

    // set ball movement velocity speed*direction for x and y axis (the ball stay in movement at any time)

    this.ball.setVelocity(this.speed*this.dx, this.speed*this.dy);

  }
[…]
```


To see the script in action we need, in the same way than before, to attach the
script Ball in the Scene Game to the Actor Ball. (new component>behavior, class = BallBehavior)

We should have now a game working nicely.


### Score system

We will now add a simple system of score. What we want is that player who get the
ball in the other camp mark one point. The score increment of one.

To do so, we add two conditions at the end of our class BallBehavior, one for the
case the ball touche the side of player 1 and one for the side of player 2.

We use then the **textRenderer** method to change the display of the score that we connect with
the method getActor('Player1').getChild('Score') for the score of player 1 and getActor('Player2').getChild('Score')
for the score of player 2.

```TypeScript
[...]
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
  }
}
Sup.registerBehavior(BallBehavior);
```

The game logic is finished, we just need to polish a bit before release but we have now our game.
