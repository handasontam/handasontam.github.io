---
layout: post
title: "Fugitive #1: Tracking the Fugitive with Graph-Based Deduction"
date: 2024-09-20 15:09:00
description: Tracking the Fugitive with Graph-Based Deduction
tags: board-game data-structure algorithm game-theory reinforcement-learning fugitive
categories: sample-posts
featured: false
citation: true
thumbnail: assets/img/fugitive.webp
---

In the realm of two-player deduction games, "Fugitive" stands out as a thrilling cat-and-mouse chase. One player, the Fugitive, tries to escape by moving through a series of hidden locations, while the other player, the Marshal, attempts to deduce and uncover these hideouts. Today, we'll dive into a fascinating AI implementation that brings the Marshal's deductive reasoning to life using graph theory.

I recently had the pleasure of playing Fugitive with friends and found it to be an engaging game of strategy and deduction. As someone who enjoys analyzing optimal strategies in games, I was naturally drawn to exploring the strategic depths of Fugitive.

In this series of posts, I'll model the game using graph theory and reinforcement learning techniques. My goal is to uncover optimal strategies for both players and gain insights into the game's balance.

Interestingly, when researching online strategies, I discovered a divide in player opinions regarding which role has an advantage. Some players claim the Marshal can never win (as discussed [here](https://boardgamegeek.com/thread/2997124/cant-win-as-marshal)), while others question if the Marshal can lose at all (see [this thread](https://boardgamegeek.com/thread/1757632/can-the-marshal-lose)). This disparity in experiences adds an intriguing dimension to our analysis.

### Game Overview

Fugitive is a two-player card game where one player takes on the role of a Fugitive trying to escape, while the other plays as the Marshal attempting to catch them. The game uses a deck of 43 numbered cards (0-42) representing potential hideouts. The Fugitive wins by playing the #42 card, signifying a successful escape, while the Marshal wins by uncovering all of the Fugitive's hideouts before they can escape.

Players take turns, with the Fugitive starting at hideout 0. The Fugitive establishes hideouts by playing cards face-down in ascending order. Each new hideout must be numbered within 1, 2, or 3 of the previous one. To cover larger distances, the Fugitive can "Sprint" by placing additional cards underneath a hideout. These sprint cards have values (+1 or +2) that allow the Fugitive to jump further than the usual 3-card limit.

The Marshal attempts to guess and reveal these hideouts. The game's tension builds as the Fugitive tries to outmaneuver the Marshal, using special "Sprint" moves to cover larger distances, while the Marshal uses deduction and strategy to close in on their target.


### Research Questions

Through this series, we aim to answer the following questions:

1. How can we optimally take and update notes as the Marshal?
2. What are the optimal strategies for both players?
   - For the Marshal:
     - Which deck should the Marshal draw from? Is drawing exclusively from a higher pile to create a 'roadblock' useful?
     - Which numbers should the Marshal guess?
     - Should the Marshal guess more than one number?
     - Should the Marshal aim to win by manhunt or by uncovering all hideouts before the Fugitive escapes?
   - For the Fugitive:
     - Which hideouts should the Fugitive establish?
     - Which sprints should the Fugitive use?
     - Does bluffing help?
3. What are the winning probabilities for both players if they play optimally? This will help us determine if the game is balanced.

In this post, we'll tackle the first question by exploring an advanced note-taking system for the Marshal using graph-based deduction.

### The Marshal's Dilemma

Imagine you're the Marshal. You know the Fugitive started at location 0 and is making their way towards location 42. Each turn, they place a new hideout card face-down, occasionally "sprinting" by adding extra cards to cover larger distances. Your job? Guess these hideouts before the Fugitive reaches their final escape.

The challenge lies in efficiently tracking all possible locations for each hideout based on the limited information you receive. This is where our `MarshalNotes` class comes in, leveraging a directed graph to model the Marshal's deductions.

This approach will also be useful when we implement the action mask in the reinforcement learning setting for filtering out illegal actions. Consider the action space for the Marshal's guess:

1. The Marshal should only guess numbers within the possible range of hideouts. For example, if the Fugitive plays 2 hideouts without sprinting at the start of the game, the Marshal should not guess any number larger than 6.

2. The Marshal must not guess numbers that are in their hand, have already been revealed as hideouts, or used as sprints.

3. The Marshal can avoid guessing numbers that have already been guessed, with one caveat: A wrong guess only reveals that the number is not a hideout among those already placed. It could still be a future hideout.

As the game progresses, the Marshal must keep track of the possible range for each hideout and update this information based on guesses, revealed cards, and drawn cards.

Based on the above considerations, the official board game provided a Marshal's notepad ([example](https://apps.apple.com/hk/app/fugitive-notepad/id1135595567)), which consists of a grid of 41 numbers, and the marshal can mark the number that has been known not to be a hideout. 
This helps the marshal to keep track of the potential hideouts.
However, we can do much better than this with a more sophisticated data structure.

## Beyond the Notepad: Graph-Based Deduction in Fugitive

### Why Graphs Trump Simple Notepads

While a traditional notepad approach (crossing out numbers from 1-42) is intuitive for human players, our graph-based `MarshalNotes` offers several advantages for AI implementation:

1. **Sequence Information**: The graph explicitly models the order of hideouts. A simple notepad might tell you that 15 and 20 are possible hideouts, but not which comes first. The graph maintains this crucial sequence information.

2. **Constraint Propagation**: When new information is learned, the graph can efficiently propagate these constraints. For example, if we learn the second hideout is 7, this immediately restricts the possible range for the third hideout, which in turn affects the fourth, and so on. A simple notepad struggles to represent these cascading effects.

3. **Sprint Modeling**: The graph can easily incorporate the effects of sprints. If we know a sprint occurred between two hideouts, the graph can represent the expanded range of possibilities, something a basic notepad can't capture.

4. **Efficient Querying**: Want to know all possible locations for the 3rd hideout? In the graph, this is a simple query of nodes at depth 3. With a notepad, you'd need additional data structures or complex logic to maintain this information.

5. **Revealing Impossibilities**: As the game progresses, certain hideout sequences become impossible. The graph naturally represents this by removing nodes or edges, while a notepad approach might struggle to identify these implicit impossibilities.

### Graphing the Escape Route

Let's break down the key components of our graph-based approach:

1. **Nodes**: Each node in our graph represents a potential hideout location. It's defined by two values:
   - Depth: The hideout's position in the sequence (0 for start, 1 for first hideout, etc.)
   - Value: The numeric location (0-42)

2. **Edges**: An edge between two nodes indicates that it's possible for the Fugitive to move from one hideout to the next, given the game's rules and known information.

3. **Hideouts Range**: We maintain a 2D array `hideouts_range` where `hideouts_range[i][j] = 1` means the i-th hideout could potentially be at location j.

4. **Graph Consistency**: We ensure that the graph remains consistent with the game rules and known information. This involves removing nodes and edges that become impossible based on new information, and updating the `hideouts_range` array accordingly.

### Building the Graph

Let's walk through how we construct and update this graph as the game progresses:

#### 1. Initialization

We start with a single node (0, 0), representing the known starting point.
```python
self.graph = nx.DiGraph()
self.graph.add_node((0, 0))
self.hideouts_range = np.zeros((MAX_HAND_SIZE, 43), dtype=np.int8)
self.hideouts_range[0, 0] = 1
self.n_hideouts = 1
```

#### 2. Adding New Hideouts

When the Fugitive places a new hideout, we expand our graph:
```python
def add_new_hideout(self, n_sprint: int, filtered_vals: List[int]) -> None:
    prev_depth = self.n_hideouts - 1
    new_depth = self.n_hideouts
    nodes_from_prev_depth = [node for node in self.graph.nodes() if node[0] == prev_depth]
    for prev_node in nodes_from_prev_depth:
        prev_value = prev_node[1]
        min_new_value = prev_value + 1
        max_new_value = min(prev_value + 3 + 2 n_sprint + 1, 42)
        for new_value in range(min_new_value, max_new_value):
            if new_value in filtered_vals: # Eliminated seen values
                continue
            new_node = (new_depth, new_value)
            self.graph.add_edge(prev_node, new_node)
            self.hideouts_range[new_depth, new_value] = 1
        self.n_hideouts += 1
```

This method considers:
- The previous hideout's possible locations
- The number of sprint cards played
- Any values we know can't be hideouts (filtered_vals)

It then creates new nodes and edges for all potential new hideout locations.

#### 3. Eliminating Possibilities

As the Marshal gains information (through correct guesses or drawn cards), we update our graph:

```python
def eliminate_vals(self, vals: List[int]) -> None:
    nodes_to_remove = [(d, v) for d, v in self.graph.nodes() if v in vals and d > 0]
    self.graph.remove_nodes_from(nodes_to_remove)
    self.hideouts_range[:, vals] = 0
    self.remove_unreachable_nodes()
```

This removes any nodes that we now know can't be hideouts and updates our `hideouts_range` accordingly.

#### 4. Revealed Hideouts

When a hideout is revealed (either through a correct guess or the Fugitive reaching location 42), we update our graph to reflect this certain information:
```python
def hideout_revealed(self, depth: int, hideout: int) -> None:
    # Remove all nodes at the given depth except the revealed hideout
    nodes_to_remove = [(d, v) for d, v in self.graph.nodes() if d == depth and v != hideout]
    self.graph.remove_nodes_from(nodes_to_remove)
    
    # Update hideouts_range for this depth
    self.hideouts_range[depth, :] = 0
    self.hideouts_range[depth, hideout] = 1
    
    # Remove unreachable nodes
    self._remove_unreachable_nodes()
```


### Maintaining Graph Consistency

After each update, we call `_remove_unreachable_nodes()` to ensure our graph remains consistent:

This method performs two crucial cleanup operations (Assuming depth is zero indexed): 
1. Removing nodes unreachable from root. This requires a linear scan of the graph from depth 1 to depth n_hideouts-1 and remove the nodes that are not reachable from the previous depth (i.e. nodes with in-degree of 0).
2. Removing nodes unreachable to the latest hideout (leaf at depth n_hideouts). This requires a linear scan of the graph from depth n_hideouts-2 to depth 0 and remove the nodes that are not reachable to the next hideout (i.e. nodes with out-degree of 0).


```python
def _remove_unreachable_nodes(self):
    # Remove nodes unreacheable from root
    for depth in range(1, self.n_hideouts):
        nodes_at_depth = [node for node in self.graph.nodes() if node[0] == depth]
        for node in nodes_at_depth:
            if self.graph.in_degree(node) == 0:
                self.graph.remove_node(node)
                self.hideouts_range[node[0], node[1]] = 0
    
    # Remove nodes unreachable to the leaf at depth n_hideouts
    for depth in range(self.n_hideouts-2, 0, -1):
        nodes_at_depth = [node for node in self.graph.nodes() if node[0] == depth]
        for node in nodes_at_depth:
            if self.graph.out_degree(node) == 0:
                self.graph.remove_node(node)
                self.hideouts_range[node[0], node[1]] = 0
```

This ensures that every node in our graph is part of a valid path from the start to the current game state.

### Conclusion

This graph-based approach to modeling the Marshal's deductive reasoning in Fugitive showcases the power of applying computer science concepts to game AI. By representing the game state as a directed graph, we create a flexible and efficient way to track complex possibilities and make intelligent decisions.

The next time you play Fugitive (or design an AI for a similar deduction game), consider how graph theory might help you model the hidden information and logical constraints. Happy hunting!
