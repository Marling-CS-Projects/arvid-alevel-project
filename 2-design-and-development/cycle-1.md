# 2.2.1 Cycle 0

## Design

### Objectives

I plan to create a webpage using html and javascript to host the game. I have tested several libraries for purpose of creation of the game, and Phaser seems like it will be the most appropriate for my game because it has support for matter.js, a library focused on physics. On the webpage, use phaser to make a character and have it fall from where it's created onto a platform.

* [x] Make a html/javascript project
* [x] Create a webpage using node and express
* [x] Have it redirect automatically redirect to a page with Phaser
* [x] Create an object to represent the player and have it fall onto a platform

### Usability Features

* Automatic redirect - Ease of use/efficiency. The player won't have to try and connect to the game manually, saving time and reducing the chance a player would be confused.
* Clear visuals - The object representing the player will be a vastly different colour from the background, and should be easily recognisable from the ground

### Key Variables

| Variable Name | Use                                                             |
| ------------- | --------------------------------------------------------------- |
| preload       | a function that initialises the sprites/images for game objects |
| create        | a function that initialises variables                           |
| ground        | stores position, size and physics of the ground                 |
| player        | stores position, size and physics of the player                 |
| background    | it's just the background image                                  |

### Pseudocode

```
procedure preload
    ground = ground.png
    background = background.png
    player = player.png
end procedure

procedure create
    player = create rectangle (position, physics and scaling)
    ground = create rectangle (position, scaling and physics)
end procedure
```

## Development

### Outcome

I have gotten my server.js to automatically redirect to game.html. Within game.html, I have an object called config which sets the size of the game window and sets up basic physics such as gravity. Under preload, I have just defined&#x20;

### Challenges

Description of challenges

## Testing

Evidence for testing

### Tests

| Test | Instructions | What I expect | What actually happens | Pass/Fail |
| ---- | ------------ | ------------- | --------------------- | --------- |
| 1    |              |               |                       |           |

### Evidence
