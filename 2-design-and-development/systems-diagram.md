# 2.1 Design Frame

![](<../.gitbook/assets/image (5).png>)

In the diagram above, each section of my game is split into the sections I will work on. Each section has sub-sections, based off of the features presented in [my success criteria](../1-analysis/1.5-success-criteria.md).&#x20;

## Usability Features

Usability Features are features aimed at increasing the accessibility of my game to as many users as possible, and improve the quality of play. Some of a key aspects are as follows:

### Effective

For my game to be considered effective, players should be able to clearly see where they are able to go, where all hazards are, and easily get a clear idea of how movement and combat is done. Clarity in this way ensures a better experience for all players, as more time is spent playing than being confused.

#### Aims

* Create a clear goal to reach to determine the end of a level
* Create a clear goal for any multiplayer modes

### Efficiency

The speed and accuracy to which a user can complete the goal. To do this, I will create a menu system which is easy to navigate through in order for to find what you are looking for. The information which is more important can be found with less clicks.

#### Aims

* Create a menu system that is quick and easy to navigate through
* Create a controls system that isn't too complicated but allows the player to do multiple actions

### Engaging

The solution is engaging for the user to use. To do this, I will create 5 levels and an online multiplayer mode to keep the players engaged and allow them to have fun while playing the game. Using vector style art will also make the game nicer to look at than blocks, so will draw more people in, keeping them engaged.

#### Aims

* Create a series of levels to work through
* Create a multiplayer mode to play
* Incorporate a style of game art the suits the game

### Error Tolerant

The solution should have as few errors as possible and if one does occur, it should be able to correct itself. To do this, I will write my code to manage as many different game scenarios as possible so that it will not crash when someone is playing it.

#### Aims

* The game doesn't crash
* The game does not contain any bugs that damage the user experience

### Easy To Learn

The solution should be easy to use and not be over complicated. To do this, I will create simple controls for the game. I will make sure that no more controls are added than are needed in order to keep them as simple as possible for the players.

#### Aims

* Create a list of controls for the game
* Create an in-level guide that helps players learn how to play the game

## Pseudocode for the Game

### Pseudocode for game

This is the basic layout of the object to store the details of the game. This will be what is rendered as it will inherit all important code for the scenes.

```
object Game
    type: Phaser
    parent: id of HTML element
    width: width
    height: height
    physics: set up for physics
    scenes: add all menus, levels and other scenes
end object

render Game to HTML web page
```

### Pseudocode for a level

This shows the basic layout of code for a Phaser scene. It shows where each task will be executed.

```
class Level extends Phaser Scene

    procedure preload
        load all sprites and music
    end procedure
    
    procedure create
        start music
        draw background
        create players
        create platforms
        create puzzle elements
        create enemies
        create obstacles
        create finishing position
        create key bindings
    end procedure
    
    procedure update
        handle key presses
        move player
        move interactable objects
        update animations
        check if player at exit
    end procedure
    
end class
```
