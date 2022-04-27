# 2.2.1 Cycle 1

## Design

### Objectives

Make a movement script that allows for acceleration and deceleration.

* [x] Allows for acceleration
* [x] Decelerates when you release the movement key

### Usability Features

### Key Variables

| Variable Name | Use                                                            |
| ------------- | -------------------------------------------------------------- |
| isRunning     | Used to start and stop accelerating                            |
| facingRight   | Which way to accelerate                                        |
| maxSpeed      | The maximum speed you will accelerate to                       |
| currentVel    | The current speed and direction you're moving in               |
| decelerator   | The value the player slows down by after releasing the run key |

### Pseudocode

```
procedure player movement
if movement key is pressed
    speed = speed + 10
    if player is facing right
        move right with speed
    else
        move left with speed
when movement key is released
    decelerator = speed / 7
    move speed - decelerator
end procedure
```

## Development

### Outcome

### Challenges

Description of challenges

## Testing

Evidence for testing

### Tests

| Test | Instructions                                                                               | What I expect                                                                                       | What actually happens                                                                                                                      | Pass/Fail |
| ---- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| 1    | Run code                                                                                   | It spawns the player and drops them onto a platform                                                 | As expected                                                                                                                                | Pass      |
| 2    | Press Left/Right                                                                           | Player accelerates in the held direction as long as the key is held.                                | As expected                                                                                                                                | Pass      |
| 3    | Release Left/Right                                                                         | Player decelerates in the direction they were moving.                                               | As expected                                                                                                                                | Pass      |
| 4    | Press in the opposite direction to which you are currently accelerating.                   | Player will slide in the direction they were moving, and then accelerate in the opposite direction  | As expected                                                                                                                                | Pass      |
| 5    | Press in the opposite direction to which you are currently moving and then quickly release | Player will slide and decelerate at a greater rate in the direction they were moving.               | They player suddenly stops instead. If you start moving again after this, you will keep going in the direction you were previously moving. | Fail      |

### Evidence
