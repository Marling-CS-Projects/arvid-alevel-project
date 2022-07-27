# 2.2.6 Cycle 5

## Design

### Objectives

* [ ] make a new functional system of gravity
* [ ] gravity should accelerate the player up to a point, but that point should be quite large
* [ ] add a jump
* [ ] jump should have variable height based off of how long the key is held

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name        | Use                               |
| -------------------- | --------------------------------- |
| finalVerticalVel     | final vertical velocity component |
| drivingVerticalForce | driving force vertically          |
| resVerticalForce     | resisting force vertically        |
| playerMass           | player mass                       |

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
            if facingRight
                turn
        elif inputKeys.right is down and canRun = true
            drivingForce = positive
            if facingLeft
                turn
        elif 
    end if
    resForce = finalVel / 2
    finalVel += (drivingForce - resForce) / 2
    move player finalVel
end procedure
```

## Development

### Outcome

This method for finding the velocity using a force is based off of the work energy principal which I attempted to model in the previous examples. The same benefits of implementing movement boosts such as dashes and slides still remain, but this is a far more simple implementation. The way the turn is implemented is based off of Rain World, where a small boost is gained while turning.

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
        var turningLeft = false;
        var turningRight = false;
        
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
                'mass: ' + playerMass,
                'turning? ' + 'Left: ' + turningLeft + ' Right: ' + turningRight
            ]);

            if (isGrounded) {
                canRun = true;
            }

            if (canRun && inputKeys.left.isDown) {
                if (drivingForce >= 90 && !turningLeft) {
                    /*this checks for weather or not the player can perform a turn boost. 90 allows for input imperfections as it is one frame
                     worth of deceleration below the top speed making it easier to perform*/
                    turningLeft = true; //turning left and right had to be separated into 2 variable to prevent interactions between them
                }
                else if (drivingForce > -200 && turningLeft) {
                    drivingForce += -20; //this makes you move faster when you're turning
                    if (drivingForce <= -200) { //this allows a greater top speed just after turning
                        turningLeft = false; //and once that top speed is reached, you are no longer considered to be turning
                        drivingForce = -200;
                    }
                }
                else if (drivingForce > -100 && !turningLeft) { //force is a positive or negative quantity as velocity is a vector
                    drivingForce += -10;
                }
                else if (drivingForce < -100 && !turningLeft && isGrounded) {
                    drivingForce += 10; //this checks if the player is travelling above the maximum base speed and decelerates them accordingly
                }
            }
            else if (canRun && inputKeys.right.isDown) {
                if (drivingForce <= -90 && !turningRight) {
                    turningRight = true;
                }
                else if (drivingForce < 200 && turningRight) {
                    drivingForce += 20;
                    if (drivingForce >= 200) {
                        turningRight = false;
                        drivingForce = 200;
                    }
                }
                else if (drivingForce < 100 && !turningRight) {
                    drivingForce += 10;
                }
                else if (drivingForce > 100 && !turningRight && isGrounded) {
                    drivingForce += -10;
                }
            }
            else {
                if (drivingForce != 0) {
                    if (drivingForce < 0) {
                        drivingForce += 10;
                    }
                    else {
                        drivingForce += -10;
                    }
                }
                else {
                    drivingForce = 0;
                }
            }
            resForce = 0.7 * (finalVel / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
            finalVel = parseInt(finalVel + ((drivingForce - resForce) / playerMass));
            /*calculates horizontal velocity based off of the 'forces' applied. Doing it this way means dashes and extra movement boosts
             can be implemented by saying they apply a larger force*/
            player.setVelocityX(finalVel); //move the player
        }

    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

Figuring out the acceleration based off of force and calculating it as a vector wasn't too hard to do, based off of what I have learned from the previous cycles. The turn was the most difficult part of this cycle, as the way I initially implemented it didn't distinguish between the different directions to see if the player was dashing. I fixed this by using two separate variables to determine which direction the player was turning in.

## Testing

### Tests

| Test | Instructions                                                              | What I expect                                                                                                                                               | What actually happens | Pass/Fail |
| ---- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code                                                              | A white rectangle falls onto some grey platforms on a black background.                                                                                     | As expected           | Pass      |
| 2    | let the player touch the ground                                           | No movement, grounded = true and canRun = true                                                                                                              | As expected           | Pass      |
| 3    | Press key in either direction                                             | Player should reach a maximum velocity and accelerate gradually                                                                                             | As expected           | Pass      |
| 4    | Release Key after allowing for full acceleration                          | Player decelerates back down to a stop                                                                                                                      | As expected           | Pass      |
| 5    | Change the resistance multiplier to increase/decrease and repeat 3 and 4. | <p>Acceleration should decrease/increase and deceleration should increase/decrease respectively.<br>Max velocity should decrease/increase respectively.</p> | As expected           | Pass      |
| 6    | Don't allow for full deceleration, and press opposite direction.          | Slower acceleration to the opposite direction                                                                                                               | As expected           | Pass      |
| 7    | Turn while at the full velocity in the opposite direction                 | turnDireciton = true for the direction turned, slight acceleration and rapid deceleration after threshold                                                   | As expected           | Pass      |

### Evidence

![](<../.gitbook/assets/image (4).png>)
