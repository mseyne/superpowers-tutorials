# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 6


### **Writing Alien and Asteroids behaviors**

Since we now have a basic structure for our game, we can start to build step by step our game objects, we will start in this chapter to implement the Alien ship and asteroids mechanics then
we will do the next chapter on the player ship behavior.

#### Alien Behavior

##### Alien datas

We first initialize the local datas we need for the alien ship actor in the Alien/Alien script.

```ts
class AlienBehavior extends Sup.Behavior {
  // Ship size of this Actor
  size: number;
  // Ship amplitude of this Actor
  amplitude: number;
  // Ship position
  position: Sup.Math.Vector2;
  // Ship linear movement
  velocity : Sup.Math.Vector2;
  // Timer before shooting
  shootCooldown: number;
  // Timer before death
  deathTimer: number;
  [...]
}
Sup.registerBehavior(AlienBehavior);
```

##### Alien awakening and closing

We first give life to our Alien by giving a true to the flag alive when the behavior wake up. 
We choose randomly an index to get a different size (with related amplitude) each new alien ship and set it to the Actor.
And we set some value to start, the lifes, the shooting timer and the checkLifeHUD flag which give a signal for an other process in the game to update the HUD with the new alien life. (We will write this process later)

We also want that alive turn false when the alien ship is destroyed, there is a default method onDestroy() we can use. We also give to the spawn time before a new alien ship appear the default Respawn time.


```ts
[...]
  awake() {
    // Set Alien.alive flag to true
    Alien.alive = true;
    // Get a random index for alien ship size and amplitude
    let randomShip: number = Sup.Math.Random.integer(0, 2);
    // Get the default size related to the random index
    this.size = Alien.sizes[randomShip];
    // Get the default collision amplitude related to the random index
    this.amplitude = Alien.amplitudes[randomShip];
    // Set the size to the actor
    this.actor.setLocalScale(this.size);
    // Set the value of Alien.startLife to the current Alien.lifes
    Alien.lifes = Alien.startLife;
    // Set the shootCooldown timer to the Alien.shootTime value
    this.shootCooldown = Alien.shootTime;
    // Set the Game.checkLifeHUD value to true 
    Game.checkLifeHUD = true;
  }

  update() {
  }

  onDestroy() {
      // Set Alien.alive flag to false
      Alien.alive = false;
      // Give to Alien.spawnTimer the value of Alien.respawnTime
      Alien.spawnTimer = Alien.respawnTime;
  }
[...]
```

##### Alien position and velocity on start

We create a new method start() where we will initialize the alien position and velocity instead to do it in the awake() method, 
the reason is that the in-built start method wait after the position in Game.spawnAlien() function from Global script to initialize a random position before to run.

```ts
[...]
  start() {
    // Set position to the current actor position
    this.position = this.actor.getLocalPosition().toVector2();
    // Set x axis linear movement toward the opposite edge of spawn
    if (this.position.x === Game.bounds.width / 2) {
      this.velocity = new Sup.Math.Vector2(-Alien.linearSpeed, 0);
    }
    else {
      this.velocity = new Sup.Math.Vector2(Alien.linearSpeed, 0);
    }
  }
[...]
```

##### Alien shooting

Before to write the process of the alien ship, we can set a little method which say what happen when the ship shoot a missile.

```ts
[...]
  shoot() {
    // Create a new alien missile and set the Missile actor to a variable
    let missile = Sup.appendScene("Alien/Missile/Prefab")[0];
    // Set the alien ship position to the missile actor position
    missile.setLocalPosition(this.position);
    // Reset value of Timer to default value
    this.shootCooldown = Alien.shootTime;
  }
[...]
```

##### Alien update loop

Now we write the complete behavior of the ship in the update loop. Here the complete process :

* Death timer : give some frames to finish the explosion animation before to destroy the actor.
* Death setting : Check if there is life left, if not, apply death to the actor and update score of player.
* Keep moving : Move the alien ship at each frame.
* Keep rotating : Rotate the alien ship at each frame.
* Keep shooting : With an interval of a timer, the ship shoot a missile. 
* Stay on the screen : When the ship reach an edge of the screen (inside the invisible border), it reappear on the other side.

