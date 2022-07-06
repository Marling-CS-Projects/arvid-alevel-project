# 2.2.2 Cycle 2

## Design

### Objectives



* [ ] Make a functioning acceleration system
* [ ] Deceleration should come automatically
* [ ] Make it a function so that it can be used for other movement mechanics

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

I split the calculation for velocity into several parts to ensure it was carried out correctly. These were stored as the variables cal'n', with n referring to a number to distinguish the calculations from each other. The&#x20;

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
```
{% endtab %}
{% endtabs %}

### Challenges



## Testing

### Tests

| Test | Instructions | What I expect | What actually happens | Pass/Fail |
| ---- | ------------ | ------------- | --------------------- | --------- |
| 1    | Run the code |               |                       |           |

### Evidence
