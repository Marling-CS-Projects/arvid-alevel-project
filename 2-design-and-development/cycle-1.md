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
    player = create rectangle (position, physics and image scaling)
    ground = create rectangle (position, image scaling and physics)
end procedure
```

## Development

### Outcome

I have gotten my server.js to automatically redirect to game.html. Within game.html, I have an object called config which sets the size of the game window and sets up basic physics such as gravity. In preload I just assigned each image a name and loaded them so they could be used, and in create I made the player and ground platform in their respective locations with a collider to make sure they would interact with each other.

{% tabs %}
{% tab title="server.js" %}
```javascript
var express = require('express');
var app = express();

app.use(express.static('tree'));
app.use(express.static('assets'));

app.get('/', function (req, res) {
    res.redirect('/game.html');
})

var server = app.listen(8081, function () {
    var host = server.address().address
    var port = server.address().port

    console.log("Example app listening at http://%s:%s", host, port)
})
```
{% endtab %}

{% tab title="game.html" %}
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>mogus</title>
    <script src="//cdn.jsdelivr.net/npm/phaser@3.11.0/dist/phaser.js"></script>
    <style>
        body{
            margin: 0;
        }
    </style>
</head>
<body>

    <script type="text/javascript">

        var config = {
            type: Phaser.AUTO,
            width: 1920,
            height: 1080,

            scale: {
                min: {
                    width: 0,
                    height: 0,
                },
                max: {
                    width: 1920,
                    height: 1080,
                },
            },

            physics: {
                default: 'arcade',
                arcade: {
                    gravity: { y: 300 }, //gravity
                    debug: false,
                }
            },
            scene: {
                preload: preload,
                create: create,
            },
        };

        var player;
        var platforms;

        var game = new Phaser.Game(config);

        function preload() {
            this.load.image('background', '/background.png');
            this.load.image('ground', '/ground.png');
            this.load.image('player', '/player.png');
        }

        function create() {
            //Background
            this.add.image(400, 300, 'background');

            //The things we stand on
            ground = this.physics.add.staticGroup();

            //Create the ground, and get it to scale to be bigger
            ground.create(400, 584, 'ground').setScale(2).refreshBody();

            //Player settings
            player = this.physics.add.image(400, 470, 'player');

            //This makes sure the player doesn't run off screen. This is only temporary for testing purposes.
            player.setCollideWorldBounds(true);

            //Player will obey the laws of physics, and not phase through solid ground
            this.physics.add.collider(player, ground);
        }
    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

Learning how express used files was a bit challenging when trying to load images. Originally in lines 52 to 54 of game.html, I misunderstood how the node server would read files, and hence the program was looking in the assets folder for the images, which, as far as it was concerned, didn't exist.

## Testing

### Tests

| Test | Instructions                              | What I expect                                                                                 | What actually happens | Pass/Fail |
| ---- | ----------------------------------------- | --------------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code                              | A node console should appear with information about the server                                | As Expected           | Pass      |
| 2    | Connect to the server with localhost:8081 | A black screen with a platform and player, which should fall onto the platform and stay there | As Expected           | Pass      |

### Evidence

![](<../.gitbook/assets/image (3).png>)
