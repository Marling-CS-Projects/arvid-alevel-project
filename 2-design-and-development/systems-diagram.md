# 2.1 Design Frame

![](<../.gitbook/assets/image (5) (1) (2).png>)

In the diagram above, each section of my game is split into the sections I will work on. Each section has sub-sections, based off of the features presented in [my success criteria](../1-analysis/1.5-success-criteria.md).&#x20;

## Usability Features

Usability Features are features aimed at increasing the accessibility of my game to as many users as possible, and improve the quality of play. Some of a key aspects are as follows:

### Effective

For my game to be considered effective, players should be able to clearly see where they are able to go, where all hazards are, and easily get a clear idea of how movement and combat is done. Clarity in this way ensures a better experience for all players, as more time is spent playing than being confused.

#### Aims

* The game clearly differentiates between hazards, terrain, and the player
* The controls are easy to learn and use
* In game instructions are clear and easy to use

### Efficiency

Efficiency can be defined as the speed, ease, and accuracy with which a player can complete a task. The fewer things must be done on the player's behalf to get an action done, the more efficient the system is. For this reason, menus will be clear and controls will be the least keys possible whilst still allowing for simplicity.&#x20;

#### Aims

* There are enough controls to feel varied, but not to the point of being overbearing (a good amount)
* In game instructions are clear and easy to follow

### Engaging

The game is engaging for the players. In other words, the game is fun to play. I will have multiple ways for the player to achieve in game things such as combat and movement, which means there will be gameplay variety and choice for the player which should decrease any repetitive feeling the game has, and hence increase enjoyment.

#### Aims

* The game is decently challenging whilst remaining fun
* The movement system is intuitive and feels dynamic when used

### Error Tolerant

The game should be free of any errors that may affect a user's experience, and be resilient to strange scenarios that may not be easy to predict. It should be able to correct any bugs it has and be constructed in such a way that it does not break when an unintended action is taken.

#### Aims

* The game doesn't crash
* The game does not contain any bugs that negatively affect the user's experience
* The game tolerant to strange interaction that may not have been intended and should allow them to function without crashing/causing any bugs.&#x20;

### Easy To Learn

The game should have very basic controls and interactions to learn, but allow for complexity in interaction and depth for the player once they get to grips with the basics. Both the simple and in-depth interactions should be easy to understand and not require complicated inputs.

#### Aims

* The controls must be easy to use and easy to learn
* The movement system must be understood, meaning the interaction between movement options can be figured out and used with relative ease.

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
    physics2: set up more physics
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
        check if player is at screen edge
    end procedure
    
end class
```
