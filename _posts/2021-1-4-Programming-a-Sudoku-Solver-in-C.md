---
layout: post
tags: project 
title: Programming a Sudoku Solver in C
---


# Introduction

[GitHub Repo](https://github.com/kmg731/Sudoku-Solver-in-C)

Over the summer I took a freshman CS class at Harvard to make up for some lost time.  One of the most fun
projects we had to make was a sudoku solver.  At the time, it took a few days of tinkering and fumbling around
to really understand what I was doing but once I sorted it all out, I found it wasn't that difficult.  This project
was originally written in Java (with much of the secondary code being written by the professor) so part of my work
was also moving everything over to C and making everything work.  There were a few little C problems that I had 
but overall the project was not as difficult as I originally thought it would be. 

Making a sudoku solver is a fun way to quickly get a grasp on recursive backtracking while also building something
quite fun and practical.  


# Recursive Backtracking

Recursive backtracking is a simple but powerful way to solve a problem which requires some set of elements 
to be placed according to some set of rules.  

The classic recursive backtracking problem is the N-Queens problem, in which a computer is given a chess board and told
to place N number of queens on the board so that they're all safe.  The constraints are the movement rules of chess 
and the computer has to place some number of them on a board.  Recursive backtracking is a way of using computers to 
(kind of) brute force these problems.  Rather than solve the problem using logical deduction like a human would, 
the computer is able to try out most options until it finds one that works.  You could do this yourself for a sudoku
puzzle but it would take a very long time.  

The general algorithm goes like this:

    1. Try the first option in the set
    2. If it works, go to the next option
    3. If it doesn't work, go back to the last
       option that did work. 

You could imagine doing this with a puzzle yourself.  You start at the first square, run through 1-9 and find the first
safe option.  Then you would go to the next square and try it again.  Eventually you would hit a square where none of the
numbers work. You'd instead have to go back to the last safe square and repeat the process, then advance from there and 
repeat the process until all the numbers are filled in.  


# The Code

Now it's time to dig into the actual implementation of the Sudoku Solver.  I don't think this was all that difficult to 
get working but I'll go over the bugs I ran into when we hit that part. 

    typedef struct
    puzzle_t
    {
            int values[DIM][DIM];
            bool isFixed[DIM][DIM];
            bool subgridHasVal[SUBGRID_DIM][SUBGRID_DIM][DIM];
            bool colHasVal[DIM][DIM];
            bool rowHasVal[DIM][DIM];
    
    }Puzzle;

The program is built around a `Puzzle` struct, which contains a bunch of different `boolean` arrays and one `int` array.
The boolean arrays allow us to check what values are set and which are safe to place down on any given square.  
`isFixedp[][]` checks for numbers which were given before we started the puzzle.  `subgridHasVal[][][]` checks each 
3x3 subgrid in the puzzle for a certain value.  colHasVal and rowHasVal both just check a column or row for a certain
value.  

    Puzzle*
    loadPuzzle(char* fileName)
    {
    
            Puzzle* p = (Puzzle*)calloc(1, sizeof(Puzzle));
            if (p == NULL){
                    printf("ERROR: Could not allocate memory for puzzle P\n");
                    exit(-1);
            }
    
            FILE *fp;
            fp = fopen(fileName, "r");
    
            if (fp == NULL){
                    printf("ERROR: File %s could not be opened\n", fileName);
                    exit(-1);
            }
    
            int puzzleVal;
            int i = 0;
    
            while((puzzleVal = fgetc(fp)) != EOF){
                    if (puzzleVal >= 48 && puzzleVal <=57){
                            int row = i / 9;
                            int col = i % 9;
                            int val = puzzleVal - 48;
    
                            p->values[row][col] = val;
    
                            if(val > 0){
                                    p->isFixed[row][col]   = true;
                                    p->rowHasVal[row][val] = true;
                                    p->colHasVal[col][val] = true;
                                    p->subgridHasVal[row / SUBGRID_DIM][col / SUBGRID_DIM][val] = true;
                                    }
    
                            ++i;
                    } 
            }
    
    
            fclose(fp);
            return p;
    }

The program starts by `calloc()`-ing a new Puzzle struct.  Originally, I was using `malloc()` and a whole lot of
ugly `memset()` calls.  Though I soon realized that `calloc()` makes the code a lot cleaner (and my life much easier).
It runs through the appropriate error handling for `calloc()` and `fopen`, then goes into a loop which reads all the 
values into the arrays and sets the proper truth values for each square in the puzzle.  Since blank spaces are zero 
values, we want to make sure none of those are in our boolean arrays. 

With that out of the way, we can get to the main recursive backtracking function.

    int
    solvePuzzle(Puzzle* p, int n){
    
            if (n == 81){
                    return true;
            }
    
            int row = n / 9;
            int col = n % 9;
    
            if (p->isFixed[row][col]){
                    if(solvePuzzle(p, n + 1)){
                            return true;
                    }
            }
    
            for (int val = 1; val <= 9; ++val){
                    if (isSafe(p, val, row, col)){
                            placeVal(p, val, row, col);
    
                            if (solvePuzzle(p, n + 1)){
                                    return true;
                            }
    
                            removeVal(p, val, row, col);
                    }
            }
            return false;
    }

The base case for this recursive function is when `n` `= 81, since the =n` value is telling us which square we're
currently on.  The function then calculates the current row and column value from that `n` value to be used in the 
rest of the function.  

The next conditional block checks if the current value is fixed.  If it is fixed, we can just go to the next value, 
since there is nothing more we can do to a fixed value. 

The for loop runs through each of the possible values and checks if they're safe.  If it is safe, the function 
places the value and recurses.  If the value is not safe, `solvePuzzle` will return false, remove the value and 
move onto the next value in the for loop.

The recursive backtracking happens with the `return false` outside of the for loop.  If all possible values have been
tried without the program recursing, the "square" will return false and move back to where the previous square left
off, meaning that one of the values placed previously cannot work.  


# Difficulties

There were a few bugs I ran into that I feel I should mention.  The first and most major bug came when I couldn't do
math.  When I was originally using `memset()` for initializing the struct, I think I had initialized `subgridHasValue`
to 36 instead of 81.  This led to only part of the array being initialized to 0 and messing up a lot of numbers. This is
the best explanation I have for why it didn't originally work.  Originally, it would fail after solving the first row but
then, after an hour or so of just working on other parts of the program, I ran it again and it worked perfectly.  

I also originally thought I'd have to wrestle around with some array numbers but found that there were no problems
just leaving it as it is.  

One common bug I ran into this time and when I originally did this was that the recursion would end too soon.  It would
get a bit in and then just return.  This is usually caused by a boolean array being off, either the correct number is 
being marked as incorrect or a fixed number is being treated as non-fixed.  Check over everything and see where the 
program is ending.


# Conclusion

I'm most surprised by the performance of this program.  The original implementation in Java ran a bit slow but this 
one prints the result just as soon as you've pressed the enter key.  While it's not an extremely practical tool, it's
something fun that makes you feel a bit like you're gaming the system.

The only thing I'm not entirely happy with is having to type the puzzle into a blank text file, though the solution
to this would be to add in some kind of computer vision and I'm not really prepared to add that in (maybe in the future
though).  

