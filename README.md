# Documentation
This document contains information about the reinforcement learners and the processes related to getting them to do things.

# Learners
`learners.py` contains four learners. The first is the `BaseLearner` which contains the core functionality of a learner.
The learners are the building blocks of game-playing agents.

## QLearn and SARSA 
`QLearn` and `SARSA` are separate implementations of learners. 
`SARSA` assumes that exploration remains on whereas `QLearn` optimizes for offline us (exploration turned off after training). 

## HistoryManager
`learners.py` provides a learner that has a memory, called `HistoryManager`.
It is a learner that makes decisions based on information 'now' (at time step `t`) or 'the past' (time step `t-x` where `history_levels >= x > 0`.
The history manager rewards the decider for selecting which sub-learner to use as well as the sub learners themselves.

## MetaLearner
The meta learner learns from two different sublearners of any kind. The meta learner learns to select the sublearner that yields the highest reward and trains all sublearners simultaneously (this doesn't apply to `HistoryManagers`, since their learning is based on decisions and not the game itself).

The meta learner can be configured to take either the state space for the left or the right learner for its own state space with the `side` property. There seems to be a trend for one-sided learners being slightly wores than double-sided learners.

# Agents

Although learners are generic, agents need to be specifically tailored to the game they are designed to play.
In brief, an agent will access the game's internals and manage learners. 
Agents should implement the `perform` method, which does the following:

1. Get the state of the game
2. Simplify the state, or put it into context
3. Set the learner state to the game state
4. Record a decision from the learner
5. Play the recorded decision
6. Check if the recorded action was good or bad (e.g. 'the score went up' is good)
7. Apply rewards to the learners based on the actions
8. Return the result of step (6.)

# Other components

## Renderer
 
The renderer can be found in `render.py` and uses _pygame_ to render on the screen. Currently the `render` method takes a collection of items and a score. It renders the first three items a specific color and the score in the corner.
 
The renderer exists only for visualization and debugging purposes, and is not actually necessary.

## Task + Environment

In this package, one game is implemented -- with three difficulty levels (essentially making it three different games).

The game consists of a mouse (the player, white), cheese (yellow) and a trap (red).
The aim of the game is to accumulate as much cheese as possible and avoid touching a trap.
The world loops around itself.

If the difficulty is set to 0, there is no trap. At 1, the trap is there. At 2, the trap randomly moves towards the player (acts like a cat).


# Extending the provided agents for other games

The provided agents already do a lot of the heavy lifting in a generalized context. `Agent:get_fov(fov)` might need to be replaced, along with the `Agent:decide(learner)` method. The `decide` method is responsible for updating the selected action in non-chosen learners and the `get_fov` method generates a tuple based on the game state.

## Case study: Mouse game (difficulty 0)

Assume the goal is to create a simple SARSA agent that plays the game at difficulty 0.

`agent.py` contains the agent code. Most of the methods in the `Agent` class are helper methods to represent and mutate game internals with respect to what the agent should see. `get_fov()` is the method that creates the current state in the format for the learner (the provided implementation creates a cone-like field of view). The learners should be able to learn from any state space, but the design of the agent decides what state space the learner attempts to learn from.

Instead of building a field of view, we could pass all of the game's internals, but this creates a very large state space. 
The `get_fov()` method calls `create_cone()`, which returns a 2-tuple (cheese, traps). At difficulty 0, there are no traps, so the 2-tuple `(cheese, traps)` will simplify to `(cheese, 0)`, so the code returns only `cheese`.

The `perform()` method is the _meat_ of the agent. This is where the state is fetched (with `get_fov()`) and passed on to the learner. The learner is then activated by selecting an action from the learner and playing it in the game. Once the action has been performed, the agent should check if the game changed for better or worse. In the provided agent, this happens in the `check_reward()` method which returns whether or not the score increased, decreased, or neither as an integer between 1 and -1. The agent can then use that value to determine whether or not to administer a reward to the learner.

The agent code must also handle the decision making at the meta level -- i.e. it must implement some logic to decide which learner makes the final decision (and the traversal down the tree to that learner).

