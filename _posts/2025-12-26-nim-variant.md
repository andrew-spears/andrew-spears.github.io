---
title: Combinatorial Game Theory for Codenames
date: 2025-12-26 10:00:00 -0500
categories: [game theory, algorithms]
tags: [game theory]
math: true
---

## Codenames

Codenames is a popular word-based party game where players are divided into two teams, each with a "spymaster" who gives one-word clues to help their teammates identify a set of words from a grid. The strategy combines multiple aspects, including word association, balancing risky and conservative play, and knowing your teammates' thought process.

When playing with a large diverse group, the game usually boils down to whichever spymaster can come up with the best clues that perfectly combine many clues. Some research has been done on finding the optimal clues for a given board, with interesting results - Friedman and Panigrahi found that with a certain model of word similarity, an optimal clue on round 1 can connect over 6 words before accidentally hinting an opponent's clue, in expectation [1]. 

However, we are focusing on the ordering of clues, which is often equally as important as the clues themselves. 

### Motivating Example

Suppose you are the spymaster for Team A and you have target words {apple, banana, cherry, spider} while Team B has {tomato, carrot, fern}. You are debating between the clues "fruit", which you expect will help your team guess {apple, banana, cherry}, and "arachnid" which will give away {spider}. It is tempting to go for the big group first, but if you do, then your opponent can easily say "plant" and win the game. If you do "arachnid" first, then your opponent has a difficult time connecting all three words, and you will be able to win with "fruit" next turn. Thus, the order of clues matters a lot to not give away information to the other team.

If we strip away the complexity of coming up with clues and reading your teammates' minds, we can reduce this to an abstract game theory problem.

## Abstraction

Two players each have target sets A = {a₁, ..., aₙ} and B = {b₁, ..., bₘ}. There's a shared collection of "clues," where each clue is a chain of alternating subsets of A and B, ordered by similarity.

### Gameplay

Players alternate choosing clues (repeats allowed). When a clue is picked, its first set is removed from that clue's chain and those targets are eliminated. First player to eliminate all their targets wins.

### Example clue

$$\{a_1, a_3\} \to \{b_1, b_3\} \to \{a_2\} \to \{b_2\}$$

This models something like clue="small" with targets a₁="tiny", a₂="dog", a₃="ant" for team A and b₁="mouse", b₂="horse", b₃="rat" for team B. In real Codenames, some such chains are unlikely or impossible because of the structure of English semantics, but we ignore that for this abstract model.

### Full game example

Suppose our initial state has the following chains:

$$\begin{align*}
\text{Chain 1}: & \quad \{a_1, a_2, a_3, a_4\} \to \{b_1, b_2, b_3, b_4\} \\
\text{Chain 2}: & \quad \{a_5\} \to \{b_3, b_4\} \\
\text{Chain 3}: & \quad \{b_2, b_3\} \\
\text{Chain 4}: & \quad \{b_1\}
\end{align*}$$

with player A to move.

Let's see what happens if player A chooses the greedy option and plays Chain 1 first:

| Action | Chain 1 | Chain 2 | Chain 3 | Chain 4 | Explanation |
|--------|---------|---------|---------|---------|-------------|
| Initial | $\{a_1, a_2, a_3, a_4\} \to \{b_1, b_2, b_3, b_4\}$ | $\{a_5\} \to \{b_3, b_4\}$ | $\{b_2, b_3\}$ | $\{b_1\}$ | A has 5 targets, B has 4 |
| A plays Chain 1 | $\{b_1, b_2, b_3, b_4\}$ | $\{a_5\} \to \{b_3, b_4\}$ | $\{b_2, b_3\}$ | $\{b_1\}$ | A clears 4 targets at once |
| B plays Chain 1 | — | $\{a_5\}$ | — | — | B clears all 4 targets and wins |

B wins because playing Chain 1 opened up an instant win for B.

What if A played more conservatively at the start?

| Action | Chain 1 | Chain 2 | Chain 3 | Chain 4 | Explanation |
|--------|---------|---------|---------|---------|-------------|
| Initial | $\{a_1, a_2, a_3, a_4\} \to \{b_1, b_2, b_3, b_4\}$ | $\{a_5\} \to \{b_3, b_4\}$ | $\{b_2, b_3\}$ | $\{b_1\}$ | A has 5 targets, B has 4 |
| A plays Chain 2 | $\{a_1, a_2, a_3, a_4\} \to \{b_1, b_2, b_3, b_4\}$ | $\{b_3, b_4\}$ | $\{b_2, b_3\}$ | $\{b_1\}$ | A clears $a_5$, keeping Chain 1 safe |
| B plays Chain 3 | $\{a_1, a_2, a_3, a_4\} \to \{b_1, b_4\}$ | $\{b_4\}$ | — | $\{b_1\}$ | B clears $b_2, b_3$, which also removes them from chain 1 |
| A plays Chain 1 | $\{b_1, b_4\}$ | $\{b_4\}$ | — | $\{b_1\}$ | A clears remaining 4 targets and wins |

