# Testing

## Part 1 - Dependency Injection

When we are testing we often do not want to test each dependency that our `unit`
(class) depends on. If we just take `dealer.js shuffle` function for example we 
do not `require` any other javascript files but we are still dependant on 
`Math.random` since it's not deterministic there are not a lot of interesting 
tests we can write.

What we could do is change the shuffle function to take the random function as
a parameter (this is called dependency injection) like we did in day 7:
```javascript
// dealer.js
module.exports = () => {
    return {
        shuffle: (deck, random) => {
            for (let i = 0; i < deck.length - 1; i++) {
                const j = Math.floor(random() * (deck.length - i)) + i;
                const card = deck[j];
                const old = deck[i];
                deck[i] = card;
                deck[j] = old;
            }
        },
        draw: (deck) => {
            const card = deck.pop();
            return card;
        },
    };
};

// dealer.unit-test.js
const dealerConstructor = require('./dealer.js');

test('dealer should shuffle cards', () => {
    // Arrange
    let deck = ['a', 'b', 'c'];
    let dealer = dealerConstructor();

    // Act
    // Expected Shuffle
    // ['a', 'b', 'c'] Initial
    // ['c', 'b', 'a'] Switch index 0 and 2
    // ['c', 'a', 'b'] Switch index 1 and 2
    dealer.shuffle(deck, () => {
        return 0.99;
    });
    
    // Assert
    expect(deck).toEqual(['c', 'a', 'b']);
});
```

But that only moves the dependency into the `lucky21.js` since it will need to provide
the parameter to the dealer. You could do this until you reach the `app.js` file and
make it pass all the dependencies down the chain but that would quickly become
unmanageable for a large code base, since most projects have large chains of dependencies
and each time someone in the team wanted to add a dependency to file it would have to be
carried down from the applications entry point.

Luckily there is a way around this, we will create a `Context` that manages all our
dependencies for us, there are a lot of frameworks that provide this behaviour in all
major programming languages. But to get a better insight into how they work we will
create our own framework.

Now start by renaming `app.js` to `server.js`.

Create a file called `inject.js`:
```javascript
module.exports = function(dependencies) {
    dependencies = dependencies || {};
    let injector = function(dependencyName) {
        if(!dependencies[dependencyName]) {
            throw new Error("Required dependency <" + dependencyName + "> is not provided.")
        }
        return dependencies[dependencyName];
    };
    return injector;
};
```

and another file called `context.js`:
```javascript
const express = require("express");
const database = require("./database.js");
const lucky21 = require("./lucky21.js");
const { Client } = require('pg');
const deck = require('./deck.js');
const dealer = require('./dealer.js');
const server = require('./server.js');
const inject = require('./inject.js');

module.exports = {
    newContext: () => {
        return inject({
            'express': express,
            'pgClient': Client,
            'database': database,
            'lucky21': lucky21,
            'deck': deck,
            'dealer': dealer,
            'server': server,
        });
    },
};
```

and create a new `app.js`:
```javascript
const context = require('./context.js').newContext();
const server = context('server')(context);
server.listen();
```

Now we just have to modify our existing code to be compatible with the our new
framework. Here is a solution to:

`lucky21.js`:
```javascript
module.exports = (context) => {
    let deckConstructor = context('deck');
    let deck = deckConstructor(context);
    
    let dealerConstructor = context('dealer');
    let dealer = dealerConstructor(context);
    
    dealer.shuffle(deck);
    let card0 = dealer.draw(deck);
    let card1 = dealer.draw(deck);
    let state = {
        deck: deck,
        dealer: dealer,
        cards: [
            card0,
            card1,
        ],
        // The card that the player thinks will exceed 21.
        card: undefined,
    };
    return {
        state: state,
        // Is the game over (true or false).
        isGameOver: (game) => {
            // TODO
        },
        // Has the player won (true or false).
        playerWon: (game) => {
            // TODO
        },
        // The highest score the cards can yield without going over 21 (integer).
        getCardsValue: (game) => {
            // TODO
        },
        // The value of the card that should exceed 21 if it exists (integer or undefined).
        getCardValue: (game) => {
            // TODO
        },
        getTotal: (game) => {
            // TODO
        },
        // The player's cards (array of strings).
        getCards: (game) => {
            // TODO
        },
        // The player's card (string or undefined).
        getCard: (game) => {
            // TODO
        },
        // Player action (void).
        guess21OrUnder: (game) => {
            // TODO
        },
        // Player action (void).
        guessOver21: (game) => {
            // TODO
        },
        getState: (game) => {
            return {
                cards: game.state.cards,
                cardsValue: game.getCardsValue(game),
                card: game.state.card,
                cardValue: game.getCardValue(game),
                total: game.getTotal(game),
                gameOver: game.isGameOver(game),
                playerWon: game.playerWon(game),
            };
        },
    };
};
```

