# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 8

### **Writing Collision, Life and Score systems.**

To give a substance to all our actors and give the main gameplay to our game we need to set the collision system without which the actors would never interact to each other.

#### Collision system

##### The Game.checkCollisionMissile function

We add a new function that we will call each time we need to check a collision with a missile list, we put it in the Module Game of the Global script.

```ts
namespace Game{
[...] 
  export function checkCollisionMissile(actor: Sup.Actor, actorList: Sup.Actor[], amplitude: number){
    /* 
    We get the distance between actor and all the actors of the actorList
    If the distance if inferior to the amplitude box around the Actor, there is a collision
    */
    
    // Get the position 1 of the actor we check collision with actorList
    let position1: Sup.Math.Vector2 = actor.getLocalPosition().toVector2();
    // Loop through the actors of listToCheck and check if ....
    for(let i = 0; i < actorList.length; i++){
      // Get the position 2 of the current actor of the loop inside actorList
      let position2: Sup.Math.Vector2 = actorList[i].getLocalPosition().toVector2();
      // Get the distance between position 1 and position 2
      let distance: number = Math.round(position1.distanceTo(position2)*100)/100;
      // If the distance is inferior to the amplitude (collision radius), then it is a collision
      if (distance < amplitude) {
        // The current actor of the actorList is destroyed
        actorList[i].destroy();
        // Return true to the behavior which check for collision
        return true;
      }
    }
    // Return false to the behavior which check for collision
    return false;
  }
}
[...]
```

##### The Game.checkCollisionAsteroid function

We add a new function that we will call each time we need to check a collision with an asteroid list (different amplitude for each asteroid)

```ts
namespace Game{
[...]   
  export function checkCollisionAsteroids(ship: Sup.Actor, shipAmplitude: number) {
    /*
      Get the distance between the ship and the asteroids, compare the distance with the biggest amplitude
    */
    // Initialize a variable asteroid which will take all the asteroids of the list as value
    let asteroid: Sup.Actor;
    // Get the current position of the ship
    let shipPosition: Sup.Math.Vector2 = ship.getLocalPosition().toVector2();
    // Loop through all the asteroids in game
    for(asteroid of Asteroids.list){
      // Get the amplitude of the current asteroid
      let asteroidAmplitude: number = asteroid.getBehavior(AsteroidBehavior).amplitude;
      // Get the position of the current asteroid
      let asteroidPosition: Sup.Math.Vector2 = asteroid.getLocalPosition().toVector2();
      // Convert to distance between ship and current asteroid positions
      let distance: number = Math.round(shipPosition.distanceTo(asteroidPosition)*100)/100;
      // Check if distance is less than the biggest amplitude 
      if (distance < shipAmplitude || distance < asteroidAmplitude) {
        // Destroy the current asteroid
        asteroid.getBehavior(AsteroidBehavior).die();
        // Return true, the ship is destroyed too
        return true;
      }
    }
    return false;
  }
}
[...]
```

##### The Game.checkCollisionShip function

We add a new function that we will call each time we need to check a collision with an other ship (alien or other player)

```ts
namespace Game{
[...]   
  // Check collision between actor1 and actor2
  export function checkCollisionShip(actor1: Sup.Actor, actor1Amplitude: number) {
    // Initialize variable for the actors
    let actor2: Sup.Actor;
    let actor1Position: Sup.Math.Vector2;
    let actor2Position: Sup.Math.Vector2;
    let actor2Amplitude: number;
    let actor2Alive: boolean;
    // If the actor1 is the Ship1, then we check the Ship2 as actor2
    if (actor1.getName() === "Ship1"){
      actor2 = Sup.getActor("Ship2");
    }
    // Else, if the actor1 is Ship2 or Alien Ship, we check the Ship1 as actor2
    else {
      actor2 = Sup.getActor("Ship1");
    }
    // Get the positions of both actors
    actor2Position = actor2.getLocalPosition().toVector2();
    actor1Position = actor1.getLocalPosition().toVector2();
    // Get amplitud of actor 2
    actor2Amplitude = Ships.amplitude;
    // Get the life status of the actor 2
    actor2Alive = actor2.getBehavior(ShipBehavior).alive;
    // Get the distance between the two actors
    let distance: number = Math.round(actor1Position.distanceTo(actor2Position)*100)/100;
    // To avoid to collide when ship is blinking with invulnerability, return false the collisionChecking if actor 2 not alive
    if(!actor2Alive){
      return false;
    }
    // Check if distance if less than the biggest amplitude (or collision radius)
    if (distance < actor1Amplitude || distance < actor2Amplitude) {
        // Destroy the current asteroid
        actor2.getBehavior(ShipBehavior).die();
        // Return true, both actor are destroyed
        return true;
      }
    // If condition not met, return false
    return false
  }
}
[...]
```

