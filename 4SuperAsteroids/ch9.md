# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 9

### **Setting menu and game over screens**

We will now set the menu which is the startup scene with the buttons that allow us to choose what game we want to play.

#### Menu behavior

##### Menu datas
We start to write the MenuBehavior by initializing the datas we need in the Menu script.

```ts
class MenuBehavior extends Sup.Behavior {
  // The current menu screen
  screen: string;
  // Ray casting
  ray = new Sup.Math.Ray();
  // First button
  button0: Sup.Actor;
  // Second button
  button1: Sup.Actor;
  awake() {
  }
  update() {
  }
}
Sup.registerBehavior(MenuBehavior);
```

*Note : To do test with the menu, we have to change the Startup Scene in the Settings with Menu/Scene.* 

##### Awakening the setScreen method

When the menu awake the first time, we want it to set the Main screen. We need to write a method setScreen and call this function in the awake method of the MenuBehavior.

```ts
[...]
  awake() {
      // Call the function setScreen
      this.setScreen(Menu.screens.main);
  }
  
  setScreen(screenName: string){
    // Set the current screen to the screen string Name
    this.screen = screenName;
    // Loop through all the screen of the scene and set visible the current screen
    for(let screen in Menu.screens){
      // If the screen of the loop is the same as the current screen
      if (Menu.screens[screen] == this.screen){
        // Set visible true the screen
        Sup.getActor("Screens").getChild(Menu.screens[screen]).setVisible(true);
      }
      // Else, hide the screen, set visible false
      else {
        Sup.getActor("Screens").getChild(Menu.screens[screen]).setVisible(false);
      }
    }
    // call the function updateTitle
    this.updateTitle();
  }
  
  updateTitle(){
    // Set title visible true
    Sup.getActor("Title").setVisible(true);
    // Set score visible false
    Sup.getActor("Score").setVisible(false);
    // Change the title depending if the game is asteroids or spacewar
    if(this.screen === Menu.screens.asteroids|| this.screen === Menu.screens.spacewar){
      Sup.getActor("Title").getChild("Text2").textRenderer.setText(this.screen);
    }
    // Else, it change the title with an empty string
    else{
      Sup.getActor("Title").getChild("Text2").textRenderer.setText("");
    }
  }
[...]
```

##### Awakening the updateButtonsText method

We also need to update the buttons each time a screen is set.

```ts
[...]
  awake() {
    [...]
    // Update the buttons
    this.updateButtonsText(); 
  }
[...]  
  updateButtonsText() {
    // Get the two buttons actors in one variable each
    this.button0 = Sup.getActor("Buttons").getChild("Button1");
    this.button1 = Sup.getActor("Buttons").getChild("Button2");
    // If it is the Main screen
    if(this.screen === Menu.screens.main){
      // Set the button to display the game name
      this.button0.getChild("Text").textRenderer.setText("asteroids");
      this.button1.getChild("Text").textRenderer.setText("spacewar");
    }
    // For the other screens
    else {
      // Set the button to display start and return
      this.button0.getChild("Text").textRenderer.setText("start");
      this.button1.getChild("Text").textRenderer.setText("return");
    }
  }
[...]
```

##### Menu process

Now, in the update loop of the menu, we need to check the actions of the player with the mouse and the button and respond to them.

