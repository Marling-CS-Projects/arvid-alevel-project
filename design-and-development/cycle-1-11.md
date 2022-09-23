# 2.2.12 Cycle 11

## Design

### Objectives

* [x] Make walls and the ability to slide down them
* [x] Add an ability to hold onto the wall
* [x] Make a wall jumping system

### Usability Features

* Visual Clarity - There is a clear distinction between different terrains (ground and wall), and the player is easy to see on screen.
* Simplified controls - the controls are consistent between forms: jumping on the ground and walls are the same.

### Key Variables

| Variable Name  | Use                                                                           |
| -------------- | ----------------------------------------------------------------------------- |
| touchingWall   | touching a wall or not                                                        |
| left/rightWall | if the player is touching a wall, it checks to see which side the wall is on. |
| wallJump       | a check for if a wall jump is charged                                         |
| wallSlide      | a check that enables/disables wallJump and slows falls on walls.              |

### Pseudocode

To keeps things clean, I'll only include relevant changes in pseudocode

```
if player touches wall
    touchingWall = true
    if playerLeft touches wall
        leftWall = true
    end if
    if playerRight touches wall
        rightWall = true
    end if
end if

if touchingWall = true & wallSlide = false
    resVerticalForce = resVerticalForce * 10
    wallSlide = true
else
    resVerticalForce = resVerticalForce / 10
    wallSlide = false
end if

if movementKeys.jump.isDown & wallSlide
    wallJump = true
end if

if movementKeys.jump.isUp & wallJump
    if leftWall
        drivingVerticalForce = up
        drivingHorizontalForce = right
    else
        drivingVerticalForce = up
        drivingHorizontalForce = left
    end if
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
                width: 1024,
                height: 512
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

        //VAR GANG
        //variables are here
        var player;
        var platforms;
        var movementKeys;

        //collision checks
        var isGrounded = false;
        var leftWall = false;
        var rightWall = false;
        var touchingWall = false;

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

        var canRun = false;

        var wallSlide = false;
        var gravityDragMultiplier = 0.0001;
        var wallJump = false;

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
        var gravityAcceleration = 2;
        var jumpForce = 70;

        var maxSlideForce = 450;
        var slideAcceleration = 30;

        var wallJumpVert = 10;
        var wallJumpHoriz = 50;

        //camera controls
        var leftScreen = 0;
        var rightScreen = 1024;
        var screenWidth = 1024;

        var game = new Phaser.Game(config);

        function preload() {
            this.load.image('background', '/background.png');
            this.load.image('ground', '/ground.png');
            this.load.image('player', '/player.png');
            this.load.image('wall', '/wall.png');
        }

        function create() {

            var camera = this.cameras.main;

            text = this.add.text(10, 10, '', { font: '16px Courier', fill: '#00ff00' }).setScrollFactor(0);

            //camera
            camera.setZoom(1);

            //Background
            this.add.image(400, 300, 'background');

            //The things we stand on
            ground = this.physics.add.staticGroup();
            wall = this.physics.add.staticGroup();
            //---------------- secret scene ----------------
            wall.create(-1008, 448, 'wall').setScale(2).refreshBody();
            wall.create(-1008, 384, 'wall').setScale(2).refreshBody();

            ground.create(-896, 496, 'ground').setScale(2).refreshBody();
            ground.create(-640, 496, 'ground').setScale(2).refreshBody();
            ground.create(-384, 496, 'ground').setScale(2).refreshBody();
            ground.create(-128, 496, 'ground').setScale(2).refreshBody();
            //---------------- secret scene ----------------

            //---------------- first scene ----------------
            ground.create(128, 496, 'ground').setScale(2).refreshBody();
            ground.create(384, 496, 'ground').setScale(2).refreshBody();
            ground.create(640, 496, 'ground').setScale(2).refreshBody();
            ground.create(896, 496, 'ground').setScale(2).refreshBody();
            //---------------- first scene ----------------

            //---------------- second scene ----------------
            ground.create(1152, 496, 'ground').setScale(2).refreshBody();
            ground.create(1408, 496, 'ground').setScale(2).refreshBody();

            ground.create(1664, 496, 'ground').setScale(2).refreshBody();
            ground.create(1536, 496, 'ground').setScale(2).refreshBody(); //this is here to prevent a physics bug where the player falls through the floor between gaps in the ground
            ground.create(1664, 464, 'ground').setScale(2).refreshBody(); //up a floor

            ground.create(1920, 496, 'ground').setScale(2).refreshBody();
            ground.create(1792, 464, 'ground').setScale(2).refreshBody(); //this is here to prevent a physics bug where the player falls through the floor between gaps in the ground
            ground.create(1920, 432, 'ground').setScale(2).refreshBody(); //up another
            //---------------- second scene ----------------

            //---------------- third scene ----------------
            ground.create(2176, 496, 'ground').setScale(2).refreshBody(); //stay on the upper floor
            ground.create(2176, 432, 'ground').setScale(2).refreshBody();

            ground.create(2432, 496, 'ground').setScale(2).refreshBody();
            ground.create(2432, 432, 'ground').setScale(2).refreshBody();

            ground.create(2688, 496, 'ground').setScale(2).refreshBody();
            ground.create(2688, 432, 'ground').setScale(2).refreshBody();

            ground.create(2944, 496, 'ground').setScale(2).refreshBody();
            ground.create(2944, 432, 'ground').setScale(2).refreshBody();
            //---------------- third scene ----------------

            //Player settings
            player = this.physics.add.image(400, 170, 'player').setScale(0.5);
            player.setCollideWorldBounds(false);

            //Player will obey the laws of physics, and not phase through solid ground
            
            this.physics.add.collider(player, ground);
            this.physics.add.collider(player, wall);

            movementKeys = this.input.keyboard.addKeys({
                left: 'left',
                right: 'right',
                down: 'down',
                jump: 'Z'
            })
        }

        function update() {

            var camera = this.cameras.main;

            if (player.x > rightScreen) {
                camera.scrollX = rightScreen;
                leftScreen += screenWidth;
                rightScreen += screenWidth;
            }
            if (player.x < leftScreen) {
                leftScreen += -screenWidth;
                rightScreen += -screenWidth;
                camera.scrollX = leftScreen;
            }

            //makes checks for the player touching what can be considered ground
            if (player.body.touching.down) {
                isGrounded = true;
            }
            else if (!player.body.touching.down) {
                isGrounded = false;
            }

            //this will check for any surface the player touches with their sides, and consider it a wall
            if (player.body.touching.left || player.body.touching.right) {
                touchingWall = true;
                if (player.body.touching.left) {
                    leftWall = true;
                }
                if (player.body.touching.right) {
                    rightWall = true;
                }
            }
            if (!player.body.touching.left && !player.body.touching.right) {
                touchingWall = false;
                leftWall = false;
                rightWall = false;
            }

            //debug text for testing
            text.setText([
                'player vel: ' + finalHorizontalVel,
                'leftWall: ' + leftWall,
                'rightWall: ' + rightWall,
                'touchingWall: ' + touchingWall,
                'canSlide: ' + canSlide,
                'canRun: ' + canRun,
                'hasSlid: ' + hasSlid,
                'sliding: ' + sliding
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

                if (!canJump && movementKeys.jump.isUp && !sliding) {
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
            if (canRun && movementKeys.left.isDown && !leftWall) {
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
            }   //tax evasion enabler
            else if (canRun && movementKeys.right.isDown && !rightWall) {
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
            if (touchingWall) {
                gravityDragMultiplier = 0.1;
                wallSlide = true;
            }
            if (!touchingWall) {
                gravityDragMultiplier = 0.0001;
                wallSlide = false;
            }

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

            if (wallJump && movementKeys.jump.isUp) {
                if (leftWall) {
                    touchingWall = false;
                    drivingHorizontalForce = wallJumpHoriz;
                    drivingVerticalForce = -wallJumpVert;
                }
                if (rightWall) {
                    touchingWall = false;
                    drivingHorizontalForce = -wallJumpHoriz;
                    drivingVerticalForce = -allJumpVert;
                }
                wallJump = false;
            }
            //... and ends here

            //special movement starts here...
            if (movementKeys.down.isDown && canSlide) {
                sliding = true;
                canSlide = false; //starts the slide if the player is able to
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

            //this sections controls horizontal forces and the amount applied. You decelerate faster when sliding than runnig, and instantly if you hit a wall.
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

            //wall checks
            if (touchingWall && !isGrounded) {
                drivingHorizontalForce = 0;
                if (leftWall) {
                    finalHorizontalVel = -2;
                }
                if (rightWall) {
                    finalHorizontalVel = 2;
                }
            }
            

            //Ben Cook has no comments to make on this program
            resHorizontalForce = 0.65 * ((finalHorizontalVel) / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
            finalHorizontalVel = parseInt(finalHorizontalVel + ((drivingHorizontalForce - resHorizontalForce) / playerMass));
            /*calculates horizontal velocity based off of the 'forces' applied. Doing it this way means dashes and extra movement boosts
             can be implemented by saying they apply a larger force*/
            player.setVelocityX(finalHorizontalVel); //move the player

            if (wallSlide) {
                if (movementKeys.jump.isDown) {
                    player.setVelocityY(0);
                    wallJump = true;
                }
                if (movementKeys.jump.isUp) {
                    resVerticalForce = gravityDragMultiplier * ((finalVerticalVel ^ 2) / 2);
                    finalVerticalVel = parseInt(finalVerticalVel + (drivingVerticalForce - resVerticalForce));
                    player.setVelocityY(finalVerticalVel);
                }
            }
            if (!wallSlide) {
                resVerticalForce = gravityDragMultiplier * ((finalVerticalVel ^ 2) / 2);
                finalVerticalVel = parseInt(finalVerticalVel + (drivingVerticalForce - resVerticalForce));
                player.setVelocityY(finalVerticalVel);
            }
            //louie is in pain from if statement overload. perfect.
        }
    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

The greatest challenge here was getting the player to stick to the wall and then be removed at the correct time. The way phaser handles collisions makes the player and wall push apart slightly, so I have to remove all forces being applied to the player while they're in contact with the wall and still apply movement in the direction of the wall to make the player stick till I want them removed.

## Testing

### Tests

| Test | Instructions                                        | What I expect                                                | What actually happens | Pass/Fail |
| ---- | --------------------------------------------------- | ------------------------------------------------------------ | --------------------- | --------- |
| 1    | Run the code                                        | Player should be created, fall onto a platform               | As expected           | Pass      |
| 2    | Jump towards a wall and release jump before hitting | Player should fall down next to wall, but slower than in air | As expected           | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
