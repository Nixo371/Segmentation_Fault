---
title: Why my sudoku solver sucks.
date: 2025-07-12
draft: false
author: Nicolas Ucieda
---
I wanted to make a sudoku solver using backtracking, a project only slightly less common than a todo app. I was genuinely expecting it to take an afternoon and I could be done with it, but as all my projects go, there were a lot of setbacks. Even now, it's not as good as I want it to be, and I'll keep iterating on it. For now let's say this is
## Sudoku Solver v1

Sudoku via backtracking is super simple to understand. This is the algorithm:
1. Guess a number to put in the first cell
2. Check if you've broken any sudoku rules (same number in the same row, column, or region). If so, guess a different number
3. Move to the next cell
4. Repeat until you've solved the whole grid
5. If you ever run out of numbers for a cell, you have to change your guess of the previous cell, and so on recursively

# Setbacks
I need to pick a format for inputting and outputting a sudoku, one that is easy to parse and readily available. On my search for a puzzle generator I found [QQWings](https://qqwing.com/generate.html) which had this format:
```
......162...12....5.....3.....6..51....5872.6.......3.6...3.7....2....58..89.6...
```
Simple enough, I downloaded the command line version and created 3 files, each one having 1000 generated puzzles of easy, intermediate, and expert difficulties.

Then I ran my program against the easy file and waited. I made a function to print out the current state of the puzzle in a human readable way so I could see what my algorithm was doing:
```
-------------------------------
| 3  4  8 | 7  1  5 | 9  2  6 |
| 1  6  2 | 9  8  4 | 3  5  7 |
| 5  9  7 | 2  6  3 | 4  8  1 |
-------------------------------
| 8  7  5 | 6  4    |    9  3 |
| 6       | 5  3    |         |
| 2     9 | 8     1 |    7    |
-------------------------------
|         |       8 |         |
| 9       | 1     2 | 5  3    |
|       1 |       6 | 2       |
-------------------------------
```
Which led me to quickly realizing that it was going to take forever, as it had taken about 10 minutes to solve the first 30 puzzles. Ctrl+C and back to the drawing board.

# Turns out profilers aren't just to show off
I ran a bunch of easy diagnostics like printing the times that each puzzle took and immediately noticed that there were puzzles that took less than a second and others that took way way longer (shout out to one that took ~1800 seconds). If you're curious, this is it:
```
.....27.8....1.3.........................5..26..3.71...8.19..2.5.3.26.....15...97
```

The backtracking approach is fastest when it has to do most of the guessing near the end, where it can quickly eliminate wrong guesses. This puzzle is an outlier in that most of the clues are at the end, meaning a lot of wrong guesses happen so early that it takes millions of iterations to change those guesses.

I also wrote a Profiler class so I could see how long the validation function took in comparison to the rest (I hate C++ templates so much), turns out it accounts for >90% of solving time.

Clearly the issue lies in that my `check_valid` function is too slow, so I need to speed it up in any way possible. One immediate error I spotted is that the function checks the whole puzzle for errors. The way I've implemented my solver it only needs to check 1 row, column, and region. I modified the function to take a `row` and `column` variable so that it only checks the corresponding areas. This one change sped the solving process up five fold.

That puzzle that took ~1800 seconds? Now it only takes ~360.

# That still sucks
6 minutes... for an **easy** sudoku puzzle. There are *humans* that can do them in under a minute. Check out [Fastest SuDoku Solver](https://youtu.be/IPfsrt58Aow)'s 41 second solve, it's insane.

I took the first 100 easy puzzles and ran them through my solver for a total time of 466.645 seconds (average of 3 runs). And the infamous puzzle takes ~335 seconds of that, nearly 72% of the total runtime for a ***single*** puzzle.

I had our AI overlords make a Python script to make some matplotlib graphs of the puzzle times:
![Image Description](/images/Pasted%20image%2020250713001228.png)
The main columns show the total time the validation function is run, and the remaining darker column is the total time for each puzzle. While it's clear that it's an outlier (4.611s average time including puzzle #74, 1.3085s without), I still think my program should be able to handle it in a reasonable amount of time.

So clearly there's more work to be done, and I already have some ideas that I'll implement and test with.

One is instead of running through all 9 numbers, have each cell store an array of possible values and eliminate possibilities from given clues. This, if done properly,  would actually allow for a complete removal of a validation function, since having an "empty" array would be the indication that a mistake was made.

Here's the project on [GitHub](https://github.com/Nixo371/SudokuSolver/tree/master) if you want to try it out yourself. I won't merge any PRs but I definitely will read and comment on them :)