Now do this for the other files.

# Part 2 - Adding Dependencies

Now create two new files 
`random.js` 
```javascript
module.exports = function(context) {
    return {
        randomInt(min, max) {
            return Math.floor(Math.random() * (max - min) + min);
        },
    };
};
```

`config.js`
```javascript
module.exports = function(context) {
    return {
        // Postgres
        pgHost: process.env.POSTGRES_HOST || 'default_host',
        pgUser: process.env.POSTGRES_USER || 'default_user',
        pgPassword: process.env.POSTGRES_PASSWORD || 'default_password',
        pgDatabase: process.env.POSTGRES_DB || 'default_database',
        
        // Port
        port: process.env.PORT || 3000,
    };
};
```

Now add these new dependencies to the context and use them in your API.\
You will also need to update `docker-compose.yml` to set the new `PORT`
environment variable.

## Part 3 - Test Doubles

Now that our dependencies are injected, we can start using test doubles
to substitute dependencies in our tests.

Lets create a simple test `dealer.unit-test.js`:
```javascript
function newRandom(randomReturnValues) {
    let i = 0;
    return {
        randomInt: (min, max) => {
            return randomReturnValues[i++];
        }
    };
}

test('dealer should should shuffle cards', () => {
    // Arrange
    let dependencies = {
        'random': () => newRandom([2, 1]),
    };
    let newDealer = require('./dealer.js');
    let dealer = newDealer((name) => {
        return dependencies[name];
    });
    let deck = ['a', 'b', 'c'];

    // Act
    dealer.shuffle(deck);

    // Assert
    expect(deck).toEqual(['c', 'b', 'a']);
});
```

Create at least 2 unit tests for `dealer.js - shuffle`.\
Create at least 2 unit tests for `dealer.js - draw`.\
Create at least 1 unit tests for `deck.js`.\
Create at least 3 unit tests for `random.js`.

## Part 4 - Update the API

Now that we have implementated and tested the game logic it is time to make it available
to our playerbase. Here is an updated version of the server and the database.

`server.js`
```javascript
module.exports = function(context) {
    const express = context("express");
    const databaseConstructor = context("database");
    const database = databaseConstructor(context);
    const configConstructor = context('config');
    const config = configConstructor(context);
    const lucky21Constructor = context("lucky21");

    let app = express();

    app.get('/status', (req, res) => {
        res.statusCode = 200;
        res.send('The API is running!\n');
    });

    let game = undefined;

    // Gets game statistics
    app.get('/stats', (req, res) => {
        database.getTotalNumberOfGames((totalNumberOfGames) => {
            database.getTotalNumberOfWins((totalNumberOfWins) => {
                database.getTotalNumberOf21((totalNumberOf21) => {
                    // Week 3
                    // TODO Explain why we put each consecutive call inside the onSuccess callback of the
                    // previous database call, instead of just placing them next to each other.
                    // E.g.
                    // database.call1(...);
                    // database.call2(...);
                    // database.call3(...);
                    res.statusCode = 200;
                    res.send({
                        totalNumberOfGames: totalNumberOfGames,
                        totalNumberOfWins: totalNumberOfWins,
                        totalNumberOf21: totalNumberOf21,
                    });
                }, (err) => {
                    console.log('Failed to get total number of 21, Error:' + JSON.stringify(err));
                    res.statusCode = 500;
                    res.send();
                });
            }, (err) => {
                console.log('Failed to get total number of wins, Error:' + JSON.stringify(err));
                res.statusCode = 500;
                res.send();
            });
        }, (err) => {
            console.log('Failed to get total number of games, Error:' + JSON.stringify(err));
            res.statusCode = 500;
            res.send();
        });
    });

    // Starts a new game.
    app.post('/start', (req, res) => {
        if (game && game.isGameOver(game) == false) {
            res.statusCode = 409;
            res.send('There is already a game in progress');
        } else {
            game = lucky21Constructor(context);
            const msg = 'Game started';
            res.statusCode = 201;
            res.send(msg);
        }
    });

    // Returns the player's board state.
    app.get('/state', (req, res) => {
        if (game) {
            res.statusCode = 200;
            res.send(game.getState(game));
        } else {
            const msg = 'Game not started'
            res.statusCode = 204;
            res.send(msg);
        }
    });

    // Player makes a guess that the next card will be 21 or under.
    app.post('/guess21OrUnder', (req, res) => {
        if (game) {
            if (game.isGameOver(game)) {
                const msg = 'Game is already over'
                res.statusCode = 403;
                res.send(msg);
            } else {
                game.guess21OrUnder(game);
                if (game.isGameOver(game)) {
                    const won = game.playerWon(game);
                    const score = game.getCardsValue(game);
                    const total = game.getTotal(game);
                    database.insertResult(won, score, total, () => {
                        console.log('Game result inserted to database');
                    }, (err) => {
                        console.log('Failed to insert game result, Error:' + JSON.stringify(err));
                    });
                }
                res.statusCode = 201;
                res.send(game.getState(game));
            }
        } else {
            const msg = 'Game not started'
            res.statusCode = 204;
            res.send(msg);
        }
    });

    // Player makes a guess that the next card will be over 21.
    app.post('/guessOver21', (req, res) => {
        if (game) {
            if (game.isGameOver(game)) {
                const msg = 'Game is already over'
                res.statusCode = 403;
                res.send(msg);
            } else {
                game.guessOver21(game);
                if (game.isGameOver(game)) {
                    const won = game.playerWon(game);
                    const score = game.getCardsValue(game);
                    const total = game.getTotal(game);
                    database.insertResult(won, score, total, () => {
                        console.log('Game result inserted to database');
                    }, (err) => {
                        console.log('Failed to insert game result, Error:' + JSON.stringify(err));
                    });
                }
                res.statusCode = 201;
                res.send(game.getState(game));
            }
        } else {
            const msg = 'Game not started'
            res.statusCode = 204;
            res.send(msg);
        }
    });

    const port = config.port;
    return {
        listen: () => {
            app.listen(port, () => {
                console.log('Game API listening on port ' + port);
            });
        }
    };
}
```

