# 2.2.2 Cycle 2

## Design

### Objectives



*

### Usability Features

* Simplified controls - no complications yet, just very basic movement. Easy to understand, should be easy to implement.

### Key Variables

| Variable Name | Use |
| ------------- | --- |
|               |     |

### Pseudocode

```
```

## Development

### Outcome

The player still spawns from the same position and collides with the ground. I have created larger ground for the purpose of testing the movement. The keys were assigned using phaser's inbuilt system, as was the scaling. Collision detection was implemented in this step so that the acceleration was compatible with the system I wanted to use for deceleration and jumping in the future, in hopes of minimising errors and edits in the future. I decided to have components of the speed the player can travel at held as separate values for similar reasons as the collision detection: by implementing it this way, I can ensure deceleration, turning and interactions with speed boosts/dashes/slides (should I choose to implement them) will require less editing, and acceleration will already work with them. The server required no changes, and the html just had a favicon added.

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