##### Collision alien ship with player missile

Inside the update loop of the **Alien** script, we call a Game.checkCollisionMissile with the ship missiles of the player.

```ts
[...]  
  update() {
  [...]
    // If there is player ship1 missiles, call a collision checking for this actor with the list of the player Ship missiles
    if(Ships.missiles[0].length > 0){
      // If the collision checking return true
      if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)) {
        // Decrease Alien lifes by one
        Alien.lifes--
        // Flag true to check and update the life HUD in Game Script
        Game.checkLifeHUD = true;
      }
    }
  }
[...]
```

##### Collision asteroid with player missile

Inside the update loop of the **Asteroid** script, we call a Game.checkCollisionMissile with the ship missiles of the player. If the collision is true the asteroid is destroyed and two new asteroids spawn.

```ts
[...]
  update() {
  [...]
    // If there is player ship1 missiles, call a collision checking for this actor with the list of the player Ship missiles
    if(Ships.missiles[0].length > 0){
      // If the collision checking return true
      if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)) {
        // Destroy this asteroid
        this.die();
        // If the asteroid is not small
        if (this.sizeClass !== "small"){
          // Spawn two new asteroid of the class below
          this.spawnChildren();
        }
      }
    }
  }
[...]
```

##### Collision player ship with asteroids

Inside the **Ship** script, if the game is Super Asteroids we now check for collision betwen the ship and the alien missiles and all the asteroids in the update loop.

If the game is Super Spacewar we check collision with the missile of other ship or a collision with the other ship.


```ts
[...]
  update() {
  [...]
    // If game is Super Spacewar, check collision with other missiles and other ship
    if (Game.nameIndex === 0) { 
      // Check collision between the player ship and the asteroids
      if (Game.checkCollisionAsteroids(this.actor, this.amplitude)){
        this.die();
      }
    }
[...]
```

##### Collision player ship with alien missile

Still in the **Ship** script, in the same block that check collision in the case the game is Asteroids (index 0).

```ts
[...]
  update() {
  [...]
    if (Game.nameIndex === 0) {
    [...]
      // Check collision between the player ship and the alien missiles
      if (Game.checkCollisionMissile(this.actor, Alien.missiles, this.amplitude)){
        this.die();
      }
    }
[...]
```

##### Collision player ship with alien ship

In the **Alien** script, we add a call of the Game.checkCollisionShip function.

```ts
[...]
  update() {
  [...]
    // Check collision between Alien ship and Player ship
    if (Game.checkCollisionShip(this.actor, this.amplitude)){
      // Destroy the alien ship by setting its lifes to 0
      Alien.lifes = 0;
    }
    }
[...]
```

##### Collision player ship with other player missiles

In the **Ship** script we add an other block of checking in the case the game is Spacewar (index 1)

```ts
[...]    
    // If game is Super Spacewar, check collision with other missiles and other ship
    if (Game.nameIndex === 1) {
      // If the ship is 1 check collision with ship 2 missile
      if (this.index === 0){
        if (Game.checkCollisionMissile(this.actor, Ships.missiles[1], this.amplitude)){
          // if there is collision, add point to the ship 2 and destroy ship 1
          Game.addPoints(false, Game.points.ship);
          this.die();
        }
      }
      // Else the ship is 2 check collision with ship 1 missile
      else {
        if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)){
          // if there is collision, add point to the ship 1 and destroy ship 2
          Game.addPoints(true, Game.points.ship);
          this.die();
        }
      }
    }
[...]
```

*Note : To do test, we need to comment the Game.addPoints function.*

##### Collision player ship with other player ship

```ts
[...]
    if (Game.nameIndex === 1) {
        [...]
      // Check collision between this ship and the other ship
      if (Game.checkCollisionShip(this.actor, this.amplitude)) {
            this.die();
      }
    }
[...]
```

The collision work now fine, we need now to update the Game HUD when something happen in the game.

#### Score system

##### Global script : Game.addPoints function

We can now create the function that addPoints to the player score, to make the difference between the score of ship 1 and ship 2, we simply use a boolean parameter, when
it is true, we add points to ship 1 else, we add them to ship 2.

```ts
namespace Game{
  [...]
  // Add points to the ship score
  export function addPoints(ship1: boolean, points: number){
    // If ship1 is true, add points to ship1
    if(ship1){
      Sup.getActor("Ship1").getBehavior(ShipBehavior).score += points;
    }
    // If ship1 is false, add points to ship2
    else {
      Sup.getActor("Ship2").getBehavior(ShipBehavior).score += points;
    }
  }
}
[...]
```

