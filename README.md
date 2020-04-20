# [Ocean of Code](https://www.codingame.com/contests/ocean-of-code/leaderboard/global)
###### - bot contest on [codingame](http://codingame.com)

## Ranking

![Final Contest Rank](https://github.com/SpiritusSancti5/OceanOfCode/blob/master/oceanofcode.jpg)

###### 3rd place in UK

This was by far the most complicated contest on codingame, might be partly because of my approach. However, with one month length, I had a lot more fun playing with pointers and trying to build and squeeze in a neural network. ML unfortunately isn't something the platform supports, mainly due to the bot size restriction. The limit is 100,000 characters. So I prioritized improving the bot's structure, over working on the ML part.

## Intro

Ocean of Code is a Captain Sonar inspired, 2 player challenge, where each player controls a submarine. Your goal is to eliminate the other submarine, but you don't know where it's located. All you have is the opponent's output for the previous turn, which means you have to track them down first.

## Structure

### Gamestate

This is a hidden information game, where there's a lot of data to process and update every turn to make sure you are accurately tracking everything.

You have to track your opponent's possible positions on the map. To start off, they could be anywhere. At the same time you need to keep track of your own movement. If you navigate around a tight area you might reveal your position much sooner than your opponent. Therefore the best starting position would be in the middle of the ocean.

Each action performed by a player gives away information. There are 7 possible actions that can be combined or played separetly in any order. To track accurately you need to process both players' actions in the order they occured.

#### Cell

My main focus was to have a structure that allows me to quickly update and move information. Keep it readable and optimize once it's finished. My game state was a map of 225 Cells.

Each Cell had:
* A bitset of length 8, where the bits represented:
1. land-type
2. my path
3. opponent's path (only used when I knew the exact path taken)
4. my possible positions
5. opponent's possible positions
6. my mines
7. opponent's mines (only after it was first updated and updated only when there was a single possible path)
8. explosions (wanted to make it easier to track opponent's life change, never used it though)
* An array of 225 representing the 15x15 map tiles. Each containing the BFS length to every other place on the map. This was calculated turn 1.
* A vector for each player containing possible paths. I store the paths in the form of lists of pointers to Node objects.

**Note** The possible positions a torpedo can hit from a cell, can also be precalculated. Same goes for any MOVE+SILENCE combination.

### Node

For convinience and ease of access to information about the gamestate, I made sure i added every single proprety I needed.

Each node contains:
- Node* start - to quickly access the starting point
- Node* parent - quickly access the previous node
- vector<Node*> children - used when i had to split the list into multiple ones due to SILENCE skill
- int x y - as position
- int timer - to track how many turns ago a mine was layed
- int life_expectancy - this helps me track life change on the submarine's path
- Visits* vis - an object to keep track of visited cells and do some other related operations
- vector<Node*>* mines - quick reference to mines layed on the path this node is on

### Bot Logic

Below is the final plan i had in mind for the bot, however a previous incomplete version of the bot worked better than it and i didn't find the issue yet, why the newest version doesn't yield the best results. Perhaps the newer version plays too passively and doesn't take risks at the right time.

The logic is split into two states, one for when the enemy position is known and one where it's still unknown. 

Torpedoes are only fired when there is a single 3x3 or smaller area where the enemy is located. This is of the *enemy-location-is-known* state. For this state the goal was to write a minimax or any predictor to find the best move combinations. I didn't get around doing this yet, as there are a lot of tiny details to take into consideration and i absolutely wanted to avoid a rewrite late in the contest.

For the *enemy-location-is-unknown* state I simply try to keep prevent giving too much info away while moving.

For the move generation of either state, I find all possible combinations of MOVE + SILENCE that are available and score them.
Torpedo shots are scored separately. Once before any movement, once during movement if it's a MOVE+SILENCE combo and once after. This ends up with a lot of calculations of course, but i didn't time out much thanks to using C++

The bot triggeres mines any time it seems most convinient to do so. Mostly to cut down on possible enemy paths and guess their location. This didn't work so well when people fixed their tracking based on mine explosions, since they could then find my bot faster.

## Machine Learning

### Motivation

Due to the nature of the game, using ML is a tedious and very demanding task. I am sure a lot people might be quick to disagree with me, but there's no doubt that not all the aspects of the game are covered even by the top bots on the leaderboard and there's still a lot of information that remains hidden even from them.

There are a lot of trends a bot would follow if it were to aim for the optimal move from any given position for example. Not to mention there's a deeper dive into statistics one has to take to try and cover everything.

To avoid all this and try avoid any unnecessary extra work, i have attempted to build a model and train it in two phases.

### Game state predictor

First phase would be obviously predicting the game state accurately. For that i used as input all the game information and generated a map which ideally should contain all the map information required to win. Perhaps it would be best to have a different map generated for each different detail that is being tracked, but there's no space for all that in the CG IDE. There's hardly any space for a regular bot!

As output I would get an array of 225 ints with values between 0 and 15 or higher, depending on the amount information I try to predict. Each bit of the number can represent a certain aspect of the map's cell, possible enemy position, possible mine or something else.

### Commands generator

I then used the original input from the game turn with this processed map to generate an output that represents the bot's commands for the turn. There are 7 outputs that represents the priority of the individual command.

I corrected the output by using replays of the top 10 bots at the time of training this model. Basically a type imitation learning.

### Result

I spent about a week of my free time on this in order to get a fully functional bot that correctly outputs **MOVE N TORPEDO** about 90% of the time and random weird, yet valid commands on any other turn.

I might have failed with this because either of some bug in the backpropagation, not enough training data or any of the many possible flaws with the model itself. However this was back when silver was the highest league, so it might have been too soon either way to train it, not to get started though.

The next attempt would be to take either of the two parts and train them individually and then add them back together later.

## Thanks for reading and see you in the next contest!

[CodinGame Spring Challenge 2020](https://www.codingame.com/contests/spring-challenge-2020)
