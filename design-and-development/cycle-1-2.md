# 2.2.2 Cycle 2

## Design

### Objectives

* [x] Make a functioning acceleration system
* [x] Deceleration should come automatically
* [ ] Velocity should be self limiting

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name | Use                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------- |
| finalVel      | the velocity being calculated and the velocity the player will move at by the end of the frame |
| initialVel    | the velocity at the start of the frame                                                         |
| drivingForce  | the force applied to the player to get them to move                                            |
| resForce      | the frictional force which acts against the driving force                                      |
| playerMass    | a constant of player mass                                                                      |
| cal(n)        | calculation variables                                                                          |
| rightFacing   | determines weather or not the player is facing right                                           |
| isGrounded    | determines if the player is touching ground                                                    |
| canRun        | is the player able to run                                                                      |

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
            drivingForce = negative
            resForce = negative
            finalVel = velocity calculation
            move player finalVel
        elif inputKeys.right is down and canRun = true
            drivingForce = positive
            resForce = positive
            finalVel = velocity calculation
            move player finalVel
        elif finalVel > 0
            drivingForce = 0
            resForce = resForce
            finalVel = velocity calculation
            move player finalVel
        else
            move player 0
            resForce = 0
        end if
    end if
end procedure
```

## Development

### Outcome

I split the calculation for velocity into several parts to ensure it was carried out correctly. These were stored as the variables cal'n', with n referring to a number to distinguish the calculations from each other. The calculation is not yet a function as it meant I could have 2 versions of the same movement system in different directions to test and compare changes. The maths behind it originally used distance as a value, but as the velocity is calculated every frame I can use the initial velocity (velocity at the start of the current frame from the end of the last frame) instead as this is the distance travelled each frame.

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
        var finalVel;
        var initialVel = 10;
        var playerMass = 2;
        var drivingForce;
        var resForce;

        var cal1;
        var cal2;
        var cal3;
        var cal4;
        var cal5;
        var cal6;
        
        var isGrounded = false;
        var canRun = false;
        var rightFacing = true;

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
            text.setText([
                'x pos: ' + player.x, //debug text. tells location of player and their velocity
                'vel: ' + finalVel
            ]);

            if (isGrounded == true) {
                canRun = true;

                if (canRun == true && inputKeys.left.isDown) {
                    drivingForce = 50;
                    resForce = 30;
                    rightFacing = false;

                    cal1 = drivingForce * initialVel;
                    cal2 = resForce * initialVel;
                    cal3 = cal1 - cal2; //calculate overall force
                    cal4 = cal3 / playerMass;
                    cal5 = Math.pow(initialVel, 2); //initial velocity ^2
                    cal6 = cal5 + cal4 + cal4; //initial velocity + 2 force/mass
                    if (cal6 < 0) { finalVel = -Math.sqrt(-cal6); } /* This was useful for vecor quantities but has
                     no effect on the currecnt implementation */
                    else if (cal6 > 0) { finalVel = Math.sqrt(cal6); }
                    else { finalVel = 0; }
                    initialVel = finalVel;

                    player.setVelocityX(-finalVel); /* Vel is meant to be a vector quantity, however
                     due to errors with calculations of negative quantities, this a a temporary workaround to keep
                     values positive. */
                }
                else if (canRun == true && inputKeys.right.isDown) {
                    drivingForce = 50;
                    resForce = 30;
                    rightFacing = true;

                    cal1 = drivingForce * initialVel;
                    cal2 = resForce * initialVel;
                    cal3 = cal1 - cal2;
                    cal4 = cal3 / playerMass;
                    cal5 = Math.pow(initialVel, 2);
                    cal6 = cal5 + cal4 + cal4;
                    if (cal6 < 0) { finalVel = -Math.sqrt(-cal6); }
                    else if (cal6 > 0) { finalVel = Math.sqrt(cal6); }
                    else { finalVel = 0; }
                    initialVel = finalVel;

                    player.setVelocityX(finalVel);
                }

                else if (isGrounded == true && finalVel > 0) {
                    drivingForce = 0;
                    resForce = 30;

                    cal1 = drivingForce * initialVel;
                    cal2 = resForce * initialVel;
                    cal3 = cal1 - cal2;
                    cal4 = cal3 / playerMass;
                    cal5 = Math.pow(initialVel, 2);
                    cal6 = cal5 + cal4 + cal4;
                    if (cal6 < 0) { finalVel = -Math.sqrt(-cal6); }
                    else if (cal6 > 0) { finalVel = Math.sqrt(cal6); }
                    else { finalVel = 0; }
                    initialVel = finalVel;

                    if (rightFacing == true) { player.setVelocityX(finalVel); }
                    else { player.setVelocityX(-finalVel); } //more temporary workarounds for the lack of vectors
                    
                }

                else {
                    player.setVelocityX(0);
                    initialVel = 10;
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

While this is meant to calculate a vector quantity, having both direction and magnitude, I had to use a workaround and always calculate a positive value, because the use of negative values with the Math.sqrt method would return NaN (not a number). The other huge issue I seek to address is the limitless acceleration: the equation I used is meant to limit acceleration as the 2 simulated forces acting on the object equal each other, however I have not implemented a function that changes the frictional force with velocity so the velocity increases to no limit.

The calculation for acceleration also is not a function yet, so for ease of use I will turn it into one as well as add the friction calculation and vector quantities for the velocity.

## Testing

### Tests

| Test | Instructions                         | What I expect                                                                                                        | What actually happens                                                    | Pass/Fail |
| ---- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | --------- |
| 1    | Run the code                         | A white rectangle falls onto some grey platforms on a black background.                                              | As expected                                                              | Pass      |
| 2    | Tap arrow keys                       | The player moves slightly in the direction of the pressed key                                                        | As expected                                                              | Pass      |
| 3    | Hold arrow key and release           | The player should accelerate then decelerate and stop                                                                | As expected                                                              | Pass      |
| 4    | Hold arrow keys and keep holding     | The player should accelerate to a point, and then keep moving at a constant velocity                                 | The player accelerates and keeps accelerating at a constant rate         | Fail      |
| 5    | Hold arrow keys and change direction | Player should accelerate to a point, then decelerate before accelerating again when turning, with a maximum velocity | The player turns with no delay and keeps accelerating at a constant rate | Fail      |
| 6    | Hold arrow keys when in air          | Player should not move                                                                                               | As expected                                                              | Pass      |

### Evidence

![](<../.gitbook/assets/image (6).png>)

The velocity displayed is far too high, and keeps increasing.