*Note : to make the function work in all our game, we need to uncomment every place we have a Game.addPoints function in ou game. In the Ship script, the asteroid script and the alien script.*


##### Global script : Game.updateHUDScore

Inside the **Global** script, we add a function updateHUDScore update in the Game module.

```ts
namespace Game{
  [...]
  // Update the score HUD with the score for ship 1 or optionally for ship2
  export function updateHUDScore(Score1:number, Score2?:number) {
    // Change the score text displayed on HUD with the new score
    Sup.getActor("HUD").getChild("UIShip1").getChild("Score").textRenderer.setText(Score1);
    // If there is a score 2, do the same for the HUD of ship 1 and ship 2
    if(Score2){
        Sup.getActor("HUD").getChild("UIShip1").getChild("Score").textRenderer.setText(Score1);
        Sup.getActor("HUD").getChild("UIShip2").getChild("Score").textRenderer.setText(Score2);
    }
    // Set flag to false
    Game.checkScoreHUD = false;
  }
}
[...]
```


##### Game script : checkHUDScore

Now, in the Game script we simply check the flag checkHUDScore to see when to call the Game.updateHUDScore function.
```ts
[...]
  update() {  
    // Check if score need to be updated with the HUD
    if (Game.checkScoreHUD) {
      // If the game is spacewar, we update the two ship scores
      if (Game.nameIndex == 1) {
        let player1Score = Sup.getActor("Ship1").getBehavior(ShipBehavior).score;
        let player2Score = Sup.getActor("Ship2").getBehavior(ShipBehavior).score;
        Game.updateHUDScore(player1Score, player2Score);
      }
      // Else the game is asteroids, we update the ship 1 score
      else {
        let playerScore = Sup.getActor("Ship1").getBehavior(ShipBehavior).score;
        Game.updateHUDScore(playerScore);
      }
    }
  }
[...]
```

#### Life system

#### Global script : Game.updateHUDLife

Inside the **Global** script, we add a function updateHUDLife update in the Game module.

```ts
namespace Game{
  [...]  
  // Update the life HUD
  export function updateHUDLife(life1:number, life2:number){
    /*
    Explanation of life 
    */
    // If the game is Asteroids
    if (Game.nameIndex === 0){
      // Update the player Ship life
      for (let i = 0; i < Ships.startLife; i++) {
        let heart = Sup.getActor("HUD").getChild("UIShip1").getChild("Life").getChild(i.toString());
        heart.spriteRenderer.setAnimation(hearts[life1][i]);
      }

      // Update the alien life
      for (let i = 0; i < Alien.startLife; i++) {
        let heart = Sup.getActor("HUD").getChild("UIAlien").getChild(i.toString());
        heart.spriteRenderer.setAnimation(hearts[life2][i]);
      }      
    }
    
    if (Game.nameIndex === 1){
      // player1 life
      for (let i = 0; i < Ships.startLife; i++) {
        let heart = Sup.getActor("HUD").getChild("UIShip1").getChild("Life").getChild(i.toString());
        heart.spriteRenderer.setAnimation(hearts[life1][i]);
      }
      
      // player2 life
      for (let i = 0; i < Ships.startLife; i++) {
        let heart = Sup.getActor("HUD").getChild("UIShip2").getChild("Life").getChild(i.toString());
        heart.spriteRenderer.setAnimation(hearts[life2][i]);
      }
    }
    // Set false to flag checkLifeHUD
    Game.checkLifeHUD = false;
  }
}
[...]
```

##### Game script : checkHUDlife

Now, in the Game script we simply check the flag checkHUDLife to see when to call the Game.updateHUDLife function.
```ts
[...]
  update() {  
    // Check if lifes need to be updated with the HUD
    if (Game.checkLifeHUD) {
      if (Game.nameIndex === 0) {
        let playerLife = Sup.getActor("Ship1").getBehavior(ShipBehavior).lifes;
        let alienLife = Alien.lifes;
        Game.updateHUDLife(playerLife, alienLife);
      }
      if (Game.nameIndex === 1) {
        let player1Life = Sup.getActor("Ship1").getBehavior(ShipBehavior).lifes;
        let player2Life = Sup.getActor("Ship2").getBehavior(ShipBehavior).lifes;
        Game.updateHUDLife(player1Life, player2Life);
      }
    }
  }
[...]
```

The HUD working fine, we need to work the menu screen inside our game mechanic to be also able to conclude the game when the ship have no more life or the timer is out with a game over screen.
Let's finish our game with the next chapter. 

[<-- back to chapter 7](ch7.md) -- [go to chapter 9 -->](ch9.md)