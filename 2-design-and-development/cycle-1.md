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

| Test | Instructions  | What I expect     | What actually happens | Pass/Fail |
| ---- | ------------- | ----------------- | --------------------- | --------- |
| 1    | Run code      | Thing happens     | As expected           | Pass      |
| 2    | Press buttons | Something happens | As expected           | Pass      |

### Evidence
