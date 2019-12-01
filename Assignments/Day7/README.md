# Lucky21 Game API

## Part 1 - Lucky 21

For the remainder of the course we will be working creating a simple game called Lucky 21.

### Rules

The player start the game with 2 cards, each card has an associate point value:

- The Jack, Queen and King are worth 10 points
- An Ace is worth 1 or 11 points depending on which is most beneficial to the player

The player can choose between two actions "21 or under" or "over 21":

"21 or under" -> The player draws a new card if his total point value:

* Is under 21, the game continues
* Is 21, the player wins the game
* Is over 21, the player loses the game

"over 21" -> The player draws the final card if his total point value:

* Is under 21, the player loses the game
* Is 21, the player loses the game
* Is over 21, the player wins the game

## Part 2 - Game Logic

We will now take the item_repository that we created in week 1 and change it into the
API for our game.\
Change the name of the project (directory etc) from item_repository to game_api.\
Now create the following files:\
`game_api/deck.js`:
```javascript
module.exports = () => {
    return [
      '01H', '02H', '03H', '04H', '05H', '06H', '07H', '08H', '09H', '10H', '11H', '12H', '13H', // Hearts
      '01C', '02C', '03C', '04C', '05C', '06C', '07C', '08C', '09C', '10C', '11C', '12C', '13C', // Clubs
      '01D', '02D', '03D', '04D', '05D', '06D', '07D', '08D', '09D', '10D', '11D', '12D', '13D', // Diamonds
      '01S', '02S', '03S', '04S', '05S', '06S', '07S', '08S', '09S', '10S', '11S', '12S', '13S', // Spades
    ];
};
```

`game_api/dealer.js`:
```javascript
module.exports = () => {
    return {
        shuffle: (deck) => {
            for (let i = 0; i < deck.length - 1; i++) {
                const j = Math.floor(Math.random() * (deck.length - i)) + i;
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
```

`game_api/lucky21.js`:
```javascript
const deckConstructor = require('./deck.js');
const dealerConstructor = require('./dealer.js');

module.exports = () => {
    let deck = deckConstructor();
    let dealer = dealerConstructor();
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
        // Is the game finished.
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
        // The cards value + the card value if it exits (integer).
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
    };
};
```

Do not implement the `TODO`s for now

### Game State
**Cards**: An array of string representing the cards the player has guessed are 21 or under.\
**Card**: The card that the player guessed should go over 21.

#### Example 1
![Example 1](https://is5-ssl.mzstatic.com/image/thumb/Purple111/v4/ad/f2/17/adf217d4-cbb7-879a-a920-4f8d9a259a4b/pr_source.png/643x0w.jpg)
```javascript
{
    "deck": "...",
    "dealer": "...",
    "cards": [
        "13D", "11D",
    ],
    "card": undefined
}
```
getCardsValue => 20\
getCardValue => undefined\
getTotal => 20

#### Example 2
![Example 2](https://is1-ssl.mzstatic.com/image/thumb/Purple111/v4/38/f6/4f/38f64f6f-8a77-51de-a775-ce3209cd0ce5/pr_source.png/643x0w.jpg)
```javascript
{
    "deck": "...",
    "dealer": "...",
    "cards": [
        "02S", "09S", "09C",
    ],
    "card": "11S"
}
```
getCardsValue => 20\
getCardValue => 10\
getTotal => 30

#### Example 3
![Example 3](https://is2-ssl.mzstatic.com/image/thumb/Purple122/v4/ea/c4/c2/eac4c234-0eff-0bcb-59b3-286f0a14aa91/pr_source.png/643x0w.jpg)
```javascript
{
    "deck": "...",
    "dealer": "...",
    "cards": [
        "03D", "07H", "06D", "05S",
    ],
    "card": undefined
}
```
getCardsValue => 21\
getCardValue => undefined\
getTotal => 21

## Part 3 - Unit Test Setup

Before we start implementing our game's logic we will create unit tests to validate whether
we are done or not, for this we will use [Jest](https://jestjs.io/).\
Which is a javascript testing framework, that helps us create and run our tests.\
Use `npm` to install the [Jest](https://www.npmjs.com/package/jest) package as a **dev** dependency.

Now lets create a simple test for our game logic inside \
`lucky21.unit-test.js`:
```javascript
const lucky21Constructor = require('./lucky21.js');

test('a new game should have 50 cards left in the deck', () => {
  let game = lucky21Constructor();
  expect(game.state.deck.length).toEqual(50);
});

test('a new game should have 2 drawn cards', () => {
  let game = lucky21Constructor();
  expect(game.state.cards.length).toEqual(2);
});
```

Then create another script inside `package.json` that runs Jest on all files ending in `.unit-test.js`
(HINT: `jest --testRegex [YOUR_REGEX] --collectCoverage true`).
```bash
npm run test:unit
```

## Part 4 - Dependency Injection

Now we want to create tests where we play an entire game. But we have a problem, the dealer's
shuffle function uses `Math.random` to shuffle the deck, so it is impossible to know what the
player will draw in the test.

To work around this we will use dependency injection, which means the caller will provide
the callee with the dependencies in our case the constructor:\
`game-api/lucky21.js`:
```javascript
module.exports = (deck, dealer) => {
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
    };
};
```

Notice how the dependencies are sent in as a parameter instead of imported at the top 
of the file. This gives the test (the caller) more control over game's dependencies 
making it easier to test focused parts of the system.

Update the previous tests to use dependency injection.

And lets add another test that tests more of the game logic.

`lucky21.unit-test.js`:
```javascript
const deckConstructor = require('./deck.js');
const dealerConstructor = require('./dealer.js');
const lucky21Constructor = require('./lucky21.js');

test('guess21OrUnder should draw the next card', () => {
  // Arrange
  let deck = deckConstructor();
  deck = [
      '05C', '01D', '09S', '10H', 
  ];
  let dealer = dealerConstructor();
  // Override the shuffle to do nothing.
  dealer.shuffle = (deck) => {};
  
  // Inject our dependencies
  let game = lucky21Constructor(deck, dealer);
  
  // Act
  game.guess21OrUnder(game);
  
  // Assert
  expect(game.state.cards.length).toEqual(3);
  expect(game.state.cards[2]).toEqual('01D');
});
```

## Part 5 - Writing Tests

Now write more unit tests for the lucky21 game as we did above, there should be at least
1 test per function and at least 20 unit tests in total.

Run your tests and make sure they are failing:
```bash
npm run test:unit
```

## Part 6 - Implementation

Implement the `TODO`s in the game. Once you are done, all your unit tests should succeed.

## Handin

You should store all the source files in your repository:

```bash
├── game_api
│   ├── app.js
│   ├── deck.js
│   ├── dealer.js
│   ├── lucky21.js
│   ├── lucky21.unit-test.js
│   ├── database.js
│   ├── Dockerfile
│   └── package.json
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   ├── initialize_game_api_instance.sh
│   ├── verify_environment.sh
│   ├── docker_compose_up.sh
│   ├── docker_build.sh
│   ├── docker_push.sh
│   └── deploy.sh
├── docker-compose.yml
├── infrastructure.tf
├── Jenkinsfile
└── README.md
```

They must be placed at these location to get full marks.
