# 2.2.8 Cycle 7

## Design

### Objectives

* [x] make a jump
* [x] the jump should be variable in height based off of how long a key is held

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name        | Use                                         |
| -------------------- | ------------------------------------------- |
| finalVerticalVel     | final vertical velocity component           |
| drivingVerticalForce | driving force vertically                    |
| resVerticalForce     | resisting force vertically                  |
| playerMass           | player mass                                 |
| isJumping            | checks if the player in currently in a jump |
| hasJumped            | checks if the player has already jumped     |
| canJump              | checks if the player is able to jump        |

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
        if inputKeys.jump is up
            canJump = true
        end if
        if hasJumped = true
            hasJumped = false
        end if
        if isJumping = false
            drivingVertcalForce = 0
            finalVerticalVel = 0
        end if
    end if    
    if inputKeys.left is down and canRun = true
            drivingHorizontalForce = negative
            if facingRight
                turn
        elif inputKeys.right is down and canRun = true
            drivingHorizontalForce = positive
            if facingLeft
                turn
    end if
    
    if inputKeys.jump is down and canJump = true
        drivingVerticalForce = -70
        isJumping = true
    end if
    if isJumping = true and inputKeys.jump is up
        hasJumped = true
        isJumping = false
    end if
    if isJumping = true and drivingVerticalForce = 0
        hasJumped = true
        isJumping = false
    end if
    
    if isGrounded = false
        drivingVerticalForce = 10
    end if
        
    resForce = finalVel / 2
    finalVel += (drivingHorizontalForce - resHorizontalForce) / 2
    move player finalHorizontalVel
    resForce = finalVel / 2
    finalVel += (drivingVerticalForce - resVerticalForce) / 2
    move player finalVerticalVel
end procedure
```

## Development

### Outcome

I started by creating 3 separate variables which checked for if a player could jump, was currently jumping or had already jumped. While the player is in an active jump, releasing the correct key stops it early and also allows for future checks like enabling rolls. Checking if the player has jumped disables the jump key from functioning again till it is both up and the player is grounded for at least 1 frame.

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

            text.setText([
                'canJump: ' + canJump,
                'Vertical Force: ' + drivingVerticalForce,
                'Vertical Resistance: ' + resVerticalForce,
                'Final Vel: ' + finalVerticalVel,
                'grounded? ' + isGrounded,
                'jumping? ' + isJumping,
                'has jumped? ' + hasJumped
            ]);

            if (isGrounded) {
                canRun = true;
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
            }

            if (!isGrounded && drivingVerticalForce != 10 && !isJumping) {
                drivingVerticalForce += 1; //this is now the weight force
            }
            else if (!isGrounded && drivingVerticalForce != 10 && isJumping) {
                drivingVerticalForce += 10;
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
            //... and ends here

            //vertical movement starts here...
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

            resHorizontalForce = 0.65 * ((finalHorizontalVel) / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
            finalHorizontalVel = parseInt(finalHorizontalVel + ((drivingHorizontalForce - resHorizontalForce) / playerMass));
            /*calculates horizontal velocity based off of the 'forces' applied. Doing it this way means dashes and extra movement boosts
             can be implemented by saying they apply a larger force*/
            player.setVelocityX(finalHorizontalVel); //move the player

            resVerticalForce = 0.0001 * ((finalVerticalVel ^ 2) / 2);
            finalVerticalVel = parseInt(finalVerticalVel + (drivingVerticalForce - resVerticalForce));
            player.setVelocityY(finalVerticalVel);
        }
    </script>

</body>
</html>
```
{% endtab %}
{% endtabs %}

### Challenges

The main challenge was getting the movement to feel smooth: this meant the jump had to feel snappy but still obey the same laws as the rest of the movement system. Along with ensuring the player could not repeatedly jump when holding down a key, and that releasing the key would allow for another jump, when touching the ground, mean the creation of several new variables and repetitions of statements that felt redundant.

Another challenge is the way horizontal movement interacts with the player being grounded. Currently, the player will only continue to move horizontally when grounded, whereas the intent was to allow horizontal movement to continue, and only allow forces to be applied when grounded. Currently, the player stops all horizontal movement when airborne, so this will have to be fixed.

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

![](<../.gitbook/assets/image (1) (1) (1) (2).png>)
