# Copy of 2.2.15 Cycle 14 - Level Showcase and final comments

## Design

### Objectives

* [x] Add comments to everything
* [x] Showcase the level

### Pseudocode

```
variable = value //explain why this is needed and what it does
```

## Development

### Outcome

This cycle didn't aim to achieve anything development-wise. The point of this section was to simply showcase the final design and add more comments to the code for clarity.

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
        var movingSaw; //due to the way phaser handles it's own physics objects, a separate variable is needed for the moving saw
        var tutorialText = true; //a varaible which checks if the tutorial help text needs to be visible or hidden
        var isBad = false; //checks if the tutorial text needs to be constantly shown or not

        //health stuff
        //this checks the players
        var playerHealth = 10; //the player's current health
        var lastKnownHealth = 10; //checks what the player's health was on the previous frame
        var playerDead = false; //a way to see if the player is dead or not
        var respawnDelay = 10; //the amount of time the player needs to hold for the the player to respawn
        var iFrames = 0; //immunity frames: this exists to prevent the player getting hit too much too quickly

        //these variables exist to refer to objects to make it easier to reference them
        var player; 
        var platforms;
        var movementKeys;

        //collision checks
        var isGrounded = false; //checks for if the player is touching the ground
        var leftWall = false; //wall being touched is too the left
        var rightWall = false; //wall being touched it to the right
        var touchingWall = false; //general check for if the player is touching a wall
        var headUp = false; //checks if the player's head is touching a ceiling

        //values for acceleration
        var finalHorizontalVel = 0; //this is calculated every frame. it determines the amount the player is moved by horizontaly
        var playerMass = 2; //arbitrary value for the mass of the player. this will matter if a force-based combat system gets added in future versions
        var drivingHorizontalForce = 0; //the force being applied to the player at the start of every frame
        var resHorizontalForce = 0; //the force resisting the applied force
        var turningLeft = false; //checks if the player has met the conditions to turn. this states the player is turning left.
        var turningRight = false; //safe as for a left turn, but for the right turn

        //vertical movement
        var finalVerticalVel = 0; //the final vertical velocity. this works the exact same way as the horizontal velocity, just in the vertical direction
        var drivingVerticalForce = 0; //the force being applied to the player at the start of each frame. this is what replaces the normal phaser gravity.
        var resVerticalForce = 0; //this is air resistance. it's essentally negligable after a certain point but adds some slight drag to initial acceration, and the value is needed to calculate velocity
        var hasJumped = false; //this is to check if the player has finished a jump, and has not fulfilled a criteria to reset the jump. it prevents the player from jumping again before they're meant to
        var isJumping = false; //a check for the player's current jump. some of the factors such as resisting vertical force and driving vertical force are affected by it, so it is important to keep track of.

        var canJump = false; //allows/disallows the player to jump based off of some preset conditions

        var canRun = false; //checks if the player is allowed to run

        var wallSlide = false; //this variable checks if the player is currently sliding down a wall
        var gravityDragMultiplier = 0.0001; //this is used to change resVerticalForce to a smaller/greater value based off of what the player is touching
        var wallJump = false; //checks if the player is in the state for charging up a wall jump

        //special movement
        var canSlide = false; //is true when the player reaches the slide threshold, allows the player to slide
        var sliding = false; //checks for if the player is in the sliding movement
        var hasSlid = false; //this varbiable checks if a slide is completed. this likely wasn't necessary as using the sliding check could have served similar purposes
        var slideDeceleration = Math.abs(drivingHorizontalForce) / 5; //the slide deceleration aims to be sudden and sharp at the start but very quickly drop off. this is because the slide works by adding a huge force to the player

        //movement values
        var slidingThreshold = 300; //the minimum velocity required for the player to begin a slide movement

        var maxRunningForce = 100; //the maximum force the player can run with
        var runningAcceleration = 10; //the amount the force increases by when under the max
        var runningDeceleration = 10; //the amount the force decreases by when not running
        var horizontalDragMultiplier = 0.65; //affects the value of resHorizontalForce based on what the player is touching

        var maxTurningForce = 200; //the most force that can be applied during the turn movement
        var turningThreshold = 90; //the velocity threshold for turning. 90 allows for a frame's worth of delay to the player's input
        var turningAcceleration = 20; //the amount of force applied every frame during a turn up to a threshold
        var turningDeceleration = 10; //the amount of force lost after a turn is complete, down till normal running

        var maxGravity = 10; //the max acceleration due to gravity
        var gravityAcceleration = 5; //the amount the acceleration increases by every frame too a max
        var jumpForce = 70; //the force the player jumps with

        var maxSlideForce = 450; //the maximum force that can be applied during a slide
        var slideAcceleration = 30; //the force that's applied to the player every frame during a slide movement up to a threshold

        var wallJumpVert = 40; //the vertical force of the wall jump
        var wallJumpHoriz = 60; //the horizontal force of the wall jump

        //camera controls. these are taken from the top left corner
        var leftScreen = 0; //the leftmost part of the screen
        var rightScreen = 1024; //the rightmost part of the screen
        var screenWidth = 1024; //this is the amount the camera knows to increase horizontal screen size by
        var topScreen = 0; //the topmost part of the screen
        var botScreen = 512; //the bottommost part of the screen
        var screenHeight = 512; //this is the amount the camera knows to increase vertical screen size by

        function hit(target, damage, ignoreImmunity) { //this function is called to do damage to a target entity whenever it's called, with an option to ignore immunity frames if they were active
            if (ignoreImmunity) { //if the damage ignores immunity frames, this just does the damage
                return target - damage;
            }
            else if (iFrames == 0) { //otherwise, this function only does damage to the target if the target has no immunity frames
                return target - damage; //damage is done by removing the value of the damage from the target's health
            }
            else {
                return target; //if nothing occurs, the function only returns the target's starting health when the function was called
            }
        }

        var game = new Phaser.Game(config);

        function preload() { //this is needed to load all the assets phaser may need for the game
            this.load.image('background', '/background.png'); //the background image
            this.load.image('ground', '/ground.png'); //the ground
            this.load.image('player', '/player.png'); //player
            this.load.image('wall', '/wall.png'); //the ground but verticle
            this.load.spritesheet('healthBar', '/healthBar.png', { frameWidth: 34, frameHeight: 130 }); //this is a spritesheet for the healthbar. each health state is a frame
            this.load.image('spike', '/spike.png'); //a set of spikes
            this.load.image('wallSpike', 'wallSpike.png'); //a set of spikes but vertically
            this.load.image('warning', '/warning.png'); //an omnious glow
            this.load.image('block', '/block.png'); //a small block ground piece
            this.load.image('saw', '/saw.png'); //the saw
        }

        function create() {

            var camera = this.cameras.main; //this is just used to make the main camera easier to call

            health = this.add.sprite(30, 80, 'healthBar').setScrollFactor(0).setDepth(1); //this creates a healthbar near the top of the screen. the setScrollFactor 0 means it will stay on that part of the screen no matter where the camera moves
            
            //healthbar animations
            this.anims.create({ //every animation here is a health state.
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
                frames: this.anims.generateFrameNumbers('healthBar', {
                    frames: [
                        14, 14, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 27, 27, 27,
                        15, 15, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 27, 27, 27,
                        16, 16, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 27,
                        17, 17, 18, 18, 18, 19, 19, 19, 20, 20, 20, 21, 21, 21, 22, 22, 22, 23, 23, 23, 24, 24, 24, 25, 25, 26, 26, 27]
                }),
                frameRate: 24,
                paused: true
            });

            //the scale of the camera resolution.
            camera.setZoom(1);

            //Background
            this.add.image(400, 300, 'background'); //generates the background image, and sets it to the centre of the screen

            //The things we stand on and get hit by
            ground = this.physics.add.staticGroup(); //this just defines physics for all the objects the player can interact with
            wall = this.physics.add.staticGroup();
            spike = this.physics.add.staticGroup();
            deathZone = this.physics.add.staticGroup();
            warning = this.physics.add.staticGroup();
            block = this.physics.add.staticGroup();
            saw = this.physics.add.staticGroup();


            /*there was a way to add levels with spritesheets, however
             that would require me to make a tileset and ensure it was applicable to all rotations and overlapping images didn't cause 
             clipping, so I decided this would be less effort. I was wrong*/
            //---------------- l2 g ---------------- this stands for left 2 screens, on the ground level
            ground.create(-1152, 496, 'ground').setScale(2).refreshBody(); 

            wall.create(-1264, 400, 'wall').setScale(1).refreshBody();
            ground.create(-1136, 352, 'ground').setScale(1.75, 1).refreshBody();

            ground.create(-1472, 288, 'ground').setScale(7, 1).refreshBody();
            
            wall.create(-2032, 480, 'wall').setScale(1, 8).refreshBody();
            spike.create(-2012, 492, 'wallSpike').setScale(1).refreshBody().setFlipX(true);
            spike.create(-2012, 518, 'wallSpike').setScale(1).refreshBody().setFlipX(true);
            spike.create(-2012, 544, 'wallSpike').setScale(1).refreshBody().setFlipX(true);

            wall.create(-1904, 544, 'wall').setScale(1, 3.75).refreshBody();
            spike.create(-1924, 716, 'wallSpike').setScale(1).refreshBody();
            spike.create(-1924, 742, 'wallSpike').setScale(1).refreshBody();
            spike.create(-1924, 768, 'wallSpike').setScale(1).refreshBody();
            //---------------- l2 g ----------------

            //---------------- l2 d1 ----------------
            ground.create(-1920, 1008, 'ground').setScale(2).refreshBody();

            saw.create(-1968, 976, 'saw').setCircle(22).setScale(1.5).setDepth(-1).refreshBody();

            //pit of death
            deathZone.create(-1696, 1055, '').setSize(192, 64); //this is a zone which if the player enters, they die immediately

            warning.create(-1696, 1008, 'warning').setScale(3, 1).setDepth(-1);
            
            block.create(-1568, 1008, 'block').setScale(1).refreshBody();
            spike.create(-1588, 892, 'wallSpike').setScale(1).refreshBody();
            wall.create(-1568, 616, 'wall').setScale(1, 4.875).refreshBody();
            spike.create(-1548, 892, 'wallSpike').setScale(1).refreshBody().setFlipX(true);

            //pit of death 2
            deathZone.create(-1440, 1055, '').setSize(192, 64);

            warning.create(-1440, 1008, 'warning').setScale(3, 1).setDepth(-1);

            block.create(-1312, 1008, 'block').setScale(1).refreshBody();
            spike.create(-1332, 892, 'wallSpike').setScale(1).refreshBody();
            wall.create(-1312, 616, 'wall').setScale(1, 4.875).refreshBody();
            spike.create(-1292, 892, 'wallSpike').setScale(1).refreshBody().setFlipX(true);

            //pit of death 3
            deathZone.create(-1184, 1055, '').setSize(192, 64);

            warning.create(-1184, 1008, 'warning').setScale(3, 1).setDepth(-1);

            block.create(-1056, 1008, 'block').setScale(1).refreshBody();
            spike.create(-1076, 892, 'wallSpike').setScale(1).refreshBody();
            wall.create(-1056, 728, 'wall').setScale(1, 3.125).refreshBody();
            spike.create(-1036, 892, 'wallSpike').setScale(1).refreshBody().setFlipX(true);
            //---------------- l2 d1 ----------------

            //---------------- l1 g ----------------
            wall.create(-1008, 336, 'wall').setScale(1).refreshBody();
            wall.create(-880, 144, 'wall').setScale(2, 4).refreshBody();

            movingSaw = this.physics.add.image(-360, 465, 'saw').setCircle(22).setScale(2).setDepth(-1).setPushable(false).refreshBody();

            this.tweens.add({
                targets: movingSaw,
                x: -664,
                y: 465,
                ease: 'Sine.easeInOut',
                duration: 2000,
                yoyo: true,
                repeat: -1
            });

            wall.create(-512, 192, 'wall').setScale(1, 3).refreshBody();

            ground.create(-512, 496, 'ground').setScale(8, 2).refreshBody();
            //---------------- l1 g ----------------

            //---------------- c g ----------------
            ground.create(512, 496, 'ground').setScale(8, 2).refreshBody(); //this creates a ground object. it's created at the centre of the screen, and has a scale of 8 widthwise, and 2 heightwise
            //---------------- c g ----------------

            //---------------- r1 g ----------------
            ground.create(1536, 496, 'ground').setScale(8, 2).refreshBody();
            block.create(1280, 432, 'block').setScale(0.5, 1).refreshBody();
            spike.create(1280, 396, 'spike').setScale(1).refreshBody();

            ground.create(1792, 448, 'ground').setScale(4, 1).refreshBody(); //up a floor
            ground.create(1920, 416, 'ground').setScale(2, 1).refreshBody(); //up another

            spike.create(1532, 448, 'wallSpike').setScale(1).refreshBody(); //spike for the first up
            saw.create(1664, 435, 'saw').setCircle(22).setScale(1).setDepth(-1).refreshBody();
            wall.create(1664, 165, 'wall').setScale(1, 3).refreshBody();

            spike.create(1788, 416, 'wallSpike').setScale(1).refreshBody(); //spikes for the second up
            saw.create(1920, 403, 'saw').setCircle(22).setScale(1).setDepth(-1).refreshBody();
            wall.create(1920, 133, 'wall').setScale(1, 3).refreshBody();
            //---------------- r1 g ----------------

            //---------------- r2 g ----------------
            ground.create(2560, 464, 'ground').setScale(8, 4).refreshBody();
            //---------------- r2 g ----------------

            //---------------- r2 d1 ----------------
            ground.create(2560, 1008, 'ground').setScale(8, 2).refreshBody(); //here, d1 refers to going down 1
            //---------------- r2 d1 ----------------

            //---------------- r3 d1 ----------------
            ground.create(3584, 1008, 'ground').setScale(8, 2).refreshBody();
            //---------------- r3 d1 ----------------

            //Player settings
            player = this.physics.add.image(512, 200, 'player').setScale(0.5); //this is the player. the initial position is just the spawn position, and the scale is half the image size.
            player.setCollideWorldBounds(false); //this just prevents the player from colliding with the edge of the screen

            //Player will obey the laws of physics, and not phase through solid ground
            
            this.physics.add.collider(player, ground); //this means the player will be unable to pass through any ground objects
            this.physics.add.collider(player, wall); //this means the player will be unable to pass through any wall objects

            this.physics.add.overlap( //the overlap is like a collider but doesn't stop the player from moving through the area.
                player, deathZone,
                function (_player, _deathZone) {
                    playerHealth = hit(playerHealth, 10, true);
                });

            this.physics.add.collider( //damage was done through running a function whenever the collider detected the player and the object touched
                player, spike,
                function (_player, _spike) {
                    playerHealth = hit(playerHealth, 2, false); //this is the damage function from earlier
                    wallSlide = false; //and this is to ensure that the player can't climb on the spikes
                });

            this.physics.add.collider(
                player, saw,
                function (_player, _saw) {
                    playerHealth = hit(playerHealth, 1, false);
                    if (iFrames > 0) {
                        iFrames += -1; //the saw removes immunity frames every frame the player touches it, so it does more damage over time
                    }
                });

            this.physics.add.collider(
                player, movingSaw,
                function (_player, _movingSaw) { //this was only needed because the moving saw was separate from the regular saw
                    playerHealth = hit(playerHealth, 1, false);
                    if (iFrames > 0) {
                        iFrames += -1;
                    }
                });

            this.physics.add.collider(player, block);

            movementKeys = this.input.keyboard.addKeys({ //this is an object which stores the keys used by the game. each action is given a name which links to a key
                left: 'left',
                right: 'right',
                down: 'down',
                jump: 'Z',
                help: 'Q'
            })

            //death screen. the player is teleported between this text when they die to respawn
            this.add.text(362, -4948, 'YOU DIED', { font: '64px Garamond', fill: '#ffffff' }); //this creates text which tells the player they died
            this.add.text(438, -4838, 'Hold Down to respawn', { font: '16px Garamond', fill: '#ffffff' });
            controlText = this.add.text(55, 16, '', { font: '12px Sans' }).setScrollFactor(0);
        }

        function update() {

            if (movementKeys.help.isDown) {
                controlText.setText([
                    '- Release Q: Hide help text',
                    '- Left/Right Arrows: Accelerate Left/Right',
                    '- Z: Jump. Tap for short jump, hold for long.',
                    '     Tap jump while on a wall to jump again',
                    '- Down: Perform a short dash if running fast enough'
                ]);
                isBad = true;
            }
            else if (!isBad) {
                controlText.setText([
                    '- Hold Q: Show help text'
                ]);
            }
            else {
                controlText.setText([
                    ''
                ]);
            }

            Phaser.Actions.Rotate(saw.getChildren(), -0.1, 0.0005);
            movingSaw.rotation += -0.1;

            if (iFrames > 0) {
                iFrames += -1;
            }

            if (playerHealth != lastKnownHealth) {
                iFrames = 50; //if the player has taken damage, so if their new health is different from the last frame's health, they are given some immunity so they don't die too fast
                playerHealth = playerHealth;
                lastKnownHealth = playerHealth;
            }

            if (playerHealth > 10) {
                playerHealth = 10;
            }
            if (playerHealth <= 0) {
                playerHealth = 0;
            }

            if (playerHealth == 10) { //relates health to a healthbar. using a switch statement here would make more sense.
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

            //this section controls the camera
            if (player.x > rightScreen) { //if the player's position is past the rightmost point of the screen
                camera.scrollX = rightScreen; //the camera scrolls to the next screen right
                leftScreen += screenWidth; //and then updates the screen's boundaries
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
                if (!isGrounded && !wallJump) {
                    runningAcceleration = 2;
                    maxTurningForce = 100;
                }
                else if (isGrounded) {
                    runningAcceleration = 10;
                    maxTurningForce = 200;
                }
                if (movementKeys.left.isDown && !leftWall) {
                    if (drivingHorizontalForce >= turningThreshold && !turningLeft) {
                        /*this checks for whether or not the player can perform a turn boost. 90 allows for input imperfections as it is one frame
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
                else if (movementKeys.right.isDown && !rightWall) {
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
                    runningAcceleration = 0;
                    if (!wallSlide) {
                        player.setVelocityX(0)
                        finalHorizontalVel = 0;
                        drivingHorizontalForce = 0;
                        wallSlide = true;
                    }
                }
                if (!touchingWall) {
                    runningAcceleration = runningAcceleration;
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
                resHorizontalForce = horizontalDragMultiplier * ((finalHorizontalVel) / 2); //resistance here is reliant on the velocity. Velocity is self limiting.
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

As it turns out, my game is slightly more difficult than I expected and so capturing a recording for the final showcase was slightly more frustrating than I would like to admit.

### Showcase

{% embed url="https://youtu.be/fT7WVrH7BT8" %}
