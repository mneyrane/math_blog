---
title: A Simple Solution of Quantik
author: max
category: blog
tags:
- game theory
- combinatorics
---

## Motivation

During my master's degree in applied mathematics, I was quite excited about the topic of reinforcement learning and its use in games, especially in the discovery of novel playstyles or strategies. Without going into much detail, I wanted to implement some reinforcement learning algorithms for a toy game that would not require thousands of dollars of computing cluster usage to observe any results. I wanted the toy game to be relatively novel, simple, but also have a nontrivial optimal strategy. I discovered Gigamic, a board game publisher, and their lineup of abstract strategy games. From those games, I found and became interested in [Quantik by Nouri Khalifa](https://en.gigamic.com/modern-classics/564-quantik-3421271322313.html). This led to some recreational mathematics research that I want to share here!

## The Gameplay and Rules of Quantik

Quantik is essentially a two-player adversarial version of [Sudoku](https://en.wikipedia.org/wiki/Sudoku).
The game is played by having players alternate placing shapes on a 4-by-4 grid, subject to placement rules.
There are four distinct shapes, and each player starts with two of each shape.
The shapes are distinguished by colour to indicate which player they belong to.
A *zone* is a row, column, or region in the grid. Each zone is illustrated in the figure below.

{% include figure.html
  id="zones"
  file="assets/posts/quantik-solution/zones.svg" 
  width="640px"
  alt="" 
  caption="All zones in Quantik. From top to bottom, each row of diagrams corresponds to rows, columns, and regions, respectively." %}

A shape can be placed in a cell if and only if the cell is empty and the same shape *of opposite colour* is not present in a zone containing the cell.
The game ends when a zone is *completed*, that is to say, the zone contains four distinct shapes (regardless of colour), or if a player has no legal moves. In both cases, the last player who placed a shape wins, so there are no draws.

To clearly define the shapes, colours, and turn order, we adopt the following conventions in this article:

- the four distinct shapes are a circle ($$○, ●$$), square ($$□, ■$$), diamond ($$◇, ◆$$), and triangle ($$△, ▲ $$)
- Player 1 has white shapes and Player 2 has black shapes
- Player 1 refers to the player that places first

The next figure illustrates an example of a Quantik game where Player 1 wins by completing a zone on the ninth turn. Observe that the final move does not contradict the placement rule.

{% include figure.html
  id="example_game"
  file="assets/posts/quantik-solution/example_game.svg" 
  width="480px" 
  alt="" 
  caption="An example game of Quantik with a Player 1 victory. The sequence of turns is read from left-to-right, top-to-bottom." %}

The placement rule is clarified in the next figure. Remember, once a shape is placed in a cell, its opposite colour shape cannot be placed in the same row, column, or region containing that cell.

{% include figure.html
  id="placement_rule"
  file="assets/posts/quantik-solution/placement_rule.svg" 
  width="320px" 
  alt=""
  markdown="1"
  caption="Illustration of the placement rule. Each red cell appears in a zone that also contains Player 1's (white) diamond. Left: Player 2 is not allowed to place their (black) diamond in any red cell. Right: Player 1 is allowed to place their (white) diamond in a red cell." %}


## The Search for a Solution

Initially, I wanted to apply reinforcement learning to Quantik, but in a short time I pivoted away from this.
I was convinced that a brute-force search algorithm could feasibly calculate perfect play.
Indeed, it turned out to be!
I wrote a [solver for Quantik in C](https://github.com/mneyrane/Quantik-solver) that correctly labels each valid move as a win or loss under perfect play, at any reachable game state.
As a result, *Player 2 is guaranteed to win under perfect play* when starting from an empty board.
I shall discuss in the next section why this turns out to be the case!

One problem I ran into when implementing the solver was that it became impractically slow when searching from states near the start of a game.
To reduce the search time significantly, I took advantage of Quantik having many symmetries.
Quantik is, in general, invariant with respect to a subset of the [Sudoku symmetries](https://pi.math.cornell.edu/~mec/Summer2009/Mahmood/Symmetry.html).
The corresponding board symmetries include 90-degree board rotations, horizontal reflection, vertical reflection, and swapping the first two or last two rows or columns.
The shape symmetries arise by assigning each unique shape a label and then considering all permutations of those labels.
As a brief aside, one consequence of this is that on the first turn, all of Player 1's moves are equivalent.

{% include figure.html
  id="board_symmetries"
  file="assets/posts/quantik-solution/board_symmetries.svg" 
  width="320px" 
  alt="" 
  caption="Examples of Quantik board symmetries relative to the initial configuration in the top left. Grey cells indicate cells fixed under the permutation, and coloured cells are for visual clarity. More symmetries can be obtained by composing these examples." %}


I can now confidently say that Quantik is *strongly solved*, at least by older definitions from combinatorial game theory.[^1]
One limitation of my solver is that it provides no further information beyond whether a move guarantees a win or a loss under perfect play.
It does not distinguish between winning moves and between losing moves.
One way to resolve this is to calculate with respect to "true" perfect play, where the winning player tries to win in as few moves as possible and the losing player tries to lose in as many moves as possible.
Moves are then distinguished by the number of turns until the guaranteed outcome.
Such a calculation is computationally more expensive, but I am inclined to say it remains computationally tractable; however, I have not attempted to implement this due to a lack of interest.

## A Strategy for Humans - Guaranteed Victory for Player 2

Much like Tic-Tac-Toe, I speculate that Quantik is not super interesting to humans outside of being a mathematical puzzle. In contrast, Quantik is far less known.
However, a few years ago, while implementing my solver, I found a [BoardGameGeek thread discussing strategies in Quantik](https://boardgamegeek.com/thread/2297522/game-solution-space).
Therein, a member by the name of Antonio Carlucci described a simple, elegant strategy for Player 2 that seems to guarantee a win from the start.
I did not inspect it too closely at the time, but long after I moved on from Quantik, I revisited the thread and became interested in rigorously proving that Antonio's strategy secures a win for Player 2 no matter what Player 1 does.

Here is a formalization of Antonio's strategy: consider the strategy $$S^*$$ for Player 2 defined by the following sequential rule.

1. If there is a move that completes a zone, play it.
2. Otherwise, place a shape in *the opposing diagonal cell of the region* that Player 1 placed in their most recent turn, according to the following rule. If Player 1 placed shape $$x$$, respond with shape $$f(x)$$ where

$$
\begin{align*}
f(□) &= ◆ \\
f(◇) &= ■ \\
f(○) &= ▲ \\
f(△) &= ●
\end{align*}
$$

Note that, ignoring colour, the shape labels can be permuted to yield different $$f$$, and in turn different but equivalent strategies.

It is straightforward and tractable to verify by computer that this is a winning strategy for Player 2.
Such verification constitutes a proof, but it does not help us understand *why* the strategy works.
So, why does it?
If you test out the strategy on your own or with a friend, you may observe three consistent properties:

1. The strategy does not violate Quantik's placement rules
2. Player 2 always has a valid move on their turn
3. Player 1 can never play a move that completes a zone

The first property says the strategy is valid (or *well-defined* in mathematical terms): Player 2 never makes an invalid move when following the strategy.
The second and third properties combined imply that Player 2 wins, either by completion or by forcing Player 1 into a turn with no valid moves.
Remember that there are no draws with standard Quantik rules, so Player 1 having no valid moves is a victory for Player 2.

This is a surface-level description of why Antonio's strategy is a perfect play strategy.
It should be noted that the strategy secures a win when used consistently from the first turn, rather than from an arbitrary game state!
Some of you may not be satisfied with my "why" explanation, in which case I have provided a proof sketch below.
The proof is quite ad hoc and not very pretty, so I leave it as optional reading.

{% capture proof_sketch %}
The proof consists of verifying that each of the three properties hold.

**Property 1**.
We can proceed inductively on the number of pieces $$k$$ that Player 2 has placed under the assumption that the game has not ended.

If $$k = 0$$, there is only one shape on the board which is not enough to complete a region. Thus, Player 2 responds to Player 1 with Step 2 of $$S^*$$, which is valid by definition of $$f$$.

Now suppose $$k > 0$$. There are two cases. 
If Player 2 is able to use Step 1 of $$S^*$$, then this is a valid move by definition and we are done.
Otherwise, no winning move is present and Player 2 responds with Step 2.
Without loss of generality, suppose Player 1 last played $$□$$ in the top left region.
This can be done by rotating the board and relabelling the shapes.
Assume by way of contradiction that Player 2 applying Step 2 leads to an invalid move.
By induction, all of Player 2's previous moves are valid and must have followed Step 2.
By definition of Step 2, the cell that Player 2 wants to play on cannot be occupied by any shapes.
If Player 2 cannot place $$f(□) = ◆$$ in the target cell, then by Quantik's placement rules, there exists a $$◇$$ placed earlier by Player 1 on the zone intersecting the target cell.
However, by induction, this means that Player 2 placed a $$■$$ earlier in such way that Player 1's most recent move is invalid.
This is a contradicts our initial assumption.
Therefore, Player 2 applying Step 2 is a valid move.
This completes the proof of Property 1.

**Property 2**. 
We have in fact shown this in the proof of Property 1. Player 2 always has a valid move to respond with as long as the game is not over after Player 1's turn.

**Property 3**. 
Assume for a contradiction that Player 1 placed a shape in a cell that completes a zone.
Without loss of generality, the zone was completed by Player 1 placing $$□$$ in the top left corner cell.
This can be done by composing the [board symmetry permutations](#fig:board_symmetries) and relabelling the shapes.
Since Player 2 has not won by completion, all of their previous placements must have been done in accordance to Step 2 of $$S^*$$.
By definition of Step 2, each region contained either zero, two, or four pieces whenever it was Player 1's turn to place.
This means that Player 1 can never complete a region.
In combination with our initial assumption, this means Player 1 must have completed a row or column.
Without loss of further generality, suppose it is a row.
The subsequent case analysis is tedious and omitted. The resulting contradiction is that Player 2 ends up using Step 2 in an earlier turn when they were able to use Step 1 to secure a win, giving a contradiction.
{% endcapture %}

{% include details.html 
  summary="Proof Sketch - Antonio's Strategy is a Winning Strategy"
  details=proof_sketch
  markdown="1" %}

## Afterthoughts

I am not satisfied with how ad hoc the proof is.
That being said, it does lend itself to a nice idea.
In computational complexity theory, it is sometimes of interest to define and study [generalized games](https://en.wikipedia.org/wiki/Generalized_game).
In this setting, the goal is to quantify how the computational cost of solving the game scales relative to the game's "size".
One canonical way to generalize Quantik to arbitrary grid sizes is to take inspiration from the construction of Generalized Sudoku[^2].
This results in a $$n^2$$-by-$$n^2$$ grid with $$n^2$$ rows, $$n^2$$ columns, and $$n^2$$ regions, where $$n = 2k$$ for a positive integer $$k$$.
Reuse the same rules of Quantik, but adjust the number of shapes and their copies present according to the size parameter $$k$$, and we call this *Generalized Quantik*.
It is not too difficult to adapt Antonio's strategy and the previous proof to show that Player 2 has a winning strategy for Generalized Quantik, starting from an empty board.
Moreover, it can also be verified that the generalized strategy as an *algorithm* scales efficiently! The cost will be a polynomial in $$k$$...

That just about covers my recreational research with Quantik!
There is definitely more to investigate and explore, but for now, I have indefinitely put Quantik aside.
Until next time!


[^1]: I am using the convention in [Louis Victor Allis' doctoral thesis](https://doi.org/10.26481%2Fdis.19940923la). Defined in Section 1.5, a game is *strongly solved* if "for all legal positions, a strategy has been determined to obtain the game-theoretic value of the position, for both players, under reasonable resources."
[^2]: See Section 4.3 of [Yato and Seta's paper](https://globals.ieice.org/en_transactions/fundamentals/10.1587/e86-a_5_1052/_p). An archived copy of the paper is listed in [Sudoku's Wikipedia article](https://en.wikipedia.org/wiki/Sudoku#cite_note-33).