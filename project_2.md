---
layout: default
title: "Project 2: 8 Puzzle Solver"
---

## Project 2: 8 Puzzle Solver

**Competency:** Algorithms & Data Structures

## Overview

This project was created from scratch for the purpose of this portfolio, with inspiration from my CS-370 class taken in early 2026. It is a Python and Jupyter Notebook based project that builds an agentic model using the A2C (Advantage Actor-Critic) algorithm to solve the sliding 8 puzzle.

The sliding 8 puzzle is a sliding tile puzzle in a 3x3 grid with 8 numbered tiles and one empty space. The puzzle is considered finished when the numbered tiles are ordered from 1 to 8, starting from the top left, and the 9th grid space being the empty space. The puzzle starts from a randomized position, and for the purpose of the agent it is suffled x amount of times (starting with 1 time) and increases by one each time it hits a win threshold to increase the difficulty of the puzzle. The purpose of this project was to demonstrate my understanding of algorithms and data structures, as well as my ability to implement them in a real world application that I was personally looking to use to solve this type of puzzle in a game.

## Technologies Used

- **Python**
- **Jupyter Notebook**
- **TensorFlow / Keras** (neural network frameworks)
- **NumPy**
- **Matplotlib** (visualization of grid)

## Project Structure

The project has two files:

- **`Puzzle.py`** — The puzzle environment class that manages game state, moves, rewards, and win/loss detection
- **`8PuzzleSolver.ipynb`** — The Jupyter Notebook that imports the environment, builds the A2C actor and critic networks, trains the model, and then tests it

## How It Works

### The Puzzle Environment

The `Puzzle` class creates and represents a 3x3 grid. It tracks the board state, the position of the empty space, and determines valid moves (Left, Up, Right, Down) based on that position. I also created the reward system for the agent in this class to guide it towards solving the puzzle:

```python
def get_reward(self):
    if np.array_equal(self.state, self.goal_state):
        return 1.0         # Solved the puzzle
    if self.mode == 'blocked':
        return self.min_reward - 1  # No valid moves
    if self.mode == 'invalid':
        return -0.75        # Tried an invalid move
    if self.mode == 'valid':
        return -0.04        # Small penalty per step to encourage efficiency
```

Due to some complications with getting the agent solve the puzzle at higher shuffle counts, I decided to try to guide its learning a bit more. So I did some research and ended up implementing the Manhattan distance metric, which measures the distance between two points by summing the absolute differences of their coordinates. It follows grid-like paths rather than straight lines which is perfect for a grid based puzzle, and the agent receives bonus "points" for moves that bring tiles closer to their goal positions:

```python
def act(self, action):
    prev_dist = self.manhattan_sum()
    self.update_state(action)
    curr_dist = self.manhattan_sum()
    reward = self.get_reward()
    # Reward for reducing Manhattan distance
    if self.mode == 'valid' and not np.array_equal(self.state, self.goal_state):
        reward += 0.1 * (prev_dist - curr_dist)
    ...
```

### The A2C Algorithm

The A2C (Advantage Actor-Critic) algorithm uses two neural networks:

- **Actor** — A policy network with two hidden layers of 128 units each that outputs logits over the 4 possible actions. Invalid actions are omitted before sampling occurs
- **Critic** — A value network with the same layout as the actor (2 hidden layers, 128 units each) that outputs an estimate of how good the current board state is

```python
def build_actor(input_shape, num_actions, hidden_size=128):
    inputs = tf.keras.Input(shape=input_shape)
    x = Dense(hidden_size, activation='relu')(inputs)
    x = Dense(hidden_size, activation='relu')(x)
    policy_logits = Dense(num_actions, activation='linear',
                          name='policy_logits')(x)
    return tf.keras.Model(inputs=inputs, outputs=policy_logits)
```

The training loop uses Generalized Advantage Estimation (GAE) to calculate advantages with hopefully reduced variance, and uses an entropy bonus to prevent the policy from locking in to a bad solution path early:

```python
# Entropy bonus prevents premature convergence
entropy = -tf.reduce_mean(
    tf.reduce_sum(tf.nn.softmax(policy_logits)
                  * tf.nn.log_softmax(policy_logits), axis=1)
)
actor_loss = -tf.reduce_mean(log_probs * advantages) - 0.01 * entropy
```

### Scaled Learning

I found out early on that this was a pretty complicated puzzle for the agent to solve, and solve consistently. So, rather than throwing the hardest puzzles at the agent immediately, I used a scaling type of learning that gradually increases difficulty once the agent hits a certain win rate. The agent starts with puzzles shuffled only 1 move from the solution and advances to harder puzzles (up to 30 shuffles) only after getting a 70% win rate or higher at the current shuffle count:

```python
if len(level_wins) >= advance_window:
    level_win_rate = sum(level_wins[-advance_window:]) / advance_window
    if level_win_rate >= advance_threshold and n_shuffles < max_shuffles:
        n_shuffles += 1
        level_wins = []
```

I found that this approach allowed the agent to build a decent foundation on easier puzzles before attempting the harder ones. This ended up allowing the agent to get even further in the shuffle amounts, and thus improved the stability of the training and the final performance.

## Skills Demonstrated

- Reinforcement learning algorithm design (A2C with GAE)
- State space representation (Manhattan distance)
- Neural networks with TensorFlow and Keras
- Scaling learning design
- Python and Jupyter Notebook coding

[Back to Home](./)
