# Bitboards and Connect Four

[Fhourstones](https://en.wikipedia.org/wiki/Fhourstones) is a benchmark that is based on a highly efficient
[implementation](http://tromp.github.io/c4/fhour.html) of the game [Connect Four](https://en.wikipedia.org/wiki/Connect_Four).
Its author, [John Tromp](http://tromp.github.io/), uses two bitboards to represent the board from each player's
point of view. A [bitboard](https://en.wikipedia.org/wiki/Bitboard) encodes the current position of a single player
as a series of ones and zeros; a one denoting the player's token, a zero denoting either an empty position or an opponent's token.
The encoding is designed for speed in operating with the board. Especially querying the board for four in a row is extremely fast.

I figured that my students struggle with understanding John's implementation. That's why I created this document.
My explanation leans on John's code. However, we don't go through John's code line by line. Instead, I try to
help you get the point of the encoding and the way using it. Equipped with this understanding you should figure out
the details of John's implementation yourself.

Enjoy,

Dominikus Herzberg, [@denkspuren](https://twitter.com/denkspuren) 

## A Binary Board Encoding for Connect Four

### Using Two Longs to Encode the Board

In Java and in many other programming languages, a so-called long integer is composed of eight bytes.
Eight bytes are 64 bits (1 byte = 8 bits). If presented as a series of ones and zeros, the
most significant bit (MSB) is on the left hand side, the least significant bit (LSB) is on the right hand side.
That's why we number the bits in the way shown below from right to left. Remember, programmers start counting
with zero ;-)

~~~
63 62 61 60 ... 48 47 46 45 ... 3 2 1 0
~~~

Connect Four is played on a vertical board with seven columns and six rows. That makes 42 slots.
The board with these 42 slots is shown in the diagram below.
We add an additional row on top for convenience purposes. This additional row on top is for computational reasons only.
And so are the bits numbered 49 to 63, adding two more columns and a bit.
The bits of the top row (6, 13, 20, etc.) and the bits on the right (49 - 63) are seemingly unused
but nonetheless important and not to forget in their role when it comes to manipulating the bits.

The numbers indicate the position in the bit representation of a long integer.

~~~
  6 13 20 27 34 41 48   55 62     Additional row
+---------------------+ 
| 5 12 19 26 33 40 47 | 54 61     top row
| 4 11 18 25 32 39 46 | 53 60
| 3 10 17 24 31 38 45 | 52 59
| 2  9 16 23 30 37 44 | 51 58
| 1  8 15 22 29 36 43 | 50 57
| 0  7 14 21 28 35 42 | 49 56 63  bottom row
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

### Bitwise Operations

To work with a series of bits you need operations to manipulate them. All we need is two kinds of operations: 
to shift bits and to combine bits.

#### Shifting Bits

The operators `>>` and `<<` shift bits to the right and to the left, respectively.

Saying e.g. `0b10101110 >> 3` means that three bits are shifted out to the right with three zeros being shifted in on the left. Here, the prefix `0b` indicates that the following digits represent a binary number and not a decimal number. Assumed that `0b10101110` is a binary number of maximum eight bits, the result is `0b00010101`.

Accordingly, `0b10101110 << 3` shifts three bits out to the left and shifts in three zeros on the right. The result is `0b01110000` if we assume a length of eight bits.

#### Combining Bits

The only two operators we need in this context are the so-called XOR-operator and the AND-operator.

The XOR-operator (written as `^`) takes two binary numbers, say `0b10101110 ^ 0b10011111`, and compares bits of the same position. If both bits are not equal, the result is a one; otherwise it's a zero. That's the rule for the XOR-operator.

If we write the second number below the first number, the bit positions are easier to compare. Only pairs of `10` and `01` result in a `1`.

~~~
  10101110
^ 10011111
----------
  00110001
~~~

That is, `0b10101110 ^ 0b10011111` results in `0b00110001`.

The AND-operator (written as `&`) also takes two binary numbers. Comparing the bits of both numbers, it returns a one only if the first bit *and* the second bit are one; therwise it returns zero for the position under consideration. For example

~~~
  10101110
& 10011111
----------
  10001110
~~~

In other words, `0b10101110 & 0b10011111` results in `0b10001110`.

For more information on bit shifting and other bit operations such as OR, AND and NOT you might consult Wikipedia on [Bitwise Operations](https://en.wikipedia.org/wiki/Bitwise_operation).

## Let's Play with the Bitboards

To demonstrate the use of bitboards we implement four methods: make a move (`makeMove`), undo a move (`undoMove`), check whether there are four in a row (`isWin`), list all possible moves in a given situation (`listMoves`). As our means of communication, we use pseudo-code which is close to TypeScript and Java. Only slight adaptations are required for any of the two languages. It is up to you to properly embed this code in a class statement. 

### Remember the Fill Level

Before we look into the methods more closely, there is an additional piece of information tracked that speeds up making and
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

The variable `height` serves as a memory where the next disk goes given the column. Otherwise we would need to search for the next empty slot given the bottom of a column. Having a memory saves searching and makes things faster. And that is what we are up to with bitboards. We want to be as fast as possible with manipulating a board representation of Connect Four.

### Counting helps

Another piece of information we track is the number of moves done, a `counter`. Each time we make a move we increment the counter, each time we undo a move we decrement the counter. For incrementation we use the increment operator `++` (add one), for decrementation we use the decrement operator `--` (subtract one). We use incrementation and decrementation also for manipulating the `height` per column.

There is a neat trick to determine whether it is `X`'s or `O`'s turn to move given the counter. If `counter` is even, it's `X`'s turn, if it's odd, it's `O`'s turn (you might do it the other way around if you like). If you think about `counter` in its binary form, `counter` is even if the rightmost bit (the LSB) is zero and it is odd if the LSB is one. Using the AND-operator, `counter & 1` returns `0` for even counts and `1` for odd counts. We will use this trick to determine, which bitboard of the two we are interested in.

### Make Move

~~~
void makeMove(int col) {
    long move = 1L << height[col]++; // (1)
    bitboard[counter & 1] ^= move;  // (2)
    moves[counter++] = col;         // (3)
}
~~~

### Undo Move

~~~
void undoMove() {
    int col = moves[--counter];     // reverses (3)
    long move = 1L << --height[col]; // reverses (1)
    bitboard[counter & 1] ^= move;  // reverses (2)
}
~~~

### Four In A Row?

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
    for(int direction : directions)
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

### A Generator to List Moves

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
