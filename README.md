![screenshort](https://github.com/fastrgv/hbox4/blob/main/t7s.png)

Here is a link to all source code and build files:

https://github.com/fastrgv/hbox4/releases/download/v1.0.2/hbox12feb21.7z





# hbox4 -- sokoban solver using Ada


**ver 1.0.2 -- 12feb2021**

* Code cleanup;
* Replaced splaylist with simpler splaytree;
* Included both 32-bit and 64-bit Windows executables;


**ver 1.0.1 -- 10feb2021**

* added a 3rd solution method option to omit the Hungarian estimator.
* now deliver a Windows executable [runs fine under Wine]

**ver 1.0.0 -- 08feb2021**

* Initial release

## Description

This is a commandline-terminal sokoban solver written in Ada. It is "generic" in the sense that it contains no domain specific strategies. It also provides a demonstration of the advantage in using the Hungarian Algorithm.

## Usage

Sokoban puzzles typically come in files containing groups of puzzles, so the user interface assumes that you provide a file-name, and 2 integers representing the total number of puzzles in the file, and the number of the puzzle to be solved. The solver name is "hbox4".

EG: hbox4 games/Sladkey.sok 50 22

is the command to solve level 22 from the file "Sladkey.sok". The solution is written as a long string in the terminal window at the end of processing. Of course it could be redirected to a file thusly:

EG: hbox4 games/Sladkey.sok 50 22 > soln.txt

-------------------------------------------------------------
In addition to the 3 mandatory commandline parameters discussed above, there are 2 more optional ones:

* (4) [float] MaxGb memory to use, with a default of 7.5 Gb.
* (5) [int] Solution method: 
	* 0 (default) quickest solution
	* 1 more efficient solution [the goal, not always the reality]
	* 2 No Hungarian Estimator: generally more efficient solutions, sometimes faster.

EG: hbox4 games/Sladkey.sok 50 22 6.5 1

indicates using no more than 6.5 Gb of memory and the efficient solution method. But note that the solutions are not always efficient. This name merely refers to certain algorithmic choices within the solver which generally tend to produce solutions with fewer moves.

EG: hbox4 games/Sladkey.sok 50 22

indicates using the defaults, i.e. 7.5Gb memory and fastest solution.

There are many puzzles this algorithm will not solve due to memory limits, so the embedded memory limiter will exit gracefully when memory usage exceeds the preset limit. 

Finally, if you don't want to wait for the solver to finish, you can <ctrl>-c out of it to quit immediately.


## Algorithm Used

A multiple-heuristic, single-step box search, done in reverse, using four "orthogonal" priority measures [heuristics] in a round-robin sequence. The Hungarian Algorithm is used to match boxes with goals and to generate an estimate of future box moves to solution.

An article by Frank Takes shows advantages to working from a solved position backwards to the start position. This prevents box-deadlocks from taking up space in the search tree. Thusly, the formidable issue of deadlock avoidance is completely ignored. Likewise, the tricky issue of endgame goal-packing-order is sidestepped, as well.

Self balancing splaytrees are used to test whether a given configuration was seen before. There are also 4 priority queues embedded into the splaytrees that provide distinct "views" of the data.

The 4 priority measures were adapted from the "Festival" algorithm description, but simplified to allow rapid local evaluations:

* pri1: Nboxes - NboxesOnGoals...............0 means all boxes on goals
* pri2: Ncorrals - 1				...............0 means only 1 corral, i.e. pusher is free to roam
* pri3: NblockedRooms			...............0 means no boxes block openings
* pri4: NblockedBoxes			...............0 means all boxes accessible to pusher

These 4[primary] priority measures are typically small non-negative integers to be minimized. So it often occurs that there are many candidate nodes of the search-tree with the same measure. To help distinguish those that are more promising, a secondary measure is used that counts the total box-pulls to reach the current configuration, PLUS the Hungarian-Algorithm-based estimate of the remaining number of pulls. 

Finally, the round robin regimen that includes all 4 measures eventually drops the last 3 measures sometime before the endgame since corrals and blocked rooms must be permitted at the end of the reverse game, i.e. near the beginning of a forward game, and because the evaluation of the measures is costly.

### Additional algorithmic details

A nodal configuration description consists of the box layout, the upperleft corner of the puller-corral, as well as the total box pulls, the hungarian estimate of the future box pulls, and the 4 [primary] priority measures.

Conceptually, the Splaytree Priority Queue {frontier} contains 4 independent queues, each ordered by one of the 4 priority measures mentioned above. So if the round-robin parameter is K in {0,1,2,3}, we greedily pop the leading candidate node off of the front of queue #K, thereby deleting it from {frontier}.

So, in this SingleStepBoxSearch the initial configuration is read, evaluated and pushed into the {frontier} queue. But goal and boxes are interchanged because this is a backward search using a "puller" rather than a "pusher".

So here begins the main loop:-------------------------------------------

Choose "K", and pop the frontrunner off Kth queue and look at its nodal information to retrieve the hashkey. This key allows direct access via the splaytree structure that, in turn allows direct removal of the node from the splaytree and the other 3 PriQueues. Thusly, the removals from the other 3 PriQueues do not require any time consuming list-traversals.

This removed configuration is then tested to see whether a solution has been reached. If not, it is inserted into the {explored} splaytree, which has no queue structures attached. The splaytree allows rapid insertion and rapid random retrieval in case we find a solution and need to reconstruct our path. 

For this current configuration we simply cycle through each [unsorted] box and try to move it one step in each direction. If the move is successful, its 4 priority measures are evaluated and it is enqueued into {frontier}. This involves a splatree insertion and 4 priority-queue insertions that each maintain a separate ordering. (It was fairly tricky to get these priority-queue operations to be efficient enough for use in a sokoban solver.)

End of main loop.------------------------------------------------------


## What's so great about this app?

This is only a moderately capable sokoban solver (solving 32 of the original 90). What makes it interesting? In a world with extremely capable solvers like Sokolution and Festival, why look at this one? Why hunt with an algorithmic crossbow instead of a gun? Simplicity, elegance. Hbox4:

* contains no domain-specific specializations or strategies [other than the 4 priority measures]

* uses algorithms and data structures of general interest and usefullness.

* demonstrates the considerable power of the Hungarian Algorithm.

* easily buildable on Windows, OSX, and Linux using AdaCore's GNAT compiler.

It also adds further support to the effectiveness of the design choices made in the Festival solver that proposed these 4 "orthogonal" heuristic measures or, so called "features".


### SplayTree-Priority-Queue
The splaytree [self-balancing-binary-tree] based priority queues allow unique hash keys to be inserted, found & removed quickly, with 4 embedded priority queues that allows efficient insertions, access, and deletion, both from the heads [popping], and directly by key. The hash keys uniquely identify pusher & box-layouts at each saved node, to avoid duplicates. The 4 priority queue orderings allow primary and secondary (tie-breaker) priority measures. As mentioned above, the structure also allows rapid, direct access deletions given the hashkey [O(log n)].

The only expensive [O(n)] operation is finding the head of each queue. That involves a lexicographical search through a 2 dimensional array of pointers to find the first non-null pointer. All other queue operations are done in constant time [O(1)], i.e. independent of the number of queue entries.

### Dynamic Programming [flood-fill]
Dynamic programming allows efficient determination of box-valid locations and the feasibility and cost of traversing between two locations. This information is used to feed into the Hungarian Algorithm.

### Hungarian Matching Algorithm
This fully functional implementation of the Hungarian algorithm provides valid, one-to-one pairings between boxes and goals for each intermediate box-layout, and provides realistic estimates [admissible & consistent] of the number of box-moves remaining to completion. The power of this heuristic can be seen when it is swapped out; i.e. when the secondary priority measure is changed from (moves + estimated-future-moves) to just (moves), the solver is still functional, and even faster on a some puzzles, but is significantly impaired on many others.

* admissible means: heuristic(x,goal) <= actualCost(x,goal)
* consistent[monotone] means: h(x,g) <= actualCost(x,y)+h(y,g)

Note that consistent => admissible assuming h(g,g)=0.

One may choose to omit the Hungarian estimator so that the secondary [tie-breaking] priority measure then becomes simply the total box moves used to arrive at a given configuration.


### Simplicity & Generality
These qualities result from a minimalistic regimen that AVOIDS:

* complex control mechanisms, high or low level, abstract or detailed;
* domain-specific improvements, tactics;
* databases;
* macro-moves, tunnel-macros, goal-macros;
* matching specific box-pattern-templates;

For the C++ programmer this Ada code is written in a transparent style and should be easy to comprehend; and for the experienced Ada programmer there are many improvements to be made to better utilize the advanced protections and features of the Ada language.  


## Shortcomings

Disclaimer #1: the elegance lies in the algorithms, not the code.

Disclaimer #2: No attempt at solution optimality is made. The goal in this solver is to find any solution. The 4 orthogonal "features" do not lend themselves to finding solutions with any type of optimality.

Disclaimer #3: even using the option for a more efficient solution, rather than the quickest, there are strange moves produced that are clearly wasteful. Still, the solver is somewhat surprising in its capability. I have been trying for years to solve level #2 of the original 50 with the usual breadth-first search methods, but "hbox4" solves it easily. Another surprise is that it solves takaken #7 ("love") in 3 seconds, while the excellent takaken solver that I downloaded takes 27 seconds [on the same computer]. And, assuming I ran it correctly, Festival took 283 seconds!

In any case, I wish to expose this algorithm to public scrutiny, and allow anyone with an interest, the chance to improve or extend this generic approach to the branch of artificial intelligence that addresses the formidable task of solving sokoban puzzles.


## Xsokoban Levels Solved:

1 2 3 4 5 6 7 8 9 12 15 17 29
33 38 44 49 53 54 56 58 60
62 68 71 78 79 81 83 84 86 87

hbox4 is extravagant with memory; all failures
I have seen are due to shortage of memory, & lack of progress.



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

---------------------------------------------
keywords:
hungarian, ada, munkres, kuhn, kuhn-munkres,
puzzle, sokoban, solver


