# 2.2.5 Cycle 4

## Design

### Objectives

* [ ] Have acceleration as a result of forces acting upon the player
* [ ] Velocity as a vector quantity
* [ ] Acceleration and deceleration
* [ ] Deceleration when turning

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

no new variables

| Variable Name | Use                                                            |
| ------------- | -------------------------------------------------------------- |
| finalVel      | final velocity and ultimately the velocity the player moves at |
| resForce      | the modelled amount of resisting forces to the player          |
| drivingForce  | the modelled amount of driving forces upon the player          |
| isGrounded    | is the player touching the ground                              |
| canRun        | determine whether the player is able to accelerate or not      |
| playerMass    | model for the player's mass                                    |

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
    end if    
    if inputKeys.left is down and canRun = true
            drivingForce = negative
            resForce = finalVel^2 / 2
            finalVel += (drivingForce - resForce) / 2
        elif inputKeys.right is down and canRun = true
            drivingForce = positive
            resForce = finalVel^2 / 2
            finalVel += (drivingForce - resForce) / 2
        elif 
    end if
end procedure
```

## Development

### Outcome

This method for finding the velocity using a force is based off of the work energy principal which I attempted to model in the previous examples.&#x20;

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

        //values for acceleration
        var finalVel = 0;
        var playerMass = 2;
        var drivingForce = 0;
        var resForce = 0;
        
        var isGrounded = false;
        var canRun = false;

        var game = new Phaser.Game(config);

        function preload() {
            this.load.image('background', '/background.png');
            this.load.image('ground', '/ground.png');
            this.load.image('player', '/player.png');
        }

        function create() {
            text = this.add.text(10, 10, '', { font: '16px Courier', fill: '#00ff00' });

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
            player = this.physics.add.image(400, 170, 'player').setScale(0.5);

            //This makes sure the player doesn't run off screen. This is only temporary for testing purposes.
            player.setCollideWorldBounds(true);

            //Player will obey the laws of physics, and not phase through solid ground
            this.physics.add.collider(player, ground,
                //this will check to see if the player is touching the ground or not
                function (_player, _ground) {
                    if (_player.body.touching.down && _ground.body.touching.up) {
                        isGrounded = true;
                    }
                    else { isGrounded = false; }

                });

            inputKeys = this.input.keyboard.createCursorKeys();
        }

        function update() {

            initialVel = finalVel;

            text.setText([
                'x pos: ' + player.x, //debug text. tells location of player, velocity, etc.
                'finVel: ' + finalVel,
                'Grounded? ' + isGrounded,
                'resistance: ' + resForce,
                'driving: ' + drivingForce,
                'mass: ' + playerMass
            ]);

            if (isGrounded) {
                canRun = true;
            }

            if (canRun && inputKeys.left.isDown) {
                drivingForce = -70;
            }
            else if (canRun && inputKeys.right.isDown) {
                drivingForce = 70;
            }
            else if (isGrounded && finalVel != 0) {
                drivingForce = 0
            }
            else {
                drivingForce = 0;
                resForce = 0;
                player.setVelocityX(0);
                finalVel = 0;
            }
            resForce = 0.4 * (finalVel / 2);
            finalVel = parseInt(finalVel + ((drivingForce - resForce) / playerMass));
            player.setVelocityX(finalVel);
        }

    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

The velocity would not stop increasing once a key was held, so the resisting force was never enough. The velocity would also increase to a point then suddenly go back to 0 and loop round again, also weather or not a key was pressed. Furthermore, trying to get the function to give an output caused it to constantly return 0, which is why it was temporarily abandoned.

With all this trouble this implementation of acceleration has caused, I plan to partially scrap it now. I will use a similar premise but instead of simulating the real life system, just model it in a way that feels good, and more importantly, functions.

## Testing

### Tests

| Test | Instructions                                                    | What I expect                                                                           | What actually happens                                           | Pass/Fail |
| ---- | --------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------- | --------- |
| 1    | Run the code                                                    | A white rectangle falls onto some grey platforms on a black background.                 | As expected                                                     | Pass      |
| 2    | let the player touch the ground                                 | Is grounded should be true, the player should not do anything else.                     | As expected                                                     | Pass      |
| 3    | tap the arrow keys in their respective directions               | The player should move in the pressed direction and stop.                               | As expected                                                     | Pass      |
| 4    | hold then release the arrow keys in their respective directions | The player should accelerate up to a point and then decelerate when the key is released | The player keeps going. They will not stop. They cannot stop... | Fail      |
| 5    | Key is pressed in the opposite direction when at full speed     | Should rapidly decelerate and turn in the opposite direction                            | They just turn instantly ay full speed                          | Fail      |

### Evidence

![](<../.gitbook/assets/image (6).png>)

Letting go won't stop it. It keeps going in a loop in either direction.
