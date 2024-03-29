# 2.2.7 Cycle 6

## Design

### Objectives

* [x] make a new functional system of gravity
* [x] remake collision detection

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
        canJump = true
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

I am reusing the force method from the previous movement. This is for similar reasons, as it allows for better implementation of more complicated movement systems. The gravity uses this system with very little resistive force to ensure acceleration and top speed when falling is very high.

I have had to change the way collision is detected to ensure the method of creating vertical movement I wanted to use will work. Previously, the collider was a function which would only ever check if the player and ground were touching, not when they stopped. The new method checks each frame for touching the ground, and returns false if anything other than this is the case.

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
                'grounded? ' + isGrounded
            ]);

            if (isGrounded) {
                canRun = true;
                resVerticalForce = 100;
                if (!canJump && movementKeys.jump.isUp) {
                    canJump = true;
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

            if (!isGrounded && drivingVerticalForce != 10) {
                drivingVerticalForce += 1; //this is now the weight force
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
            if (isGrounded && movementKeys.jump.isDown) {
                player.y = 0;
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

The challenge was, once new collision and gravity were made, to reset the velocity of the player when they touched the ground, as touching the ground would not apply any force, just stop the player from moving. This meant the player would start falling faster if they fell too soon after a previous fall.

## Testing

### Tests

| Test | Instructions                                                         | What I expect                                                                                                                | What actually happens | Pass/Fail |
| ---- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code                                                         | A white rectangle falls onto some grey platforms on a black background.                                                      | As expected           | Pass      |
| 2    | Allow the player to fall                                             | Should accelerate downwards, stop when hitting the ground                                                                    | As expected           | Pass      |
| 3    | Stay grounded                                                        | isGrounded and canJump should be true, vertical force and resistance should be 0                                             | As expected           | Pass      |
| 4    | Press the jump key                                                   | Player should instantly teleport to the top of the screen, isGrounded and canJump = false, player should fall and accelerate | As expected           | Pass      |
| 5    | Repeatedly press the jump key before the player can reach the ground | finalVerticalVel should keep increasing, isGrounded and canJump stay false, Vertical Force should be a constant 10           | As expected           | Pass      |

### Evidence

![](<../.gitbook/assets/image (5).png>)