```ts
[...]
  update() {
    // Update the position of the raycaster of the mouse
    this.ray.setFromCamera(Sup.getActor("Camera").camera, Sup.Input.getMousePosition());
    // Return an object if the raycaster intersect the actor buttons
    let hitB1 = this.ray.intersectActor(this.button0.getChild("Sprite"));
    let hitB2 = this.ray.intersectActor(this.button1.getChild("Sprite"));
    
    // If the button 1 is hovered
    if(hitB1[0]){
      // Change the opacity of the button to full bright
      this.button0.getChild("Sprite").spriteRenderer.setOpacity(1);
      // If the left mouse button is pressed
      if (Sup.Input.wasMouseButtonJustPressed(0)) {
        // Differents case possible depending the current screen
        switch (this.screen){
          // If the current screen is the Main screen
          case Menu.screens.main:
            // Set the new screen to Asteroids screen
            this.setScreen(Menu.screens.asteroids);
            break;
          // If the current screen is the Asteroid screen
          case Menu.screens.asteroids:
            // Set the nameIndex to 0
            Game.nameIndex = Menu.names.Asteroids;
            // Start the game
            Game.start();
            return;
          // If the current screen is the Spacewar screen
          case Menu.screens.spacewar:
            // Set the nameIndex to 1
            Game.nameIndex = Menu.names.Spacewar;
            // Start the game
            Game.start();
            return;
          // If the current screen is the Game Over screen
          case Menu.screens.gameover:
            // Restart the game
            Game.start();
            return;
        }
      }
    }
    // If the button is not hovered, set opacity half bright
    else{
      this.button0.getChild("Sprite").spriteRenderer.setOpacity(0.5); 
    }
    
    // If the button 2 is hovered
    if(hitB2[0]){
      // Change the opacity of the button to full bright
      this.button1.getChild("Sprite").spriteRenderer.setOpacity(1);
      // If the left mouse button is pressed
      if (Sup.Input.wasMouseButtonJustPressed(0)) {
        // Loop through different case
        switch (this.screen){
          // If the current screen is the Main screen
          case Menu.screens.main:
            // Set the screen of the Spacewar game screen
            this.setScreen(Menu.screens.spacewar);
            break;
            // Else, in other case, return to main screen
          default:
            this.setScreen(Menu.screens.main);
        }
      }
    }
    // If the button is not hovered, set opacity half bright
    else{
      this.button1.getChild("Sprite").spriteRenderer.setOpacity(0.5);
    }
    // Update the buttons
    this.updateButtonsText();
  }
[...]
```

##### Load Game Scene

In the global script, we need to load a new Scene Game when we start a new game. (In the beginning of the function Game.start)
We remove the temporary Game.start() call if we still have it somewhere in the Game script to avoid a bug when the behavior is loaded.

```ts
[...]
namespace Game{
[...]
  // Start the game
  export function start(){
    // Load Game Scene
    Sup.loadScene("Game/Scene");
[...]
```

Our menu is now working and we can choose in the menu the game we want to play.

#### Game over screen and final score

##### Global script : The Game.gameOver function 
We will now set the game over screen when the game finish (there is two, way, one ship lose all it's life or the timer is out).

We add a function in the Game module of the Global script.

```ts
[...]
namespace Game{
  [...]
  // Load the gameOver screen and display score
  export function gameOver(winner: string){
    // Initialize variables
    let score1: number;
    let score2: number;
    // Store the ship1 score before to leave the scene
    score1 = Sup.getActor("Ship1").getBehavior(ShipBehavior).score;
    // If the game is spacewar, stock also the Ship2 score
    if (nameIndex === 1) {
      score2 = Sup.getActor("Ship2").getBehavior(ShipBehavior).score;
    }
    // Close game scene and load menu scene
    Sup.loadScene("Menu/Scene");
    // Set game over Screen
    Sup.getActor("Menu").getBehavior(MenuBehavior).setScreen(Menu.screens.gameover);
    // Get in a variable gameOverScreen the gameover screen actor
    let gameOverScreen = Sup.getActor("Screens").getChild(Menu.screens.gameover.toString());
    // Set the Sprite from the actor to display the winner frame
    gameOverScreen.spriteRenderer.setAnimation(winner);
    // Display Score
    // Set title visible false
    Sup.getActor("Title").setVisible(false);
    // Get the Score actor in a variable
    let score: Sup.Actor = Sup.getActor("Score");
    // Set the Score actor visible
    score.setVisible(true);
    // Set the score text to the current score of ship1
    score.getChild("Ship1").textRenderer.setText("Ship1:"+score1);
    // If the game is spacewar
    if (nameIndex === 1) {
      // Set visible true the score of Ship2
      score.getChild("Ship2").setVisible(true);
      // Set the score text to the current score of ship2
      score.getChild("Ship2").textRenderer.setText("Ship2:"+score2);
      
    }
    else {
      // Set visible false the ship2 score
      score.getChild("Ship2").setVisible(false);
    }
  }
}
[...]
```

#### Game options : Quit and Restart

In the game script, we add two little functionalities, the possibility to leave or restart the game at any moment.


```ts
[...]
  update() {
  [...]
    // Restart game when key (R) is pressed, call Game.start function which reload the game scene
    if (Sup.Input.wasKeyJustPressed("R")) {
      Game.start();
    }
    
    // Leave Game when key (ESCAPE) is pressed, load the menu scene
    if (Sup.Input.wasKeyJustPressed("ESCAPE")) {
      Sup.loadScene("Menu/Scene");
    }
  }
[...]
```

Our game is mostly finished, in the next chapter we will polish it to add the background scrolling and the sound effects.

[<-- back to chapter 8](ch8.md) -- [go to chapter 10 -->](ch10.md)