Introduction to My Approach:
----------------------------

For my solution, I have implemented Dancing Links (Donald Knuth, 2000, Dancing Links) to solve sudokus as exact cover problems. I chose this approach
as it is a very efficient way of implementing a fast backtracking algorithm, which sudoku is very well suited for, as sudoku can be thought of as a search problem. 

An exact cover problem is a type of constraint satisfaction problem, represented by a 2D matrix of 1's and 0's, from which a subset of rows
are selected, such that placing correspondin 1's over 0's yields a row of only 1's.
e.g. [1,1,0,0] and [0,0,1,1]

A sudoku grid can be translated into this sort of matrix by represnting each state of a cell and it's effects of each constraint. 

Below is an extratct of a visualisation of this:


"This version uses digits 1-9 simply for easier reading" (Note in reality, all 1-9 are just 1's in the matrix, and all blank spaces are 0's)

       cell constraints (only one of value in each of 81 cells)------------------------- row constraints (only one of 1-9 in each of 9 rows)------------------------------ column constraints (only one of 1-9 in each of 9 columns------------------------- block constraints (only one of 1-9 in each of 9 blocks)--------------------------
       0        1         2         3         4         5         6         7         8  1        2        3        4        5        6        7        8        9         1        2        3        4        5        6        7        8        9         1        2        3        4        5        6        7        8        9        
       123456789012345678901234567890123456789012345678901234567890123456789012345678901 123456789123456789123456789123456789123456789123456789123456789123456789123456789 123456789123456789123456789123456789123456789123456789123456789123456789123456789 123456789123456789123456789123456789123456789123456789123456789123456789123456789

cand/N 999999999999999999999999999999999999999999999999999999999999999999999999999999999 999999999999999999999999999999999999999999999999999999999999999999999999999999999 999999999999999999999999999999999999999999999999999999999999999999999999999999999 999999999999999999999999999999999999999999999999999999999999999999999999999999999 
r1c1#1 1                                                                                |1                                                                                |1                                                                                |1                                                                                |
r1c1#2 2                                                                                | 2                                                                               | 2                                                                               | 2                                                                               |
r1c1#3 3                                                                                |  3                                                                              |  3                                                                              |  3                                                                              |
r1c1#4 4                                                                                |   4                                                                             |   4                                                                             |   4                                                                             |
r1c1#5 5                                                                                |    5                                                                            |    5                                                                            |    5                                                                            |
r1c1#6 6                                                                                |     6                                                                           |     6                                                                           |     6                                                                           |
r1c1#7 7                                                                                |      7                                                                          |      7                                                                          |      7                                                                          |
r1c1#8 8                                                                                |       8                                                                         |       8                                                                         |       8                                                                         |
r1c1#9 9                                                                                |        9                                                                        |        9                                                                        |        9                                                                        |
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
r1c2#1  1                                                                               |1                                                                                |         1                                                                       |1                                                                                |
r1c2#2  2                                                                               | 2                                                                               |          2                                                                      | 2                                                                               |
r1c2#3  3                                                                               |  3                                                                              |           3                                                                     |  3                                                                              |
r1c2#4  4                                                                               |   4                                                                             |            4                                                                    |   4                                                                             |
r1c2#5  5                                                                               |    5                                                                            |             5                                                                   |    5                                                                            |
r1c2#6  6                                                                               |     6                                                                           |              6                                                                  |     6                                                                           |
r1c2#7  7                                                                               |      7                                                                          |               7                                                                 |      7                                                                          |
r1c2#8  8                                                                               |       8                                                                         |                8                                                                |       8                                                                         |
r1c2#9  9                                                                               |        9                                                                        |                 9                                                               |        9                                                                        |
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
...
...
...


(https://www.stolaf.edu/people/hansonr/sudoku/exactcovermatrix.htm)

Method and Decisions Breakdown:
-------------------------------

Note: This section discusses my overall approaches and ideas - specific explanaition of code can be found in the commenting on the actual code file.

I wanted to keep my code modular and re-usable, so I decided to solve the exact cover problem first, then attempted to use it to implement a sudoku solver.
This was so I can re-purpose my code for future projects which can be turned into exact cover problems. 

I decided to test my initial solution with very simple data like below:
[[0,1,0],
[1,1,0],
[1,0,1]]
as there is no reason to use large complex data like the exact cover matrix for sudoku; the process is the same, but it is much easier to see for this data set that
the final output is [0,2] because combining rows at these indeces gives a row of [1,1,1].

I decided to implement the data structure first, as the actual algorithm relies on this to be functioning before it can be tested:

	The first class implements the core data structure as set out in Knuth's paper - a 4 way doubley linked list. After playing around with a procedural approach, 
	it was clear that an OOP approach was much more suited to this problem, as each node has attributes (the current pointer values), 
	and methods (to update the values of these pointers).

	The second class implements the column objects. These objects have very similar properties to regular nodes in the linked list, i.e. they have pointers that need to be updated. 
	They also have additional attributes like size and name, to help keep track of the order of the data structure (i.e. which column comes first), 
	and additional methods such as covering and un-covering. Therefore I decided to have this class inherit from dancing node, then add the extra behaviours as described.

Once I had implemented the data structure, I next implemented the actual algorithm. Again, I tried around with a procedural approach but decided that wrapping the algorithm in a class was a more elegant implementation, 
as there are several steps that must be completed before and after the actual solving step happens, that could be implemented as methods on an algorithm object, which has attributes
like solution and header node. Having it as a class also makes it easier to reference all the different variables related to the algorithm without the need for messy globals:
	 
	The constructor generates all the nodes and column nodes used in the data structure, and initialses all the pointers to create the 4-way doubley linked list.

	The solve method relies on choosing the next column to be covered. The order of this doesn't matter in terms of affecting consistency of the algorithm, but the way in which they are chosen 
	can improve performance. Since my test data for the algorithm was quite small and I just wanted to get it working, I began by implementing choose_column_object which simply 
	returns the next column object.
	
	I added choose_column_object_efficient when I had the algorithm working, as otherwise there was no way to test that the imporvement actually helped. It also meant I could debug 
	solve individually. This was a good idea as when I did add the improvement, I had an error which took me a long time to spot which may have led me to think the actual solve 
	algorithm wasn't working correctly. The improvement reduces the branching factor by choosing the column with the fewest nodes. 
	Originally, I thought it was slowing perfomance due to the overhead of finding the column with the fewest nodes, but in fact whilst this was partially true for simpler problems which didn't require many branches anyway, 
	the harder puzzles were reduced from minuites to under a second; from thousands of branches to hundreds.

Since algorithm x is modular to be re-purposed later, I have also included 2 functions for turning 2D arrays a) from sudoku to exact cover, and b) from exact cover to sudoku, 1 for input to the algorithm, 
and the other for outputting my solutions for sudokus, to create the implementation for a sudoku solver:
	
	The first function has two parts:
		
		1. Create the entire exact cover matrix for sudoku
		2. Remove the rows that don't need to be checked if they conflict with a clue in the sudoku puzzle
	
	This function is likely the most inefficient part of the code and would be the first to be refactored in the future, when devlopment time is not a limitation.
	I did try writing the matrix from 1. to a file then reading it for 2. but this actually seemed to take longer than just creating it for each puzzle.

	The second function translates the result from solve into a sudoku grid. It does this by performing calculations on the row column information of each cell
	in the solution, and how it corresponds to the constraint matrix index (see the first column in the exctract).

Potential Future Improvements:
--------------------

The final performance of the algorithm is very good - it can solve all of the test sudoku's (including the hard ones) in under a second accorsing to my own tests.
However, it takes longer for shorter puzzles than perhaps other solutions - most puzzles actually take about the same amount of time. This is because most of the time is spent setting up 
the data structure and cover matrix. One solution to this may be to implement a different algorithm for easier puzzles with more clues, but switch back to this on for harder ones.

It also follows that speeding up the creation of this data structure will lead to great imporvements. I believe this can be achieved by re-assessing the amount of iteration needed to set 
up the nodes, perhaps by calculating the number of nodes needed per column before creating them. A similiar approach may be used for the creation of the exact cover matrix





