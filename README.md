OrginalDodge
============

This is the working code!



// Import the stuff required so LinkedList works
import java.util.*;

// Store our circles into a list of circles.
LinkedList<Circle> circles = new LinkedList<Circle>();

// Store all our variables here so we don't have to change them in the code.
int moveMin = 2;            // Minimum speed
int moveMax = 5;            // Maximum speed

// Window size. The bigger, the more fun it is. See for yourself :)
int sizeX = 640;
int sizeY = 480;

// Values that need to be kept and updated on a frame-by-frame basis go here
boolean hit = false;        // Required to be global because of how draw is frame-based
int circleNumber = 1;       // The more circles, the more points the player gets per frame
int stage = 1;              // Used to move between title, game, hit, game over screens
double score = 0;           // Score keeping as a double, *but* displayed as an integer
int immunity = 100;         // Immunity from being hit. Drains when used, regenerates when not
int lives = 3;              // Player is given 3 lives
int grace = 60;             // Give time to move the cursor if the ball happens to spawn on it
int addCircleFrames = 400;  // A circle is added after the game has lasted this number of frames
int framesPassed = 0;       // This will be reset to 0 after every spawn

void spawnCircle()
{
    // Spawn a circle that moves in a random direction faster than the minimum speed for both axes
    // The code is quite complicated, but you'll see what I'm doing if you go through it.

    int moveX = 0;          // Starting movement speed for X
    int moveY = 0;          // Starting movement speed for Y
    int testValue = 0;

    while (Math.abs(moveX) < moveMin || Math.abs(moveY) < moveMin)  // Any speed < minimum
    {
        testValue = (int)random(moveMax * -1, moveMax);             // Generate random
        if (Math.abs(testValue) >= moveMin)                         // If random > minimum
        {
            if (Math.abs(moveX) < moveMin) moveX = testValue;       // Change X if necessary
            else if (Math.abs(moveY) < moveMin) moveY = testValue;  // Change Y if necessary
        }
    }

    // We're finally happy with our circle. Draw it
    circles.add(new Circle((int)random(0,sizeX), (int)random(0,sizeY), moveX, moveY));
}

void setup()
{
    size(sizeX,sizeY);
    spawnCircle();
}

// NTSenpai: Draw method updates *every frame*
void draw()
{
    background(128);
    switch (stage)                  // Stage changes depending on what happens:
    {
        case 1: begin(); break;     // When we start or replay the game
        case 2: game(); break;      // When we click the mouse after the title screen
        case 3: hit(); break;       // When the cursor hits a circle
        case 4: gameOver(); break;  // When there is no more lives
    }
}

private void begin()
{
  textAlign(CENTER);
  textSize(48);
  text("Dodging Circles", sizeX/2, sizeY/2 - 120);
  textSize(24);
  text("Click to play", sizeX/2, sizeY/2);
  textSize(18);
  text("Dodge circles and hold the mouse button to become immune for a short while", sizeX/2-200, sizeY/2+100, 400, 200);
  if (mousePressed)
    stage = 2; // Game
}

private void game()
{  
    addCircle();
    updateScore();
    updateImmunity();
    checkTouched(); 
    updateProgress();
    for (Circle circle: circles) circle.display();
}

private void hit()
{
    if (hit == false) lives --;             // So that the next frame doesn't do this again
    hit = true;
    if (lives <= 0) stage = 4;              // Ran out of lives? Go to Game Over mode
    if (stage == 3)                         // If we're still in hit mode
    {
        for (Circle circle: circles) circle.paused();
        showLives();
        if (keyPressed) resumePlay();
    }
}

private void resumePlay()
{
    grace = 120;    // 120 frame grace to get used to the directions
    hit = false;   // Disable flag
    stage = 2;      // Back to game mode
}

private void showLives()
{
    textSize(64);
    text((int)score, sizeX/2, sizeY/2);
    textSize(24);
    fill(0);
    text("Lives: " + lives, sizeX/2, sizeY/2 + 120);
    text("Press any key to resume", sizeX/2, sizeY/2 + 150);
    fill(255);
}

private void gameOver()
{
    textAlign(CENTER);
    textSize(64);
    text((int)score, sizeX/2, sizeY/2);
    textSize(32);
    text("Press any key to retry", sizeX/2, sizeY/2 + 80);
    if (keyPressed) backToTitle();
}

private void addCircle()
{
    // If it's time to add a circle...
    if (framesPassed > addCircleFrames)
    {
        circleNumber ++;      // Increment circle count
        framesPassed = 0;     // Reset the frame count
        grace = 60;           // Add a 60 frame grace
        spawnCircle();        // Finally, spawn our circle
    }
    framesPassed ++;        // Else, increment the frame count
    grace --;               // Decrease the grace if it exists
}

private void updateScore()
{
    score += (double)circleNumber/50;   // Score is an integer but we want decimal accuracy
    fill(255);                          // Text colour is now white
    textSize(64);                       // Text size is now 64
    textMode(CENTER);                   // Centre text
    text((int)score, sizeX/2, sizeY/2); // Show the score in the middle of the screen
}

private void updateImmunity()
{
    // Mouse click held? Drain immunity
    if (mousePressed && immunity > 0)
        immunity --;
    // Mouse click released? Regenerate the lost immunity
    if (!mousePressed && immunity < 100)
        immunity ++;
    // Display the immunity bar
    fill(64);
    rect(sizeX/2-100,sizeY/2+50,immunity*2,40);
    fill(255);
}

private void updateProgress()
{
    fill(192);
    rect(sizeX/2-200,sizeY/2-120,(addCircleFrames-framesPassed),20);
}

private void checkTouched()
{
    // If the circle has touched, move onto hit() method
    for (Circle circle: circles)
        if (circle.touched() && !(mousePressed && immunity > 0) && grace < 0) stage = 3;
}

private void backToTitle()
{
    circles.clear();
    spawnCircle();
    score = 0;
    lives = 3;
    stage = 1;
    circleNumber = 1;
}

// This is where we define our Circle class
class Circle
{
    int x = 0;                // X position
    int y = 0;                // Y position
    int dX = 0;               // Movement distance of X
    int dY = 0;               // Movement distance of Y
    int radius = 50;          // Radius of circle
    
    // Constructor method with required parameters
    public Circle(int x, int y, int dX, int dY)
    {
        // Set the parameters to this (the Circle's) values
        this.x = x;
        this.y = y;
        this.dX = dX;
        this.dY = dY;
    }
    
    // Random movement will flip between negative and positive when it hits the edge of the screen
    // randomMove() definition defined below...
    private void moveCircle()
    {
        if (x > sizeX) dX = randomMove() * -1;
        if (x < 0) dX = randomMove();
        if (y > sizeY) dY = randomMove() * -1;
        if (y < 0) dY = randomMove();
        x += dX;
        y += dY;
    }
    
    // Display the circle for this frame, and move it
    private void display()
    {
        if (immunity > 0 && mousePressed || grace > 0) fill(128); // Shaded circle for immunity
        else fill(255);                         // Otherwise white
        ellipse(x, y, radius, radius);          // Draw the circle
        moveCircle();                           // Move it
    }
    
    // Used to pause the movement of circles when hit. Essentially no move method after.
    private void paused()
    {   ellipse(x, y, radius, radius);  }
    
    // Did the cursor move within the circle's area?
    public boolean touched()
    {   return (dist(x, y, mouseX, mouseY) < radius/2); }
    
    // Return a random number between the values mentioned at the beginning of this file.
    private int randomMove()
    {   return (int)random(moveMin, moveMax);   }
}