```ts
[...]
  update() {
    // Death timer
    // Decrease death timer before destruction
    if(this.deathTimer > 0){
      this.deathTimer--;
      if(this.deathTimer === 0){
        this.actor.destroy();
      }
      return;
    }
    
    // Death setting
    // Set death if alien don't have lifes anymore
    if(Alien.lifes === 0){
      // Reset angles to default for the explosion sprite
      this.actor.setEulerAngles(0,0,0);
      // Set visible off the alien ship model
      this.actor.getChild("Model").setVisible(false);
      // Set the sprite animation explode to play once without looping
      this.actor.getChild('Destruction').spriteRenderer.setAnimation("explode", false);
      // Give 30 frames before actor destruction (half a second)
      this.deathTimer = 30;
      // Add point to the player for alien death
      Game.addPoints(true, Game.points.alien);
      // Flag the game to check and update the HUD score
      Game.checkScoreHUD = true;
    }
    
    // Keep moving
    // Add the linear movement to the position variable
    this.position.add(this.velocity);
    // Set the new position to the alien actor
    this.actor.setLocalPosition(this.position);
    
    // Keep rotating
    // Rotate actor using rotateLocalEulerY with the default Alien.rotationSpeed value
    this.actor.rotateLocalEulerY(Alien.rotationSpeed);
    
    // Keep shooting
    // If the timer is not finished, decrease value of one
    if (this.shootCooldown > 0) {
      this.shootCooldown--;
    }
    // If the timer is finished, shoot missile
    else {
      this.shoot();
    }
    
    // Stay on the screen
    // If position is superior to the max of the x bound then switch position
    if (this.position.x > Game.bounds.width / 2) {
      this.position.x = -Game.bounds.width / 2;
    }
    // If position is inferior to the min of the x bound then switch position
    if (this.position.x < -Game.bounds.width / 2) {
      this.position.x = Game.bounds.width / 2;
    }
    
    // Collision code will come here
  }
[...]
```

We have seen a new function for the **Global** script in the Alien process.

At this point this function can't work and will bug if we do test because we need to have the Ship behavior running.
For now, we can comment on the call of this function in the update loop of the alien ship, we will come back to the score and points at the end.

```ts
[...]
// // Add point to the player for alien death
// Game.addPoints(true, Game.points.alien);
[...]
```

We can now do test, the alien ship move, rotate and reappear each time it reach a side of the game screen. Also it shoot even if the missile do not have behavior for now.

#### Asteroid Behavior

##### Asteroid datas

Inside the Asteroid/Asteroid script, we first initialize the local datas than we will use in our script.

```ts
class AsteroidBehavior extends Sup.Behavior {
  // The size of the asteroid
  size: number;
  // The amplitude of this asteroid Actor
  amplitude: number;
  // The class of size this asteroid belong
  sizeClass: string;
  // The position of the asteroid
  position: Sup.Math.Vector3;
  // The movement of the asteroid
  velocity: Sup.Math.Vector3;
  // The movement of the asteroid on x axis
  linearSpeedX: number;
  // The movement of the asteroid on y axis
  linearSpeedY: number;
  // The rotation speed of the asteroid
  rotationSpeed: number;
  // Timer before destruction
  deathTimer: number;
[...] 
}
Sup.registerBehavior(AsteroidBehavior);
```

##### Asteroid awakening and closing

We set the awake and onDestroy method of asteroid, we need to add and remove it from the list of asteroids and we also need to keep track of the overall number.

```ts
[...]
  awake() {
    // Increase asteroids count by one
    Asteroids.currentCount++;
    // Add this actor to the Asteroids list
    Asteroids.list.push(this.actor);
  }

  update() {
  }

  onDestroy() {
    // Decrease asteroids count by one
    Asteroids.currentCount--;
    // Remove this actor from the Asteroids list
    Asteroids.list.splice(Asteroids.list.indexOf(this.actor), 1);
  }
[...] 
```

To make it work, we need to open the variables list and currentCount to the Game.start function in the Global script.

```ts
[...]
  // Start the game
  export function start(){
    // Set new asteroids list
    Asteroids.list = [];
    // Set asteroids number to 0
    Asteroids.currentCount = 0;
[...] 
```

##### Asteroid starting with velocity and size

When the asteroids has finish to spawn, we use the start method to get the random value of the current asteroid and set the velocity and size to the current asteroid actor.

```ts
[...]
    // Get the position of the actor set randomly when spawned
    this.position = this.actor.getLocalPosition();
    // Get random value for linear speed on axis X and axis Y
    this.linearSpeedX = Sup.Math.Random.float(Asteroids.linearSpeed.min, Asteroids.linearSpeed.max);
    this.linearSpeedY = Sup.Math.Random.float(Asteroids.linearSpeed.min, Asteroids.linearSpeed.max);
    // Set asteroid velocity with the linearSpeed values
    this.velocity = new Sup.Math.Vector3(this.linearSpeedX, this.linearSpeedY, 0);
    // Get random value for linear speed on axis X and axis Y
    this.rotationSpeed = Sup.Math.Random.float(Asteroids.rotationSpeed.min, Asteroids.rotationSpeed.max);
    // Set the asteroid size related to the classSize of the asteroid
    this.size = Asteroids.sizes[this.sizeClass];
    // Set the asteroid collision amplitude related to the classSize of the asteroid
    this.amplitude = Asteroids.amplitudes[this.sizeClass];
    // Set the size to the actor model
    this.actor.getChild("Model").setLocalScale(this.size);
    // Set the size to the destruction sprite
    this.actor.getChild("Destruction").setLocalScale(this.size * 2);
[...] 
```

