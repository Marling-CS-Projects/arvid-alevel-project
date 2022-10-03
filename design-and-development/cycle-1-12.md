# 2.2.13 Cycle 12

## Design

### Objectives

* [x] Add a health bar that changes with health value
* [x] Add hazards and hostile NPCs that decrease health
* [x] Add a death state when health is 0
* [x] make parts of the level/screens

### Usability Features

* Visual Clarity - The health bar is colour coded and remains a constant size and position on the screen, enemies are easily distinguished from the terrain and player.
* Simplified controls - the controls are consistent between forms: jumping on the ground and walls are the same.

### Key Variables

| Variable Name | Use                                                             |
| ------------- | --------------------------------------------------------------- |
| playerHealth  | tracks the value of the player's health                         |
| playerDead    | sees if the player is dead or not                               |
| hit           | check for players getting hit, and later player hitting things. |

### Pseudocode

To keeps things clean, I'll only include relevant changes in pseudocode

```
playerHealth = 10

function hit(damage, target)
    target += -damage
end function

if player touches spike
    hit(1, playerHealth)
end if

if playerHealth <= 0
    playerDead = true
end if
```

## Development

### Outcome

I was create a health bar using a sprite which changed with the amount of health the player had. Using Phaser's custom overlap and collision detection, I created a function which would do a set amount of damage to a target, with a toggle to enable and disable immunity protection.&#x20;

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
    <title>e</title>
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
        //health stuff
        var playerHealth = 10;
        var lastKnownHealth = 10;
        var playerDead = false;
        var respawnDelay = 10;
        var iFrames = 0;

        //variables are here
        var player;
        var platforms;
        var movementKeys;

        //collision checks
        var isGrounded = false;
        var leftWall = false;
        var rightWall = false;
        var touchingWall = false;
        var headUp = false;

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
        var gravityAcceleration = 5;
        var jumpForce = 70;

        var maxSlideForce = 450;
        var slideAcceleration = 30;

        var wallJumpVert = 40;
        var wallJumpHoriz = 60;

        //camera controls
        var leftScreen = 0; //camera controls are taken from the top left corner
        var rightScreen = 1024; //the rightmost part of the screen
        var screenWidth = 1024; //this is the amount the camera knows to increase horizontal screen size by
        var topScreen = 0;
        var botScreen = 512;
        var screenHeight = 512;

        var game = new Phaser.Game(config);

        function preload() {
            this.load.image('background', '/background.png');
            this.load.image('ground', '/ground.png');
            this.load.image('player', '/player.png');
            this.load.image('wall', '/wall.png');
            this.load.spritesheet('healthBar', '/healthBar.png', { frameWidth: 34, frameHeight: 130 });
            this.load.image('spike', '/spike.png');
            this.load.image('wallSpike', 'wallSpike.png');
            this.load.image('warning', '/warning.png');
            this.load.image('block', '/block.png');
        }

        function create() {

            function hit(target, damage, ignoreImmunity) {
                if (ignoreImmunity) {
                    return target - damage;
                }
                else if (iFrames == 0) {
                    return target - damage;
                }
                else {
                    return target;
                }
            }

            var camera = this.cameras.main;

            health = this.add.sprite(30, 80, 'healthBar').setScrollFactor(0).setDepth(1);

            //healthbar
            this.anims.create({
                key: '10HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [0] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '9HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [1] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '8HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [2] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '7HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [3] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '6HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [4] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '5HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [5] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '4HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [6] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '3HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [7] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '2HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [8] }),
                frameRate: -1,
            });
            this.anims.create({
                key: '1HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [9, 9, 10, 11, 12, 13, 13, 13, 13, 12, 11, 11, 10, 10, 9, 9] }),
                frameRate: 14,
                repeat: -1,
            });
            this.anims.create({
                key: '0HP',
                frames: this.anims.generateFrameNumbers('healthBar', { frames: [14, 15, 16, 17, 17, 16, 15, 14, 15, 16, 17, 18, 18, 18, 18, 19, 19, 20, 20, 21, 21, 22, 22, 23, 23, 24, 24, 25, 25, 26, 26, 27] }),
                frameRate: 24,
                paused: true
            });

            //camera
            camera.setZoom(1);

            //Background
            this.add.image(400, 300, 'background');

            //The things we stand on
            ground = this.physics.add.staticGroup();
            wall = this.physics.add.staticGroup();
            spike = this.physics.add.staticGroup();
            deathZone = this.physics.add.staticGroup();
            warning = this.physics.add.staticGroup();
            block = this.physics.add.staticGroup();

            //---------------- l2 g ----------------
            ground.create(-1152, 496, 'ground').setScale(2).refreshBody();

            wall.create(-1264, 400, 'wall').setScale(1).refreshBody();
            ground.create(-1184, 352, 'ground').setScale(1).refreshBody();
            ground.create(-1056, 352, 'ground').setScale(1).refreshBody();

            ground.create(-1088, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1216, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1344, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1472, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1600, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1728, 288, 'ground').setScale(1).refreshBody();
            ground.create(-1856, 288, 'ground').setScale(1).refreshBody();

            wall.create(-2032, 64, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 192, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 320, 'wall').setScale(1).refreshBody();
            wall.create(-1904, 336, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 448, 'wall').setScale(1).refreshBody();
            wall.create(-1904, 464, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 576, 'wall').setScale(1).refreshBody();
            wall.create(-1904, 592, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 704, 'wall').setScale(1).refreshBody();
            wall.create(-1904, 720, 'wall').setScale(1).refreshBody();

            wall.create(-2032, 832, 'wall').setScale(1).refreshBody();
            wall.create(-2032, 960, 'wall').setScale(1).refreshBody();
            //---------------- l2 g ----------------

            //---------------- l2 d1 ----------------
            ground.create(-1920, 1008, 'ground').setScale(2).refreshBody();

            //196 pixel gap here to add a pit of death
            deathZone.create(-1728, 1055, '').setSize(196, 64);

            warning.create(-1760, 1008, 'warning').setScale(1).setDepth(-1);
            warning.create(-1696, 1008, 'warning').setScale(1).setDepth(-1);
            warning.create(-1632, 1008, 'warning').setScale(1).setDepth(-1);
            

            block.create(-1576, 1008, 'block').setScale(1).refreshBody();
            //---------------- l2 d1 ----------------

            //---------------- l1 g ----------------
            wall.create(-1008, 336, 'wall').setScale(1).refreshBody();
            wall.create(-880, 272, 'wall').setScale(2).refreshBody();

            ground.create(-896, 496, 'ground').setScale(2).refreshBody();
            ground.create(-640, 496, 'ground').setScale(2).refreshBody();
            ground.create(-384, 496, 'ground').setScale(2).refreshBody();
            ground.create(-128, 496, 'ground').setScale(2).refreshBody();
            //---------------- l1 g ----------------

            //---------------- c g ----------------
            ground.create(128, 496, 'ground').setScale(2).refreshBody();
            ground.create(384, 496, 'ground').setScale(2).refreshBody();
            ground.create(640, 496, 'ground').setScale(2).refreshBody();
            ground.create(896, 496, 'ground').setScale(2).refreshBody();
            //---------------- c g ----------------

            //---------------- r1 g ----------------
            ground.create(1152, 496, 'ground').setScale(2).refreshBody();
            ground.create(1408, 496, 'ground').setScale(2).refreshBody();

            spike.create(1532, 448, 'wallSpike').setScale(1).refreshBody();

            ground.create(1664, 496, 'ground').setScale(2).refreshBody();
            ground.create(1536, 496, 'ground').setScale(2).refreshBody(); //this is here to prevent a physics bug where the player falls through the floor between gaps in the ground
            ground.create(1664, 464, 'ground').setScale(2).refreshBody(); //up a floor

            spike.create(1788, 416, 'wallSpike').setScale(1).refreshBody();

            ground.create(1920, 496, 'ground').setScale(2).refreshBody();
            ground.create(1792, 464, 'ground').setScale(2).refreshBody(); //this is here to prevent a physics bug where the player falls through the floor between gaps in the ground
            ground.create(1920, 432, 'ground').setScale(2).refreshBody(); //up another
            //---------------- r1 g ----------------

            //---------------- r2 g ----------------
            ground.create(2176, 496, 'ground').setScale(2).refreshBody(); //stay on the upper floor
            ground.create(2176, 432, 'ground').setScale(2).refreshBody();

            ground.create(2432, 496, 'ground').setScale(2).refreshBody();
            ground.create(2432, 432, 'ground').setScale(2).refreshBody();

            ground.create(2688, 496, 'ground').setScale(2).refreshBody();
            ground.create(2688, 432, 'ground').setScale(2).refreshBody();

            ground.create(2944, 496, 'ground').setScale(2).refreshBody();
            ground.create(2944, 432, 'ground').setScale(2).refreshBody();
            //---------------- r2 g ----------------

            //---------------- r2 d1 ----------------
            ground.create(2176, 1008, 'ground').setScale(2).refreshBody(); //here, d1 refers to going down 1
            ground.create(2432, 1008, 'ground').setScale(2).refreshBody();
            ground.create(2688, 1008, 'ground').setScale(2).refreshBody();
            ground.create(2944, 1008, 'ground').setScale(2).refreshBody();
            //---------------- r2 d1 ----------------

            //---------------- r3 d1 ----------------
            ground.create(3200, 1008, 'ground').setScale(2).refreshBody();
            ground.create(3456, 1008, 'ground').setScale(2).refreshBody();
            ground.create(3712, 1008, 'ground').setScale(2).refreshBody();
            ground.create(3968, 1008, 'ground').setScale(2).refreshBody();
            //---------------- r3 d1 ----------------

            //Player settings
            player = this.physics.add.image(-200, 170, 'player').setScale(0.5);
            player.setCollideWorldBounds(false);

            //Player will obey the laws of physics, and not phase through solid ground
            
            this.physics.add.collider(player, ground);
            this.physics.add.collider(player, wall);

            this.physics.add.overlap(
                player, deathZone,
                function (_player, _deathZone) {
                    playerHealth = hit(playerHealth, 10, true);
                })

            this.physics.add.collider(
                player, spike,
                function (_player, _spike) {
                    playerHealth = hit(playerHealth, 2, false);
            })
                

            this.physics.add.collider(player, block);

            movementKeys = this.input.keyboard.addKeys({
                left: 'left',
                right: 'right',
                down: 'down',
                jump: 'Z'
            })

            //death screen. the player is teleported between this text when they die to respawn
            this.add.text(362, -4948, 'YOU DIED', { font: '64px Garamond', fill: '#ffffff' });
            this.add.text(438, -4838, 'Hold Down to respawn', { font: '16px Garamond', fill: '#ffffff' });
            debug = this.add.text(200, 100, '').setScrollFactor(0);

        }

        function update() {

            debug.setText([
                'player health: ' + playerHealth,
                'iFramers: ' + iFrames,
                'last health: ' + lastKnownHealth
            ]);

            if (iFrames > 0) {
                iFrames += -1;
            }

            if (playerHealth != lastKnownHealth) {
                iFrames = 30;
                playerHealth = playerHealth;
                lastKnownHealth = playerHealth;
            }

            if (playerHealth > 10) {
                playerHealth = 10;
            }
            if (playerHealth <= 0) {
                playerHealth = 0;
            }

            if (playerHealth == 10) {
                health.anims.play('10HP', true);
            }
            if (playerHealth == 9) {
                health.anims.play('9HP', true);
            }
            if (playerHealth == 8) {
                health.anims.play('8HP', true);
            }
            if (playerHealth == 7) {
                health.anims.play('7HP', true);
            }
            if (playerHealth == 6) {
                health.anims.play('6HP', true);
            }
            if (playerHealth == 5) {
                health.anims.play('5HP', true);
            }
            if (playerHealth == 4) {
                health.anims.play('4HP', true);
            }
            if (playerHealth == 3) {
                health.anims.play('3HP', true);
            }
            if (playerHealth == 2) {
                health.anims.play('2HP', true);
            }
            if (playerHealth == 1) {
                health.anims.play('1HP', true);
            }
            if (playerHealth <= 0 && !playerDead) {
                health.anims.play('0HP', true);
                playerDead = true;
            }

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
            if (player.y < topScreen) {
                topScreen += -screenHeight;
                botScreen += -screenHeight;
                camera.scrollY = topScreen;
            }
            if (player.y > botScreen) {
                camera.scrollY = botScreen;
                topScreen += screenHeight;
                botScreen += screenHeight;
            }

            //makes checks for the player touching what can be considered ground
            if (player.body.touching.down) {
                isGrounded = true;
            }
            else if (!player.body.touching.down) {
                isGrounded = false;
            }

            //this will check for any surface the player touches with their sides, and consider it a wall
            if (!isGrounded && player.body.touching.left || !isGrounded && player.body.touching.right) {
                touchingWall = true;
                if (player.body.touching.left) {
                    leftWall = true;
                }
                if (player.body.touching.right) {
                    rightWall = true;
                }
            }
            if (!player.body.touching.left && !player.body.touching.right || isGrounded) {
                touchingWall = false;
                leftWall = false;
                rightWall = false;
            }

            if (player.body.touching.up) {
                headUp = true;
            }
            else if (!player.body.touching.up) {
                headUp = false;
            }

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

            if (!playerDead) {
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

                if (touchingWall && isJumping) {
                    isJumping = false;
                    hasJumped = true;
                    drivingVerticalForce = 0;
                    finalVerticalVel = 0;
                }

                if (isJumping && headUp) { //this and the check directly below it are there to stop the player from sliding along the underside of terrain when jumping into them
                    isJumping = false;
                    hasJumped = true;
                    drivingVerticalForce = 0;
                    finalVerticalVel = 0;
                }
                if (hasJumped && headUp) {
                    drivingVerticalForce = 0;
                    finalVerticalVel = 0;
                }

                if (wallJump && movementKeys.jump.isUp) {
                    drivingVerticalForce = 0;
                    drivingHorizontalForce = 0;
                    finalHorizontalVel = 0;
                    finalVerticalVel = 0;
                    if (leftWall) {
                        touchingWall = false;
                        drivingHorizontalForce = wallJumpHoriz;
                        drivingVerticalForce = -wallJumpVert;
                    }
                    if (rightWall) {
                        touchingWall = false;
                        drivingHorizontalForce = -wallJumpHoriz;
                        drivingVerticalForce = -wallJumpVert;
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
                    if (drivingVerticalForce < 0) {
                        drivingVerticalForce = 0;
                    }
                    if (finalVerticalVel < 0) {
                        finalVerticalVel = 0;
                    }
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
                        wallJump = true;
                        player.setVelocityY(0);
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
            }

            if (playerDead) {
                player.setVelocityX(0);
                player.setVelocityY(0);
                drivingHorizontalForce = 0;
                finalVerticalVel = 0;
                drivingVerticalForce = 0;
                finalVerticalVel = 0;
                player.x = 512;
                player.y = -4864;

                if (movementKeys.down.isDown && playerHealth < 10) {
                    if (respawnDelay == 0) {
                        playerHealth += 1;
                        respawnDelay = 10;
                    } 
                    if (respawnDelay != 0) {
                        respawnDelay += -1;
                    }
                }
                if (movementKeys.down.isUp && playerHealth != 10) {
                    if (respawnDelay == 0) {
                        playerHealth += -1;
                        respawnDelay = 5;
                    }
                    if (respawnDelay != 0) {
                        respawnDelay += -1;
                    }
                }
                if (playerHealth == 10) {
                    playerDead = false;
                    player.y = -256;
                }
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

The biggest challenge was sorting out immunity frames so that they would only trigger on certain events, and also assuring they only triggered if they hadn't been triggered recently.&#x20;

## Testing

### Tests

| Test | Instructions                                               | What I expect                                                                                      | What actually happens | Pass/Fail |
| ---- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run the code                                               | Player should be created, fall onto a platform. Health bar should be full                          | As expected           | Pass      |
| 2    | move onto some spikes and then move away                   | damage should be dealt, spikes should behave like whatever terrain they're on.                     | As expected           | Pass      |
| 3    | Move onto spikes and stay                                  | Damage should only be dealt once, and not be dealt again for a delay                               | As expected           | Pass      |
| 4    | Let health reach 0                                         | Player should be teleported to the death screen                                                    | As expected           | Pass      |
| 5    | Hold the respawn button on the death screen and release it | Health bar should fill gradually till released, at which point the bar should rapidly drain again  | As expected           | Pass      |
| 6    | Hold the respawn button, don't release                     | The player should drop down one screen onto the game world again and the health bar should be full | As expected           | Pass      |
| 7    | Find the death pit, fall into it                           | The player should be moved the the death screen immediately                                        | As expected           | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
