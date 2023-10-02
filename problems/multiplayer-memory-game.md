# Multiplayer memory game

Design a multiplayer game of [memory](https://youtu.be/oFfYmrGeTPs). Rules:
1. put cards on the table, face down, in random order
2. player 1 starts the game by turning any 2 cards. if the cards match, he collects the cards and gets another turn to turn 2 more cards. if they don't match, the cards are flipped over and the player loses turn and player 2's turn begins.
3. Game ends when all cards from the table are turned
4. Player with most cards wins the game

## Requirements

### Functional Requirements

1. Users can select two cards for their turn and play the game according to the rules
2. Users can see the number of cards they have and that of all players in the game at all times
3. Users should play this game with other users who are using the system from a different client.

### Non Functional Requirements

1. Scale to a billion users
2. Support up to 10 players
3. Support up to 100 cards in deck - 50 pairs

### Assumptions

1. Write heavy system as a lot of users play the game (input their moves / pick cards) than just read data from it.
2. Data about picking cards etc. should live in the session and not be persisted
3. Only data about game ids and winners should be persisted in a db
4. system is constantly updated with players' inputs and the updates should reflect in each player's client devices. Therefore, consistency should be high.

### Constraints and Data estimation
1. 1 billion active users per day = at most 500 million games per day with 2 players
2. 5 - 10 minutes per game so around 3.5 million games at any time.
3. Once a game is started only inputs are players flipping cards so around 3.5 million requests per second
4. 5 GB data per day to store info about games

## High Level Design
![multiplayer-memory-game](https://i.imgur.com/yNkofyq.png)