Notice that in addition to getting our dependencies from the context we are also
returning an object with a `listen` function, that we call from our new `app.js`.

And `database.js`
```javascript
module.exports = function(context) {
    const Client = context('pgClient');
    const configConstructor = context('config');
    const config = configConstructor(context);

    function getClient() {
        return new Client({
            host: config.pgHost,
            user: config.pgUser,
            password: config.pgPassword,
            database: config.pgDatabase,
        });
    }

    setTimeout(() => {
      let client = getClient();
      client.connect((err) => {
          if (err) {
              console.log('failed to connect to postgres!');
          } else {
              console.log('successfully connected to postgres!');
              client.query('CREATE TABLE IF NOT EXISTS GameResult (ID SERIAL PRIMARY KEY, Won BOOL NOT NULL, Score INT NOT NULL, Total INT NOT NULL, InsertDate TIMESTAMP NOT NULL);', (err) => {
                  if (err) {
                      console.log('error creating game result table!')
                  } else {
                      console.log('successfully created game result table!')
                  }
                  client.end();
              });
          }
      });
    }, 5000);

    return {
        insertResult: (won, score, total, onSuccess, onError) => {
            let client = getClient();
            client.connect((err) => {
                if (err) {
                    onError(err);
                    client.end();
                } else {
                    const query = {
                        text: 'INSERT INTO GameResult(Won, Score, Total, InsertDate) VALUES($1, $2, $3, CURRENT_TIMESTAMP);',
                        values: [won, score, total],
                    }
                    client.query(query, (err) => {
                        if (err) {
                            onError(err);
                        } else {
                            onSuccess();
                        }
                        client.end();
                    });
                }
            });
            return;
        },
        // Should call onSuccess with integer.
        getTotalNumberOfGames: (onSuccess, onError) => {
            onSuccess(0)
            // TODO week 3
        },
        // Should call onSuccess with integer.
        getTotalNumberOfWins: (onSuccess, onError) => {
            onSuccess(0)
            // TODO week 3
        },
        // Should call onSuccess with integer.
        getTotalNumberOf21: (onSuccess, onError) => {
            onSuccess(0)
            // TODO week 3
        },
    }
}
```

You should now be able to start your API using `docker-compose`, and deploy it automatically
using Jenkins (if you've updated your session credentials in the last 3 hours :neutral_face:).

You should be able to play the game using `curl` on the API.

## Handin

You should store all the source files in your repository:

```bash
├── game_api
│   ├── .eslintrc.json
│   ├── app.js
│   ├── config.js
│   ├── context.js
│   ├── database.js
│   ├── dealer.js
│   ├── dealer.unit-test.js
│   ├── deck.js
│   ├── deck.unit-test.js
│   ├── Dockerfile
│   ├── inject.js
│   ├── lucky21.js
│   ├── lucky21.unit-test.js
│   ├── package.json
│   ├── random.js
│   ├── random.unit-test.js
│   └── server.js
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   ├── deploy.sh
│   ├── docker_build.sh
│   ├── docker_compose_up.sh
│   ├── docker_push.sh
│   ├── initialize_game_api_instance.sh
│   ├── jenkins_deploy.sh
│   ├── setup_local_dev_environment.sh
│   └── sync_session.sh
├── docker-compose.yml
├── infrastructure.tf
├── Jenkinsfile
└── README.md
```

They must be placed at these location to get full marks.
