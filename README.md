This code presents an attempt to reproduce the legendary pixel game Pacman using the Python programming language. To do this, libraries such as pygame, numpy, tcod, random and enum are used. The code outputs one of the game cards with the ability to move the character (Pacman himself) and collect cookies. Casts that randomly move around the map are also implemented. 
What needs to be improved in the future:
- add animation of opening/closing the Pacman's mouth;
- detail the appearance of ghosts;
- add pacman interaction with ghosts;
- add a counter of lives and collected cookies;
- add music.

## Basic settings
First, we will create a game window where we will prepare parameters for specifying the window size, the name of the game, the refresh rate and several data fields that will contain links to game objects and the player. The tick function iteratively traverses all game objects and invokes their internal logic and rendering. Then all that remains is to redraw the entire game area and process input events, such as mouse clicks and keyboard input. The _handle_events function will serve for this purpose.

## Parent game object
Then I create a parent game object named GameObject from which other classes will inherit functionality. In the game we will have objects for the wall (Wall), Pac-Man (Hero), ghosts (Ghost) and cookies (Cookie). For the movable game objects mentioned above, I will later create a MovableObject class, which will be an extension of the GameObject class with moving functions.

During the initialization of an object, I set its color, shape and position. Each object also has a reference to the _surface rendering surface, so it can take care of its own rendering on the main surface. For this purpose, we have a draw function that is called by a previously created GameRenderer for each game object. Depending on the is_circle parameter, the object is displayed either as a circle or as a rectangle (in our case, I use a square with slightly rounded corners for walls and a circle for Pac-Man and cookies).

Creating a wall class will be easy. For the walls, I choose the blue color according to the original Pac-Man (the color parameter is Blue 255, the rest is 0).

The code for rendering and the object for the walls have been prepared. When writing, make sure that the Wall and GameObject classes are higher than the GameRenderer class so that the class "sees" them. The next step is to visualize the maze on the screen. But before that we have to create one auxiliary class.

## The game controller class
I will save the maze in ASCII characters in a variable in the new Pacman Game Controller class. I will use the original size of the maze — 28x31 tiles. Later I will need to make sure that the ghosts will be able to get through the maze correctly and possibly find the player. First I will read the maze as symbols and transform it into a matrix of ones and zeros, where the wall is zero and the traversable space is one. These values serve pathfinding algorithms as a so-called cost function. Zero means an infinite cost of passing, so the elements in the array marked in this way will not be considered passable. Pay attention to the reachable_spaces array, which contains the traversable parts of the maze.

## Rendering the maze
Everything necessary for rendering the maze has been prepared, so all that remains is to create instances of our PacmanGameController classes, walk through a 2D array with wall positions and create a Wall object in these places (I use the add_wall function). I set the refresh rate to 120 frames per second.

## Let's add ghost!
The original Pac-Man had four ghosts named Blinky, Pinkie, Inky and Clyde, each with their own character and abilities. The concept of the game is based on a Japanese fairy tale, and the original names in Japanese also suggest their abilities (for example, Pinkie has a Japanese name Thief, Blinky has a Shadow). However, for our game we will not go into such details, and each ghost will use only the basic behavioral cycle, as in the original, that is, the "pursuit", "scattering" and "fright" modes. We will write them later.

The ghost class will be simple, inheriting most of its behavior from the parent MovableObject class.

I will add RGB values for the colors of each ghost to the Pacman Game Controller class and generate four colored ghosts in the main function. I will also prepare a static coordinate transformation function that simply transforms the coordinates of the maze (for example, x=16 y=16 approximately corresponds to the center of the maze, and multiplying by the size of the cell or tile gives me the coordinate to the surface of the game in pixels).

At this stage, four ghosts will be displayed in the maze when the game starts. Now we want to make them move.

## Maze pathfinding
Now comes perhaps the most difficult part. Finding a path in a two—dimensional space or graph is a difficult task. The implementation of the algorithm for solving such a problem would take a separate article, so we will use a ready-made solution. The most efficient pathfinding algorithm is the A* algorithm. This is provided by the tcod package that we installed at the beginning.

To move the ghosts, I will create a class named Pathfinder. In the constructor, I will initialize the numpy array with the cost of passing (the array of ones and zeros described earlier) and create a variable of the pf class that will contain an instance of the navigator A*. Then the get_path function will calculate and return the path as a series of steps in the array when called with coordinates in the maze (from, to).

Now I will add a section to the main function to demonstrate pathfinding. I choose the starting coordinates [1,1] and the destination of the route [24,24]. This is an optional code.

## Randomized ghost movement
Now we are going to create a new function in the Pacman Game Controller class to select a random point from the reachable_spaces array. Each ghost will use this feature after it reaches its destination. Thus, the ghosts will simply choose a path from their current position in the maze to a random destination indefinitely. In the next part, we will implement more complex behavior, such as chasing and fleeing from the player.

In the Ghost class, we add new logic to follow the path. The reached_target function calls each frame and checks if the ghost has already reached its goal. If yes, then it determines in which direction the next step of the path through the maze is located, and begins to change its position up, down, left or right (the logic of movement is called in the parent MovableObject class). The ghosts are now created in the positions indicated by the letter G in the original ASCII maze and start looking for a random path. I have locked three ghosts in a cage — as in the original Pacman, they will be released one at a time — and one wanders through the maze.

## Player controls
To add player functionality, I will create a class called Hero. Most of the control logic of both the player and the ghosts is handled in the MovableObject class, so we only need a few functions to determine the behavior of the player. In the original Pacman, the player can move in four directions controlled by the arrow keys. If no arrow keys are pressed, the player will continue moving in the last allowed direction. If the key is pressed in a direction in which the player cannot move, this direction is saved and used in the next available move. I will reproduce this behavior in our game, and also add Pacman's ability to teleport from one end of the maze to the other — I'll just check if the player is outside the playing area on the left or right side, and set his position to the opposite side of the maze, respectively. Pacman also has a modified rendering function, we need to display it half as much as it normally takes (using pygame.rect).

I create an instance of the Hero class at the end of the main function. I set the position with coordinates [1,1] — unified_size — the size of one tile. We also need to add input event handling to the GameRenderer class so that we can control the game character.

After launching the game, we can now control the Pacman player!

## Adding coockies
It wouldn't be a Pacman without cookies in the maze. In terms of gameplay, they determine the degree of exploration of the world, and some cookies even change the abilities of ghosts and Pacman. Thus, they are the highest reward for players and the main indicator of their progress in levels. In modern games, the behavior that a game designer wants to encourage in a player is usually rewarded. An excellent example is the Elden Ring, where everyone who explores every corner of the world gets rewarded. The more dangerous and distant, the greater the reward. On the other hand, games like Assassin's Creed support the execution of tasks, so during the game you get the feeling that you are working, not playing.

Adding cookies will be the easiest in the whole lesson, so I left it at the end like the cherry on the cake. I will create a class called Cookie. Its instance will always have a size of four pixels, a yellow color and a round shape. In the main function, I will create cookies on all tiles that we initially saved in the cookie_spaces array (the same as reachable_spaces). I will add a function named handle_cookie_pickup to the player, in which I constantly check if the player encounters any cookie. If so, I will remove the cookie from the array and it will no longer be displayed.

A little interesting fact finally — in the original game, Pacman stops for one frame after eating each cookie, so it's easier for ghosts to catch him at the beginning of the game when the field is still full. In the next part, we will implement similar game mechanics, and you can also count on artificial intelligence for ghosts, score tracking, sounds, animations, textures, power-ups, screen shake effects, lives, and final game states.