##### Asteroid death method

We write the process of death of the asteroid in a death method than we will call later when the asteroid collide with a ship missile or the ship.

```ts
[...]
  die() {
    // Reset angles to default for the explosion sprite
    this.actor.setEulerAngles(0,0,0);
    // Set visible off the asteroid model
    this.actor.getChild("Model").setVisible(false);
    // Set the sprite animation explode to play once without looping
    this.actor.getChild('Destruction').spriteRenderer.setAnimation("explode", false);
    // Give 30 frames before actor destruction (half a second)
    this.deathTimer = 30;
    // We give the points to ship1 related to the classSize of this asteroid 
    if(this.sizeClass === "big") {
        Game.addPoints(true, Game.points.asteroidBig);
      }
      else if (this.sizeClass === "medium") {
        Game.addPoints(true, Game.points.asteroidMedium);
      }
      else {
        Game.addPoints(true, Game.points.asteroidSmall);
      }
    // Flag the game to check and update the HUD score
    Game.checkScoreHUD = true;
  }
[...] 
```

Same as before, we can comment the call of the function Game.addPoints that we will call later.

 
```ts
[...]
    if(this.sizeClass === "big") {
        // Game.addPoints(true, Game.points.asteroidBig);
      }
      else if (this.sizeClass === "medium") {
        // Game.addPoints(true, Game.points.asteroidMedium);
      }
      else {
        // Game.addPoints(true, Game.points.asteroidSmall);
      }
[...] 
```

##### Asteroid spawn children method

We write now a method that define what happen when the asteroid collide with a ship missile.

We want the asteroid to divide in two others asteroids and take the sizeClass down, from big to medium, from medium to small.

In code, we translate this as spawning two new asteroids and destroying the current one.

```ts
[...]
  spawnChildren() {
    // Create two now asteroid and set the Actor asteroid to each variable
    let asteroid1 = Sup.appendScene("Asteroid/Prefab")[0];
    let asteroid2 = Sup.appendScene("Asteroid/Prefab")[0];
    // Set the position of the new asteroids to the same position than the asteroid parent (this one)
    asteroid1.setPosition(this.position);
    asteroid2.setPosition(this.position);
    // If the sizeClass was big, then both asteroid are medium
    if (this.sizeClass === "big") {
      asteroid1.getBehavior(AsteroidBehavior).sizeClass = "medium";
      asteroid2.getBehavior(AsteroidBehavior).sizeClass = "medium";
      
    }
    // If the sizeClass was medium, then both asteroid are small
    if (this.sizeClass === "medium") {
      asteroid1.getBehavior(AsteroidBehavior).sizeClass = "small";
      asteroid2.getBehavior(AsteroidBehavior).sizeClass = "small";
    }
  }
[...] 
```

##### Asteroid main process

Very much like the asteroid update loop, we can now write our update loop as follow :
* Death timer : give some frames to finish the explosion animation before to destroy the actor.
* Keep moving : Move the asteroid at each frame.
* Keep rotating : Rotate the asteroid at each frame.
* Stay on the screen : When the asteroid reach an edge of the screen (inside the invisible border), it reappear on the opposite side.

```ts
[...]
  update() {
    // Death timer
    if(this.deathTimer > 0){
      this.deathTimer--;
        if(this.deathTimer === 0){
          this.actor.destroy();
        }
      return;
      }
    
    // Keep moving
    this.position.add(this.velocity);
    this.actor.setLocalPosition(this.position);
    
    // Keep rotating
    this.actor.rotateEulerAngles(this.rotationSpeed, this.rotationSpeed, this.rotationSpeed);

    // Stay on the screen
    // Check position of asteroid on x axis
    if (this.position.x > Game.bounds.width / 2) {
      this.position.x = -Game.bounds.width / 2
    }
    
    if (this.position.x < -Game.bounds.width / 2) {
      this.position.x = Game.bounds.width / 2
    }
    // Check position of asteroid on y axis
    if (this.position.y > Game.bounds.height / 2) {
      this.position.y = -Game.bounds.height / 2
    } 
    
    if (this.position.y < -Game.bounds.height / 2) {
      this.position.y = Game.bounds.height / 2
    }
    
    // Collision code will come here
  }
[...] 
```

Ok we now have asteroids moving around, rotating and staying on the game screen.

[<-- back to chapter 5](ch5.md) -- [go to chapter 7 -->](ch7.md)