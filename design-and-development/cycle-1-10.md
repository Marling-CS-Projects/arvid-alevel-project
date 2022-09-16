# 2.2.11 Cycle 10

## Design

### Objectives

* [ ] Make a world with multiple screens
* [x] Make a camera that will switch between the screens based off of the player's location

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.
* Visual Clarity - There are no filters, and all screens switch the same way and are the same size

### Key Variables

| Variable Name | Use                                     |
| ------------- | --------------------------------------- |
| camera        | the variable used to control the camera |
| player.x      | the player's x position                 |
| cameraX       | the camera's x position                 |

### Pseudocode

To keeps things clean, I'll only include relevant changes in pseudocode

```
camera.size = window width, window height
camera.position = x, y

if player.x > cameraX + window width
    cameraX += window width
    camera.position = x, y
end if
if player.x < cameraX
    cameraX += -window width
end if
```

## Development

### Outcome



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
                    gravity: { y: 0 }, //gravity is gone. you can fly now
                    debug: false,
                }
            },
            scene: {
                preload: preload,
                create: create,
                update: update,
            },
        };

        //variables are here
        var player;
        var platforms;
        var movementKeys;

        //values for acceleration
        var finalHorizontalVel = 0;
        var playerMass = 2;
        var drivingHorizontalForce = 0;
        var resHorizontalForce = 0;
        var turningLeft = false;
        var turningRight = false;

        //vertical movement
        var finalVerticalVel = 0;
        var drivingVerticalForce = 0;
        var resVerticalForce = 0;
        var hasJumped = false;
        var isJumping = false;
        var canJump = false;

        var isGrounded = false;
        var canRun = false;

        //special movement
        var canSlide = false;
        var sliding = false;
        var hasSlid = false;
        var slideDeceleration = Math.abs(drivingHorizontalForce) / 5;

        //movement values
        var slidingThreshold = 300;

        var maxRunningForce = 100;
        var runningAcceleration = 10;
        var runningDeceleration = 10;

        var maxTurningForce = 200;
        var turningThreshold = 90;
        var turningAcceleration = 20;
        var turningDeceleration = 10;

        var maxGravity = 10;
        var gravityAcceleration = 1;
        var jumpForce = 70;

        var maxSlideForce = 450;
        var slideAcceleration = 30;

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
            
            this.physics.add.collider(player, ground)

            movementKeys = this.input.keyboard.addKeys({
                left: 'left',
                right: 'right',
                down: 'down',
                jump: 'Z'
            })
        }

        function update() {

            if (player.body.touching.down) {
                isGrounded = true;
            }
            else if (!player.body.touching.down) {
                isGrounded = false;
            }

            //debug text for testing
            text.setText([
                'horizontalVel: ' + finalHorizontalVel,
                'canRun: ' + canRun,
                'canSlide: ' + canSlide,
                'sliding: ' + sliding,
                'hasSlid: ' + hasSlid,
                'driving horizontal: ' + drivingHorizontalForce
            ]);

            if (isGrounded) {
                if (!sliding && !hasSlid) {
                    canRun = true;
                }
                else if (sliding || hasSlid) {
                    canRun = false;
                }
                if (!sliding && !hasSlid) {
                    if (Math.abs(finalHorizontalVel) >= slidingThreshold) {
                        canSlide = true;
                    }
                    else {
                        canSlide = false;
                    }
                }
                
                if (!canJump && movementKeys.jump.isUp) {
                    canJump = true;
                    hasJumped = false;
                }

                if (!isJumping) {
                    /*this is here to fix a bug present when the player tried to jump and wouldn't move as they were in contact
                    with the ground when trying to start the jump. As a result, their vertical velocity would reset to 0 and
                    they wouldn't move. To fix it, the jumping variable was added to check if the player was trying to jump.*/
                    drivingVerticalForce = 0;
                    finalVerticalVel = 2;
                }
            }
            else {
                canRun = false;
                canJump = false;
                canSlide = false;
            }

            if (movementKeys.down.isUp && hasSlid) {
                hasSlid = false;
            }

            //horizontal movement starts here...
            if (canRun && movementKeys.left.isDown) {
                if (drivingHorizontalForce >= turningThreshold && !turningLeft) {
                    /*this checks for weather or not the player can perform a turn boost. 90 allows for input imperfections as it is one frame
                     worth of deceleration below the top speed making it easier to perform*/
                    turningLeft = true; //turning left and right had to be separated into 2 variable to prevent interactions between them
                }
                else if (drivingHorizontalForce > -maxTurningForce && turningLeft) {
                    drivingHorizontalForce += -turningAcceleration; //this makes you move faster when you're turning
                    if (drivingHorizontalForce <= -maxTurningForce) { //this allows a greater top speed just after turning
                        turningLeft = false; //and once that top speed is reached, you are no longer considered to be turning
                        drivingHorizontalForce = -maxTurningForce;
                    }
                }
                else if (drivingHorizontalForce > -maxRunningForce && !turningLeft) { //force is a positive or negative quantity as velocity is a vector
                    drivingHorizontalForce += -runningAcceleration;
                }
                else if (drivingHorizontalForce < -maxRunningForce && !turningLeft && isGrounded) {
                    drivingHorizontalForce += turningDeceleration; //this checks if the player is travelling above the maximum base speed and decelerates them accordingly
                }
            }
            else if (canRun && movementKeys.right.isDown) {
                if (drivingHorizontalForce <= -turningThreshold && !turningRight) {
                    turningRight = true;
                }
                else if (drivingHorizontalForce < maxTurningForce && turningRight) {
                    drivingHorizontalForce += turningAcceleration;
                    if (drivingHorizontalForce >= maxTurningForce) {
                        turningRight = false;
                        drivingHorizontalForce = maxTurningForce;
                    }
                }
                else if (drivingHorizontalForce < maxRunningForce && !turningRight) {
                    drivingHorizontalForce += runningAcceleration;
                }
                else if (drivingHorizontalForce > maxRunningForce && !turningRight && isGrounded) {
                    drivingHorizontalForce += -runningAcceleration;
                }
            }
            else {
                drivingHorizontalForce = drivingHorizontalForce;
            }
            //... and ends here

            //vertical movement starts here...
            if (!isGrounded && drivingVerticalForce != maxGravity && !isJumping) {
                drivingVerticalForce += gravityAcceleration; //this is now the weight force, so gravity is back. no more flight.
            }
            else if (!isGrounded && drivingVerticalForce != maxGravity && isJumping) {
                drivingVerticalForce += maxGravity;
            }
            if (isGrounded && movementKeys.jump.isDown && !hasJumped && canJump) {
                drivingVerticalForce = -jumpForce;
                isJumping = true; //allows for checks of an active jump
                if (finalVerticalVel > 0) {
                    finalVerticalVel = 0; //this cancels the passive downward force present on the first frame of the jump
                }
            }
            if (movementKeys.jump.isUp && isJumping) {
                    drivingVerticalForce = 0;
                    isJumping = false;
                    hasJumped = true; //ensures no more jumps can be done before the player touches the ground
            }
            if (isJumping && drivingVerticalForce == 0) {
                    isJumping = false;
                    hasJumped = true;
            }
            //... and ends here

            //special movement starts here...
            if (movementKeys.down.isDown && canSlide) {
                sliding = true;
                canSlide = false;
            }
            if (drivingHorizontalForce < 0 && sliding && drivingHorizontalForce > -maxSlideForce) {
                drivingHorizontalForce += -slideAcceleration;
            }
            else if (sliding && drivingHorizontalForce <= -maxSlideForce) {
                sliding = false;
                hasSlid = true;
            }
            if (drivingHorizontalForce > 0 && sliding && drivingHorizontalForce < maxSlideForce) {
                drivingHorizontalForce += slideAcceleration;
            }
            else if (sliding && drivingHorizontalForce >= maxSlideForce) {
                sliding = false;
                hasSlid = true;
            }
            //... and ends here

            if (isGrounded && movementKeys.right.isUp && movementKeys.left.isUp || isGrounded && hasSlid) {
                if (!hasSlid && drivingHorizontalForce != 0) {
                    if (drivingHorizontalForce < 0) {
                        drivingHorizontalForce += runningDeceleration;
                    }
                    else {
                        drivingHorizontalForce += -runningDeceleration;
                    }
                }
                else {
                    drivingHorizontalForce = 0;
                }
                if (hasSlid && drivingHorizontalForce != 0) {
                    if (drivingHorizontalForce < 0) {
                        drivingHorizontalForce += slideDeceleration;
                    }
                    else {
                        drivingHorizontalForce += -slideDeceleration;
                    }
                }
                else {
                    drivingHorizontalForce = 0;
                }
            }
            

            //Ben Cook has no comments
            resHorizontalForce = 0.65 * ((finalHorizontalVel) / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
            finalHorizontalVel = parseInt(finalHorizontalVel + ((drivingHorizontalForce - resHorizontalForce) / playerMass));
            /*calculates horizontal velocity based off of the 'forces' applied. Doing it this way means dashes and extra movement boosts
             can be implemented by saying they apply a larger force*/
            player.setVelocityX(finalHorizontalVel); //move the player

            resVerticalForce = 0.0001 * ((finalVerticalVel ^ 2) / 2);
            finalVerticalVel = parseInt(finalVerticalVel + (drivingVerticalForce - resVerticalForce));
            player.setVelocityY(finalVerticalVel);
            //louie is in pain from if statement overload. perfect.
        }
    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges



## Testing

### Tests

| Test | Instructions | What I expect                                                           | What actually happens | Pass/Fail |
| ---- | ------------ | ----------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code | A white rectangle falls onto some grey platforms on a black background. | As expected           | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
