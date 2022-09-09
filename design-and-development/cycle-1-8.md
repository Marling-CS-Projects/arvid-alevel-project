# 2.2.9 Cycle 8

## Design

### Objectives

* [x] Ensure the horizontal movement works when airborne
* [x] Add complexed movement interaction for flips and slides

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name | Use                                                             |
| ------------- | --------------------------------------------------------------- |
| isAirborne    | checks to see if the player is not on the ground                |
| rollCheck     | checks to see if the player can roll                            |
| rolling       | checks the state of the player - weather they're rolling or not |
| rolled        | checks to see if the roll is complete                           |

### Pseudocode

To keeps things clean, I'll only include relevant changes in pseudocode

```
if airborne = true
    if inputKeys.down isDown
        rollCheck = true
    endif
endif

if isGrounded and rollCheck = true
    drivingHorizontalForce = greater than walking in correct direction
endif    
```

## Development

### Outcome

I added a check to the horizontal movement section that ensured acceleration and applied forces would only occur when grounded and made momentum carry continue regardless of being grounded or not. This means the player can't&#x20;

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
        var slideThresholdVel = 300;
        var canSlide = false;
        var sliding = false;

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
                'grounded: ' + isGrounded,
                'horizontalVel: ' + finalHorizontalVel,
                'canRun: ' + canRun,
                'canSlide: ' + canSlide,
                'driving horizontal: ' + drivingHorizontalForce
            ]);

            if (isGrounded) {
                canRun = true;
                if (!sliding) {
                    if (slideThresholdVel <= Math.abs(finalHorizontalVel)) {
                        canSlide = true
                    }
                    else if (slideThresholdVel > Math.abs(finalHorizontalVel)) {
                        canSlide = false
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

            //horizontal movement starts here...
            if (canRun && movementKeys.left.isDown) {
                if (drivingHorizontalForce >= 90 && !turningLeft) {
                    /*this checks for weather or not the player can perform a turn boost. 90 allows for input imperfections as it is one frame
                     worth of deceleration below the top speed making it easier to perform*/
                    turningLeft = true; //turning left and right had to be separated into 2 variable to prevent interactions between them
                }
                else if (drivingHorizontalForce > -200 && turningLeft) {
                    drivingHorizontalForce += -20; //this makes you move faster when you're turning
                    if (drivingHorizontalForce <= -200) { //this allows a greater top speed just after turning
                        turningLeft = false; //and once that top speed is reached, you are no longer considered to be turning
                        drivingHorizontalForce = -200;
                    }
                }
                else if (drivingHorizontalForce > -100 && !turningLeft) { //force is a positive or negative quantity as velocity is a vector
                    drivingHorizontalForce += -10;
                }
                else if (drivingHorizontalForce < -100 && !turningLeft && isGrounded) {
                    drivingHorizontalForce += 10; //this checks if the player is travelling above the maximum base speed and decelerates them accordingly
                }
            }
            else if (canRun && movementKeys.right.isDown) {
                if (drivingHorizontalForce <= -90 && !turningRight) {
                    turningRight = true;
                }
                else if (drivingHorizontalForce < 200 && turningRight) {
                    drivingHorizontalForce += 20;
                    if (drivingHorizontalForce >= 200) {
                        turningRight = false;
                        drivingHorizontalForce = 200;
                    }
                }
                else if (drivingHorizontalForce < 100 && !turningRight) {
                    drivingHorizontalForce += 10;
                }
                else if (drivingHorizontalForce > 100 && !turningRight && isGrounded) {
                    drivingHorizontalForce += -10;
                }
            }
            else {
                drivingHorizontalForce = drivingHorizontalForce;
            }
            //... and ends here

            //vertical movement starts here...
            if (!isGrounded && drivingVerticalForce != 10 && !isJumping) {
                drivingVerticalForce += 1; //this is now the weight force
            }
            else if (!isGrounded && drivingVerticalForce != 10 && isJumping) {
                drivingVerticalForce += 10;
            }
            if (isGrounded && movementKeys.jump.isDown && !hasJumped && canJump) {
                drivingVerticalForce = -70;
                isJumping = true; //allows for checks of an active jump
                if (finalVerticalVel > 0) {
                    finalVerticalVel = 0; //this cancels the passive downward force present on the first jump of the frame
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
                if (drivingHorizontalForce < 0) {
                    drivingHorizontalForce += -50;
                }
                if (drivingHorizontalForce > 0) {
                    drivingHorizontalForce += 50;
                    
                }
            }
            //... and ends here
            if (isGrounded && movementKeys.right.isUp && movementKeys.left.isUp) {
                if (drivingHorizontalForce != 0) {
                    if (drivingHorizontalForce < 0) {
                        drivingHorizontalForce += 10;
                    }
                    else {
                        drivingHorizontalForce += -10;
                    }
                }
                else {
                    drivingHorizontalForce = 0;
                }
            }

            resHorizontalForce = 0.65 * ((finalHorizontalVel) / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
            finalHorizontalVel = parseInt(finalHorizontalVel + ((drivingHorizontalForce - resHorizontalForce) / playerMass));
            /*calculates horizontal velocity based off of the 'forces' applied. Doing it this way means dashes and extra movement boosts
             can be implemented by saying they apply a larger force*/
            player.setVelocityX(finalHorizontalVel); //move the player

            resVerticalForce = 0.0001 * ((finalVerticalVel ^ 2) / 2);
            finalVerticalVel = parseInt(finalVerticalVel + (drivingVerticalForce - resVerticalForce));
            player.setVelocityY(finalVerticalVel);
            //louie is in pain. perfect.
        }
    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

&#x20;There wasn't anything

## Testing

### Tests

| Test | Instructions                                                                           | What I expect                                                                                                                                                                                          | What actually happens | Pass/Fail |
| ---- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------- | --------- |
| 1    | Run the code                                                                           | A white rectangle falls onto some grey platforms on a black background.                                                                                                                                | As expected           | Pass      |
| 2    | Allow the player to fall                                                               | Should accelerate downwards, stop when hitting the ground                                                                                                                                              | As expected           | Pass      |
| 3    | Stay grounded                                                                          | isGrounded and canJump should be true, vertical force and resistance should be 0                                                                                                                       | As expected           | Pass      |
| 4    | Hold the jump key                                                                      | Player should jump, and finalVerticalVelocity should start at -70 and rapidly decrease while the key is held. isJumping = true                                                                         | As expected           | Pass      |
| 5    | The key is held till the player reaches the ground                                     | Player should eventually return to the ground, isJumping should be true on the way up, false on the way down, vise versa for hasJumped. Can jump should remain false as the player touches the ground. | As expected           | Pass      |
| 6    | The key is released early                                                              | All the conditions from test 5 should hold true, the jump should end early and thus be smaller                                                                                                         | As expected           | Pass      |
| 7    | The key is released early and then pressed again before the player touches the ground. | The player's jump should be cut short and pressing the jump key again should not do anything.                                                                                                          | As expected           | Pass      |

### Evidence

![](<../.gitbook/assets/image (1).png>)