So A wins by waiting to use their big move and avoiding opening up options for B. Notice that this is closely modeling the situation described earlier: chain 1 corresponds to "fruit" or "plant", and chain 2 corresponds to "arachnid".

## Solving

There are two natural questions: first, given a game state, which player has a winning strategy? Second, what is that winning strategy? Neither will have a clean and easy answer, but we can find some shortcuts for specific cases.

As a starter, let's consider the following state:

$$\begin{align*}
\text{Chain 1}: & \quad \{b_3\} \to \{a_1, a_2\} \to \{b_1\} \\
\text{Chain 2}: & \quad \{b_2, b_3\} \\
\end{align*}$$

No matter who is to move, the game is a win for player A. This is because target $b_1$ is completely 'trapped' by a win condition for A. For B to win, they must first clear $b_3$ and $a_1, a_2$ to be able to use chain 1. but $a_1, a_2$ are the only targets for A, so A would have won the game before B could get there.
<!-- 
We can achieve the same result from a less obvious setup:

$$\begin{align*}
\text{Chain 1}: & \quad \{b_3\} \to \{a_1 \} \to \{b_1\} \\
\text{Chain 2}: & \quad \{a_2\} \to \{b_3\}  \\
\end{align*}$$

Here, player B still needs to remove $a_1$ to reach $b_1$. But to remove $a_1$, $b_3$ must be removed first.   -->

### Logically Forced Wins

This incentivizes a different representation of the game state as a set of logical implications. each target is a proposition, and the number of times a chain has been played implies certain sets of targets.

For example, the above state can be represented as:

$$\begin{align*}
a_1 & \iff 2x_1 \\
a_2 & \iff 2x_1 \\
b_1 & \iff 3x_1 \\
b_2 & \iff x_2 \\
b_3 & \iff x_2 \lor x_1 \\
\end{align*}$$

Where $nx_i$ is the proposition that chain $i$ has been played $n$ times between both players. This list of rules was constructed by simply collecting all occurrences of each target and taking the logical or of all the chains which could have cleared that target (with the number of times they would need to be played). We also implicitly have the rules

$$\begin{align*}
x_i & \implies x_{i-1} \\
A & \iff a_1 \land a_2 \land ... \\
B & \iff b_1 \land b_2 \land ... \\
\end{align*}$$

Where A and B are the propositions that players A and B have won, respectively.

Why is this useful? Recall that B was unable to win without letting A win first. Let's try to prove that by assuming B and deriving A.

$$\begin{align*}
B & \iff b_1 \land b_2 \land b_3 \\
    & \implies 3x_1 \land x_2 \land (x_2 \lor x_1) \\
    & \implies 3x_1 \\
    & \implies 2x_1 \\
    & \implies a_1 \land a_2 \\
    & \implies A \\
\end{align*}$$

So it really is impossible for B to win. This gives us a general tool for analyzing game states: if we can prove that one player's win condition implies the other player's win condition, then the first player cannot have a winning strategy.

### General Solution

What about situations where neither player has a logically forced win, but where speed or strategy actually matter? This is more difficult. 

One approach is algorithmic, using minimax search to consider possible paths in the game tree:

```python
# pseudo-code
def winner(chains, A_targets, B_targets, player):
    if A_targets is empty:
        return A
    if B_targets is empty:
        return B

    for chain in chains:
        if chain is empty:
            continue

        # play this chain: remove first set from chain and from targets
        first_set = chain[0]
        new_chain = chain[1:]
        new_A = A_targets - first_set
        new_B = B_targets - first_set
        new_chains = chains with chain replaced by new_chain

        result = winner(new_chains, new_A, new_B, other(player))

        if result == player:
            return player  # found a winning move

    return other(player)  # no winning move exists
```

Since the number of possible moves at each step is limited by the number of chains, and each chain can only be played a finite number of times (since each play removes targets), the game tree is often tractable for small games. Alpha-beta pruning can further optimize this search, but it is unclear whether we can do better than a brute force approach.

## Conclusion

Often a game with multiple strategic elements is chalked up to a single dominant factor. In Codenames, coming up with good clues is difficult enough that more subtle aspects of strategy are often ignored. However, as we have seen, these more subtle strategic elements, like the order in which clues are given, can be just as important as the clues themselves. By abstracting away the complexity of word association, we can analyze this ordering problem in isolation and gain insights into optimal play. And, if we are lucky, these same abstracted games might mirror other real-world situations that inform how we approach the original game.

## References

[1] Friedman, S., & Panigrahi, S. (2021). Codenames: Optimal Clue Generation Using Word Embeddings. COS 521 Final Project Report, Princeton University. 


