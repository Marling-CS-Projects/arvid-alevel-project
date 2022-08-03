# 2.2.2 Cycle 1

## Design

### Objectives

In this development cycle I want to add horizontal movement with acceleration, deceleration, which function with turning. This will be controlled by the arrow keys.

* [x] Assign keys for the movement to be controlled by
* [x] Get the player to accelerate in the direction pressed
* [x] The speed is capped at a limit which the player shouldn't be able to pass
* [x] The player moves in the direction of the pressed key

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name | Use                                                                     |
| ------------- | ----------------------------------------------------------------------- |
| velocity      | the current speed the player moves at                                   |
| accelerator   | the amount the speed is increased by per frame                          |
| maxVelocity   | the speed the player can accelerate to                                  |
| baseVelocity  | the speed at which the player starts before acceleration                |
| inputKeys     | stores the value of keys on the keyboard so they can be used for inputs |
| isGrounded    | detects if the player is touching the ground                            |
| canRun        | determines if the player is allowed to move                             |
| update        | a function that runs every frame                                        |

### Pseudocode

```
object config
    height: 600
    width: 800
    physics: gravity
    scene: use create and update procedures
    scale: center game window, scale to fit
end object

procedure preload
    ground = ground.png
    background = background.png
    player = player.png
end procedure

procedure create
    player = create rectangle (position, physics and image scaling)
    ground = create rectangle (position, image scaling and physics)
    
    set collisions between player and the ground
    set collisions between player and the edge of the window
    
    if player is touching ground
        isGrounded = true
    end if
    
    inputKeys = keybind setup

end procedure

procedure update
    if isGrounded = true
        canRun = true
        if inputKeys.left is down and canRun = true
            if velocity < maxVelocity
                velocity = velocity + accelerator
            elif velocity > maxVeloctiy
                velocity = maxVelocity
            move player -veolcity
        elif inputKeys.right is down and canRun = true
            if velocity < maxVelocity
                velocity = velocity + accelerator
            elif velocity > maxVeloctiy
                velocity = maxVelocity
            move player velocity
        else
            move player 0
            veloctiy = baseVelocity
        end if
    end if
end procedure
```

## Development

### Outcome

The player still spawns from the same position and collides with the ground. I have created larger ground for the purpose of testing the movement. The keys were assigned using phaser's inbuilt system, as was the scaling. Collision detection was implemented in this step so that the acceleration was compatible with the system I wanted to use for deceleration and jumping in the future, in hopes of minimising errors and edits in the future. I decided to have components of the speed the player can travel at held as separate values for similar reasons as the collision detection: by implementing it this way, I can ensure deceleration, turning and interactions with speed boosts/dashes/slides (should I choose to implement them) will require less editing, and acceleration will already work with them. The server required no changes, and the html just had a favicon added.

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
    <link rel="icon" type="image/x-icon" href="favicon.ico?v=1" />
    <script src="//cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.js"></script>
    <style>
        body {
            margin: 0;
        }
    </style>
</head>
<body>

    <script type="text/javascript">
        //we do a little programming
        var config = {
            type: Phaser.AUTO,

            scale: {
                mode: Phaser.Scale.FIT,
                autoCenter: Phaser.Scale.CENTER_BOTH,
                width: 800,
                height: 600
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
                update: update,
            },
        };

        //game objects
        var player;
        var platforms;
        var inputKeys;

        //values
        var accelerator = 10;
        var baseVelocity = 100;
        var velocity = baseVelocity;
        var maxVelocity = 500;
        var isGrounded = false;
        var canRun = false;

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
            ground.create(116, 584, 'ground').setScale(2).refreshBody();
            ground.create(244, 584, 'ground').setScale(2).refreshBody();
            ground.create(400, 584, 'ground').setScale(2).refreshBody();
            ground.create(656, 584, 'ground').setScale(2).refreshBody();
            ground.create(784, 584, 'ground').setScale(2).refreshBody();

            //Player settings
            player = this.physics.add.image(400, 170, 'player');

            //This makes sure the player doesn't run off screen. This is only temporary for testing purposes.
            player.setCollideWorldBounds(true);

            //Player will obey the laws of physics, and not phase through solid ground
            this.physics.add.collider(player, ground,
                //this will check to see if the player is touching the ground or not
                function (_player, _ground) {
                    if (_player.body.touching.down && _ground.body.touching.up) {
                        isGrounded = true;
                    }

                });

            this.physics.add.collider(player, ground);

            inputKeys = this.input.keyboard.createCursorKeys();
        }

        function update() {
            if (isGrounded == true) {
                canRun = true;
                if (inputKeys.left.isDown && canRun == true) { //move left
                    if (velocity < maxVelocity) {
                        velocity += accelerator; 
                    }
                    else if (velocity > maxVelocity) {
                        velocity = maxVelocity;
                    }
                    player.setVelocityX(-velocity);
                }

                else if (inputKeys.right.isDown && canRun == true) { //move right
                     if (velocity < maxVelocity) {
                        velocity += accelerator;
                     }
                    player.setVelocityX(velocity);
                }

                else {
                    player.setVelocityX(0);
                    velocity = baseVelocity;
                    }
                }
            }

    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

I am overall dissatisfied with the way this current movement system feels. I was initially going to include deceleration in this cycle, however the implementation of it was sluggish and didn't function as intended. Furthermore, the acceleration feels and is linear, which also doesn't feel great. Initially I also had velocity as a negative and positive value for left and right as opposed to just moving the player at negative velocity for left whilst calculating it positively. The former version had functioning deceleration but only in one direction and would function inconsistently, breaking if the player pressed two keys at once.

I plan to redo this movement system in the next cycle using a system with friction and force to calculate velocity instead of using random values for velocity and acceleration.

## Testing

### Tests

| Test | Instructions                                                     | What I expect                                                                         | What actually happens | Pass/Fail |
| ---- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code                                                     | The player appears and falls onto a screen-wide platform                              | As expected           | Pass      |
| 2    | Arrow key pressed as player is in air                            | The player should not move                                                            | As expected           | Pass      |
| 3    | Arrow key tapped whilst player is on the ground                  | The player should move slightly in the pressed direction                              | As expected           | Pass      |
| 4    | Arrow key held down when grounded                                | The player should accelerate to a point, then continue moving (in correct direction)  | As expected           | Pass      |
| 5    | Arrow key held but the maxVelocity and baseVelocity are the same | The player should move but not accelerate as the maxVelocity has already been reached | As expected           | Pass      |
| 6    | Arrow key released                                               | The player stops moving                                                               | As expected           | Pass      |
| 7    | Arrow key released and then repressed and held again             | The player stops moving, then accelerates again from the baseVelocity                 | As expected           | Pass      |

### Evidence

![note: please replace this with a gif if possible](<../.gitbook/assets/image (1) (2).png>)
