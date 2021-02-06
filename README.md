![screenshort](https://github.com/fastrgv/hbox4/blob/main/t7s.png)

Here is a link to all source code and build files:

https://github.com/fastrgv/hbox4/releases/download/v1.0.0/hbox7feb21.7z



#AdaSokSol -- sokoban solver using Ada

**ver 1.0.0 -- 07feb2021**

* Initial release

## Description

This is a commandline-terminal sokoban solver written in Ada. It is "generic" in the sense that it contains no domain specific strategies. It also provides a demonstration of the incredible power of the Hungarian Algorithm.

## Usage

Sokoban puzzles typically come in files containing groups of puzzles, so the user interface assumes that you provide a file-name, and 2 integers representing the total number of puzzles in the file, and the number of the puzzle to be solved. The solver name is "hbox4".

EG: hbox4 games/Sladkey.sok 50 22

is the command to solve level 22 from the file "Sladkey.sok". The solution is written as a long string in the terminal window at the end of processing. Of course it could be redirected to a file thusly:

EG: hbox4 games/Sladkey.sok 50 22 > soln.txt

-------------------------------------------------------------
In addition to the 3 commandline parameters discussed above, there are 2 more optional ones:

* (4) [float] MaxGb memory to use, with a default of 7.5 Gb.
* (5) [int] Solution method: 0 (default) implies fastest, 1 implies push-optimal+best-moves

EG: hbox4 games/Sladkey.sok 50 22 6.5 1

indicates using no more than 6.5 Gb of memory and a push-optimal solution method.

EG: hbox4 games/Sladkey.sok 50 22

indicates using the defaults, i.e. 7.5Gb memory and fastest solution.

There are many puzzles this algorithm will not solve due to memory limits, so the embedded memory limiter will exit gracefully when memory usage exceeds the preset limit. 

Finally, if you don't want to wait for the solver to finish, you can <ctrl>-c out of it to quit immediately.


## Algorithm Used

A multiple-heuristic, single-step box search, done in reverse, using four "orthogonal" priority measures [heuristics] in a round-robin sequence. The Hungarian Algorithm is used to match boxes with goals and to generate an estimate of future box moves to solution.

An article by Frank Takes shows advantages to working from a solved position backwards to the start position. This prevents box-deadlocks from taking up space in the search tree. Thusly, the formidable issue of deadlock avoidance is completely ignored. Likewise, the tricky issue of endgame goal-packing-order is sidestepped, as well.

Self balancing splaytrees are used to test whether a given configuration was seen before. There are also 4 priority queues embedded into the splaytrees that provide distinct "views" of the data.

The 4 priority measures were adapted from the "Festival" algorithm description, but simplified to allow rapid local evaluations:

* pri1: Nboxes - NboxesOnGoals		0 means all boxes on goals
* pri2: Ncorrals - 1						0 means only 1 corral, i.e. pusher is free to roam
* pri3: NblockedRooms					0 means no boxes block openings
* pri4: NblockedBoxes					0 means all boxes accessible to pusher

These priority measures are typically small non-negative integers to be minimized. So it often occurs that there are many candidate nodes of the search-tree with the same measure. To help distinguish those that are more promising, a secondary measure is used that counts the total box-pulls to reach the current configuration, PLUS the Hungarian-Algorithm-based estimate of the remaining number of pulls.

Finally, the round robin regimen, that includes all 4 measures, is most effective in the beginning when there are typically blocked rooms, and many corrals. In the later stages, 3 priority measures are dropped to use only #1 because, once again, there may typically be blocked rooms and corrals that must be allowed near endgame. Recall that we are working backwards.

## What's so great about this app?

This is only a moderately capable sokoban solver (solving 32 of the original 90). What makes it interesting? In a world with extremely capable solvers like Sokolution and Festival, why look at this one? Why hunt with an algorithmic crossbow instead of a gun? Simplicity, elegance.

Hbox4:
* contains no domain-specific specializations or strategies.
* uses algorithms and data structures of general interest and usefullness.
* wonderfully demonstrates the remarkable power of the Hungarian Algorithm.

It also adds further support to the design choices made in the Festival solver that proposed these 4 "orthogonal" heuristic measures or, so called "features".


### SplayTree-Priority-Queue
The splaytree [self-balancing-binary-tree] based priority queues allow unique hash keys to be inserted and found quickly, with 4 embedded priority queues that allows efficient insertions, access, and deletion from the heads [popping]. The hash keys uniquely identify box-layouts at each saved node, to avoid duplicates. The 4 priority queue orderings allow primary and secondary (tie-breaker) priority measures.

### Dynamic Programming [flood-fill]
Dynamic programming allows efficient determination of box-valid locations and the feasibility and cost of traversing between two locations. This information is used to feed into the Hungarian Algorithm.

### Hungarian Matching Algorithm
This fully functional implementation of the Hungarian algorithm provides valid, one-to-one pairings between boxes and goals for each intermediate box-layout, and provides realistic estimates [admissible & consistent] of the number of box-moves remaining to completion. The incredible power of this heuristic can be seen when it is swapped out; i.e. when the secondary priority measure is changed from (moves + estimated-future-moves) to just (moves), the solver is still functional but becomes severely disabled versus a large class of puzzles.

* admissible means: heuristic(x,goal) <= actualCost(x,goal)
* consistent[monotone] means: h(x,g) <= actualCost(x,y)+h(y,g)

Note that consistent => admissible assuming h(g,g)=0.


### Simplicity & Generality
These qualities result from a minimalistic regimen that avoids:
* complex control mechanisms, high or low level, abstract or detailed;
* domain-specific improvements, tactics;
* databases;
* macro-moves, tunnel-macros, goal-macros;
* matching specific box-pattern-templates;

For the C++ programmer this Ada code is written in a transparent style and should be easy to comprehend; and for the experienced Ada programmer there are many improvements to be made to better utilize the advanced protections and features of the Ada language.  


## Shortcomings

Disclaimer #1: the elegance lies in the algorithms, not the code.

Disclaimer #2: even using the option for a more efficient solution, rather than the quickest, there are strange moves produced that are clearly wasteful. But given the inherent complexities of solving a sokoban puzzle, it is quite possible these wasteful moves are caused by algorithmic or programming oversights. Still, the solver is somewhat surprising in its capability. I have been trying for years to solve level #2 of the original 50 with the usual breadth-first search methods, but "hbox4" solves it easily. Another surprise is that it solves takaken #7 ("love") in 3 seconds, while the excellent takaken solver that I downloaded takes 27 seconds [on the same computer]. And assuming I ran it correctly, Festival took 283 seconds!

In any case, I wish to expose this algorithm to public scrutiny, and allow anyone with an interest, the chance to improve or extend this generic approach to the branch of artificial intelligence that addresses the formidable task of solving sokoban puzzles.



--------------------------
## License:


This app is covered by the GNU GPL v3 as indicated in the sources:


Copyright (C) 2021  <fastrgv@gmail.com>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You may read the full text of the GNU General Public License
at <http://www.gnu.org/licenses/>.


