# SNAKE-GAME-using-c-language-
Overview
This is a complete implementation of the classic Snake game written in C. The game runs in the Windows console and provides an interactive gaming experience where players control a snake to eat food and grow longer while avoiding collisions with walls and the snake's own body.

Table of Contents
Project Structure
Game Mechanics
Code Architecture
Detailed Code Explanation
Game Logic Flow
Controls
Compilation and Execution
Technical Details
Project Structure
snake/
├── snake.c          # Main source code file
├── snake.exe        # Compiled executable
└── README.md        # This documentation file
Game Mechanics
Core Gameplay
Objective: Control a snake to eat food (*) and grow longer
Movement: Snake moves continuously in the current direction
Growth: Snake grows by one segment each time it eats food
Game Over Conditions:
Snake hits the boundary walls
Snake collides with its own body
Player presses 'X' to quit
Scoring System
Points: 10 points for each food item consumed
Display: Current score is shown at the bottom of the game screen
Code Architecture
Header Files Used
#include <conio.h>    // For keyboard input functions (kbhit, getch)
#include <ctype.h>    // For character conversion (tolower)
#include <stdio.h>    // For input/output functions
#include <stdlib.h>   // For system and rand functions
#include <windows.h>  // For Sleep function
Global Variables
Variable	Type	Purpose
HEIGHT, WIDTH	#define	Game board dimensions (20x40)
snakeTailX[], snakeTailY[]	int[100]	Arrays storing snake body coordinates
snakeTailLen	int	Current length of snake's body
gameover	int	Game state flag (0=playing, 1=game over)
key	int	Current movement direction (1=left, 2=right, 3=up, 4=down)
score	int	Current game score
x, y	int	Snake head coordinates
fruitx, fruity	int	Food coordinates
Detailed Code Explanation
1. Setup Function (setup())
void setup() {
    gameover = 0;                    // Initialize game state
    x = WIDTH / 2;                   // Center snake horizontally
    y = HEIGHT / 2;                  // Center snake vertically
    fruitx = rand() % WIDTH;         // Random food X position
    fruity = rand() % HEIGHT;        // Random food Y position
    score = 0;                       // Initialize score
}
Purpose: Initializes all game variables to their starting values.

Key Features:

Centers the snake at the middle of the game board
Generates random food position (avoiding edges)
Resets score and game state
2. Draw Function (draw())
void draw() {
    system("cls");                   // Clear console screen
    
    // Draw top border
    for (int i = 0; i < WIDTH + 2; i++)
        printf("-");
    printf("\n");
    
    // Draw game area
    for (int i = 0; i < HEIGHT; i++) {
        for (int j = 0; j <= WIDTH; j++) {
            if (j == 0 || j == WIDTH)        // Side walls
                printf("#");
            else if (i == y && j == x)       // Snake head
                printf("O");
            else if (i == fruity && j == fruitx)  // Food
                printf("*");
            else {                            // Snake body or empty space
                // Check if current position is snake body
                int prTail = 0;
                for (int k = 0; k < snakeTailLen; k++) {
                    if (snakeTailX[k] == j && snakeTailY[k] == i) {
                        printf("o");
                        prTail = 1;
                        break;
                    }
                }
                if (!prTail)
                    printf(" ");
            }
        }
        printf("\n");
    }
    
    // Draw bottom border and UI
    for (int i = 0; i < WIDTH + 2; i++)
        printf("-");
    printf("\n");
    printf("score = %d", score);
    printf("\n");
    printf("Press W, A, S, D for movement.\n");
    printf("Press X to quit the game.");
}
Purpose: Renders the complete game state to the console.

Visual Elements:

- : Top and bottom borders
# : Left and right walls
O : Snake head
o : Snake body segments
* : Food
  : Empty space
3. Input Function (input())
void input() {
    if (kbhit()) {                  // Check if key is pressed
        switch (tolower(getch())) {  // Get key and convert to lowercase
        case 'a':                    // Left
            if(key != 2) key = 1;    // Prevent 180° turn
            break;
        case 'd':                    // Right
            if(key != 1) key = 2;    // Prevent 180° turn
            break;
        case 'w':                    // Up
            if(key != 4) key = 3;    // Prevent 180° turn
            break;
        case 's':                    // Down
            if(key != 3) key = 4;    // Prevent 180° turn
            break;
        case 'x':                    // Quit
            gameover = 1;
            break;
        }
    }
}
Purpose: Handles real-time keyboard input and updates movement direction.

Key Features:

Uses kbhit() for non-blocking input
Prevents 180-degree turns (snake can't reverse directly)
Converts input to lowercase for case-insensitive control
4. Logic Function (logic())
void logic() {
    // Update snake body positions
    int prevX = snakeTailX[0];
    int prevY = snakeTailY[0];
    int prev2X, prev2Y;
    snakeTailX[0] = x;              // Move first body segment to head position
    snakeTailY[0] = y;
    
    // Cascade body segments
    for (int i = 1; i < snakeTailLen; i++) {
        prev2X = snakeTailX[i];
        prev2Y = snakeTailY[i];
        snakeTailX[i] = prevX;
        snakeTailY[i] = prevY;
        prevX = prev2X;
        prevY = prev2Y;
    }
    
    // Move snake head based on direction
    switch (key) {
    case 1: x--; break;  // Left
    case 2: x++; break;  // Right
    case 3: y--; break;  // Up
    case 4: y++; break;  // Down
    }
    
    // Check wall collision
    if (x < 0 || x >= WIDTH || y < 0 || y >= HEIGHT)
        gameover = 1;
        
    // Check self collision
    for (int i = 0; i < snakeTailLen; i++) {
        if (snakeTailX[i] == x && snakeTailY[i] == y)
            gameover = 1;
    }
    
    // Check food collision
    if (x == fruitx && y == fruity) {
        score += 10;                 // Increase score
        snakeTailLen++;              // Grow snake
        // Generate new food position
        fruitx = rand() % WIDTH;
        fruity = rand() % HEIGHT;
        while (fruitx == 0) fruitx = rand() % WIDTH;
        while (fruity == 0) fruity = rand() % HEIGHT;
    }
}
Purpose: Implements the core game logic including movement, collision detection, and food consumption.

Key Algorithms:

Snake Movement: Each body segment follows the path of the segment in front of it
Collision Detection: Checks for wall and self-collision
Food Consumption: Detects when snake eats food and handles growth
5. Main Function (main())
void main() {
    setup();                        // Initialize game
    
    while (!gameover) {             // Main game loop
        draw();                     // Render game state
        input();                    // Handle user input
        logic();                    // Update game logic
        Sleep(33);                  // ~30 FPS (1000ms/30 ≈ 33ms)
    }
}
Purpose: Orchestrates the game loop and manages the overall flow.

Game Loop:

Draw: Render current game state
Input: Process user input
Logic: Update game state
Sleep: Control game speed (~30 FPS)
Game Logic Flow
Initialization Phase
Set game dimensions (20x40)
Initialize snake at center position
Generate random food position
Reset score and game state
Main Game Loop
Rendering: Clear screen and draw all game elements
Input Processing: Check for keyboard input and update direction
Movement: Update snake head position based on current direction
Body Update: Move all body segments to follow the head
Collision Detection: Check for wall and self-collision
Food Check: Detect food consumption and handle growth
Frame Rate Control: Wait 33ms before next iteration
Game Over Conditions
Snake head hits boundary walls
Snake head collides with any body segment
Player presses 'X' to quit
Controls
Key	Action
W	Move Up
A	Move Left
S	Move Down
D	Move Right
X	Quit Game
Note: The game prevents 180-degree turns to avoid instant self-collision.

Compilation and Execution
Compilation
gcc snake.c -o snake.exe
Execution
./snake.exe
Requirements
Windows operating system (uses Windows-specific headers)
C compiler (GCC recommended)
Console/Command Prompt
Technical Details
Performance Characteristics
Frame Rate: ~30 FPS (33ms delay between frames)
Memory Usage: Fixed arrays for snake body (100 segments max)
Input Handling: Non-blocking keyboard input using kbhit()
Data Structures
Arrays: snakeTailX[100], snakeTailY[100] for snake body coordinates
Variables: Simple integer variables for game state management
Algorithm Complexity
Rendering: O(HEIGHT × WIDTH) for each frame
Collision Detection: O(snakeTailLen) for self-collision check
Movement: O(snakeTailLen) for body segment updates
Limitations
Maximum snake length: 100 segments
Windows-specific implementation (uses conio.h and windows.h)
Console-based graphics (ASCII characters only)
Fixed game board size (20x40)
Future Enhancements
Potential improvements for this snake game implementation:

Cross-platform Support: Replace Windows-specific functions with portable alternatives
Configurable Board Size: Allow dynamic game board dimensions
High Score System: Implement persistent high score tracking
Multiple Difficulty Levels: Add speed variations and obstacles
Enhanced Graphics: Implement better visual representation
Sound Effects: Add audio feedback for game events
Menu System: Add game menu and pause functionality
Power-ups: Implement special food types with different effects
Conclusion
This snake game implementation demonstrates fundamental programming concepts including:

Game loop architecture
Real-time input handling
Collision detection algorithms
Array manipulation for game state
Console-based graphics rendering
The code is well-structured with clear separation of concerns between rendering, input handling, and game logic, making it an excellent example for learning game development fundamentals in C.
