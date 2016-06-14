# Bitboards and Connect Four

[Fhourstones](https://en.wikipedia.org/wiki/Fhourstones) is a benchmark that is based on a highly efficient
[implementation](http://tromp.github.io/c4/fhour.html) of  the game [Connect Four](https://en.wikipedia.org/wiki/Connect_Four).
Its author, [John Tromp](http://tromp.github.io/), uses two bitboards to represent the board from each player's
point of view. A [bitboard](https://en.wikipedia.org/wiki/Bitboard) encodes the current position as a series of ones and zeros.
The encoding is designed for speed in operating with the board. Especially querying the board for four in a row is
extremely fast.

I figured that my students struggle with understanding John's implementation. That's why I created this document.
My explanation leans on John's code. However, we don't go through John's code line by line. Instead, I try to
help you get the point of the encoding and the way using it. Equipped with this understanding you should figure out
the details of John's implementation yourself.

Enjoy,

Dominikus Herzberg, [@denkspuren](https://twitter.com/denkspuren) 

## A Binary Board Encoding for "Four in a Row"

In Java and in many other programming languages, a so-called long integer is composed of eight bytes.
Eight bytes are 64 bits (1 byte = 8 bits). If presented as a series of ones and zeros, the
most significant bit (MSB) is on the left hand side, the least significant bit (LSB) is on the right hand side.
That's why we number the bits in the way shown below from right to left. Remember, programmers start counting
with zero ;-)

~~~
63 62 61 60 ... 48 47 46 45 ... 3 2 1 0
~~~

Connect Four is played on a vertical board with seven columns and six rows. That makes 42 slots. We add an
additional row on top for convenience purposes. This additional row is for computational reasons only.

The slots are numbered from 0 to 48. The numbers indicate the position in the bit representation of a long integer.

~~~
  6 13 20 27 34 41 48    Additional row
+---------------------+ 
| 5 12 19 26 33 40 47 |  top row
| 4 11 18 25 32 39 46 |
| 3 10 17 24 31 38 45 |
| 2  9 16 23 30 37 44 |
| 1  8 15 22 29 36 43 |
| 0  7 14 21 28 35 42 |  bottom row
+---------------------+
~~~

Let's take an example, given the following position:

~~~
. . . . . . .
. . . . . . .
. . . . . . .
. . . O . . .
. . . X X . .
. . O X O . .
-------------
0 1 2 3 4 5 6  colum (counting starts with 0 from left to right)
~~~

Connect Four is played by two players. There is one bitboard for each player.
We encode the position separately for each player. One bitboard encodes the
position with disks `X` only, the other bitboard the position with disks `O` only.

~~~
               0 0 0 0 0 0 0  0 0 0 0 0 0 0   6 13 20 27 34 41 48
. . . . . . .  0 0 0 0 0 0 0  0 0 0 0 0 0 0   5 12 19 26 33 40 47
. . . . . . .  0 0 0 0 0 0 0  0 0 0 0 0 0 0   4 11 18 25 32 39 46
. . . . . . .  0 0 0 0 0 0 0  0 0 0 0 0 0 0   3 10 17 24 31 38 45
. . . O . . .  0 0 0 0 0 0 0  0 0 0 1 0 0 0   2  9 16 23 30 37 44
. . . X X . .  0 0 0 1 1 0 0  0 0 0 0 0 0 0   1  8 15 22 29 36 43
. . O X O . .  0 0 0 1 0 0 0  0 0 1 0 1 0 0   0  7 14 21 28 35 42
-------------
0 1 2 3 4 5 6  
~~~

Shown as a flat series of ones and zeros the two bitboards look like this.
Leading zeros, i.e. the bits beyond position 48, are cut off. 

~~~
... 0000000 0000000 0000010 0000011 0000000 0000000 0000000 // encoding Xs
... 0000000 0000000 0000001 0000100 0000001 0000000 0000000 // encoding Os
      col 6   col 5   col 4   col 3   col 2   col 1   col 0
~~~

Got it? It's not that hard to understand, is it?

## Remember the Fill Level

There is an addition piece of information tracked that speeds up making and
undoing moves. The position to be filled next in a column is remembered in
an array named `height`. For the example shown, 

~~~
                6 13 20 27 34 41 48
. . . . . . .   5 12 19 26 33 40 47
. . . . . . .   4 11 18 25 32 39 46
. . . . . . .   3 10 17 24 31 38 45
. . . O . . .   2  9 16 23 30 37 44
. . . X X . .   1  8 15 22 29 36 43
. . O X O . .   0  7 14 21 28 35 42
-------------
0 1 2 3 4 5 6  
~~~

the values for `height` are:

~~~
int[] height = {0, 7, 15, 24, 30, 35, 42};
~~~

## Make Move

~~~
void makeMove(int col) {
    int move = 1L << height[col]++; // (1)
    bitboard[counter & 1] ^= move;  // (2)
    moves[counter++] = col;         // (3)
}
~~~

## Undo Move

~~~
void undoMove() {
    int col = moves[--counter];     // reverses (3)
    int move = 1L << --height[col]; // reverses (1)
    bitboard[counter & 1] ^= move;  // reverses (2)
}
~~~

## Four In A Row?

~~~
  6 13 20 27 34 41 48    Additional row
+---------------------+ 
| 5 12 19 26 33 40 47 |  top row
| 4 11 18 25 32 39 46 |
| 3 10 17 24 31 38 45 |
| 2  9 16 23 30 37 44 |
| 1  8 15 22 29 36 43 |
| 0  7 14 21 28 35 42 |  bottom row
+---------------------+
~~~

~~~
if (bitboard & (bitboard >> 6) & (bitboard >> 12) & (bitboard >> 18) != 0) return true; // diagonal \
if (bitboard & (bitboard >> 8) & (bitboard >> 16) & (bitboard >> 24) != 0) return true; // diagonal /
if (bitboard & (bitboard >> 7) & (bitboard >> 14) & (bitboard >> 21) != 0) return true; // horizontal
if (bitboard & (bitboard >> 1) & (bitboard >>  2) & (bitboard >>  3) != 0) return true; // vertical
return false;
~~~

The code is highly regular and redundant.

~~~
boolean isWin(long bitboard) {
    int[] directions = {1, 7, 6, 8};
    for(int direction : direction)
        if (bitboard & (bitboard >> direction) &
           (bitboard >> (2 * direction)) & (bitboard >> (3 * direction)) != 0)
           return true;
    return false;
}
~~~

~~~
boolean isWin(long bitboard) {
    int[] directions = {1, 7, 6, 8};
    long bb;
    for(int direction : directions) {
        bb = bitboard & (bitboard >> direction);
        if (bb & (bb >> (2 * direction)) != 0) return true;
    }
    return false;
}
~~~

Per direction the routine needs one assignment, two shifts, two binary
and-operations, one multiplication, and a comparison, i.e. seven operations
which relate to almost seven byte code instructions on the JVM.
A full evaluation considering all four directions requires four times that
much, that is 4 x 7 = 28 instructions. Including the instructions required
for the for-statement, we might end up with about 40 instructions altogether.

On a machine crunching 2 billion instructions per second, we can run
50 million checks of `isWin` per second.

## A Generator to List Moves

~~~
int[] listMoves() {
    int[] moves; // ArrayList<int> bzw. ArrayList<Integer>
    long TOP = 1000000_1000000_1000000_1000000_1000000_1000000_1000000L;

    let long bitboard = this.bitboard[counter & 1];
    let long newboard;
    for(int col = 0; col <= 6; col++) {  
        newboard = bitboard | (1L << height[col]); // simulation of move
        if (newboard & TOP == 0) moves.push(col);
    }
    return moves;
}
~~~
