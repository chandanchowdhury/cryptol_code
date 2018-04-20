# Cryptol Basics

* Not a general purpose language, so bit hard to think in it. 
* Functional
* Not designed to write generic routines that will work on arbitrary length values 

Why?


## Commands

### Load

Cryptol>:l <file name>

### Reload

Cryptol>:r 

### Type

Check the type of something.

Cryptol>:t True
Cryptol>:t 5
Cryptol>:t [1 .. 5]
Cryptol>:t (3,5,True)
Cryptol>:t tail

### Base

* Set the base of the output numbers.
* Default is hex.
* Can be binary, octal, decimal, hexadecimal.

To set decimal
Cryptol>:set base=10

### ascii

* Print ASCII values as characters instead of hex values.

Cryptol>:set ascii=on


### debug

Cryptol>:set debug=on

---

## Monomorphic Types

TODO: What is Monomorphic? (Immutable)

### Bit
* Fundamental data type.
* All other monomorphic data types are built using Bit.
* Values can be True or False.

### Words
* Sequence of Bits
* But are just numbers
* No negative or floating point numbers or Only positive integers and zero

Example, 
* 5 - is a word of 3 bits with a value of decimal 5
* 256 - is a word of 9 bits
* 0xff - is a word of 8 bits
* 5: [8] - is a word of 8 bits with a value of decimal 5
* AES128_key:[16] - is a new variable whose type is 16-bit word


### Sequences
* Collection of homogeneous (same type) elements.
* Written using square brackets.
* Sequence index starts at zero (which means it is actually an offset).
* Indices are always positive
* '@' is the Forward Index operator.
* '!' is the Reverse Index operator i.e. count backwards
* '#' is the concatenation operator
* @@ Forward permutation operator
* !! Reverse permutation operator
* >> is Right Shift
* << is Left Shift
* >>> is Right Rotate
* <<< is Left Rotate

Example
* [1, 2, 3, 4, 5]
* [1 .. 10]
* [1, 3 .. 12]
* [10, 9 .. 5]

Example
[10, 9 .. 5] @ 0
10

Example
[10, 9 .. 5] @ 2
8

Example
['a' .. 'z'] # [0 .. 9]
['a' .. 'z'] # ['0' .. '9']

Example
['A' .. 'Z'] @@ [1, 4] -> Select 1st and 4th element
['A' .. 'Z'] @@ [1, 3, 3, 6] -> Select 1st, 3rd, 3rd and 6th element
['A' .. 'Z'] !! [1, 4] -> Select 1st and 4th from reverse
[100 .. 200] !! [4, 1] -> Select 4th and 1st from reverse

The type [12][3][6] reads as follows: A sequence of 12 elements, each of which is a sequence of 3 elements, each of which is a 6-bit-wide word.

Q: How to define a sequence from 'a' to 'b' where 'a' and 'b' are arguments?
A: Not possible



### Tuples
* Collection of heterogeneous (different type) elements.
* Written using parentheses.
* A tuple component can be any type: Bits, Word, another tuple, sequence, record, etc.

Q: Can tuples contains functions?

Example: 
* (3,5,True) - ([a], [b], Bit)
* (2, (False, 3), 5) : ([8], (Bit, [32]), [32])


### Records
* Heterogeneous collection of arbitrary number of labeled elements.
* Like dictionaries/hash tables
* Written in curly braces.
* Can be nested.
* Can contain arbitrary types of elements (records, sequences, functions, etc.).

Example:
{name = "Chandan", stype = "Grad"}
{name = "Chandan", stype = "Grad"}.name
{name = "Chandan", stype = "Grad"}.stype



### Type
* Similar to 'typedef' in C
* Can be un-parameterized or parameterized

Un-parameterize Examples:
type Byte = [8]
type Char = Byte
type String n = [n]Char
type AES128_Key = [16]

Parameterize Examples:
type Point a = {x:[a], y:[a]}


Example,
type T = { foo : Bit, bar : [15] }

Here, T is our own type which contains a "foo" of type Bit and a "bar" which is a word of 15 bits.

## Polymorphic types

### reverse
reverse [1 .. 5]


### tail
tail [1 .. 5] = [0x2, 0x3, 0x4, 0x5]

Q: Can it take tuple or records?


### groupBy
groupBy\`{3} [1 .. 9]


### take

take\`{1} [1 .. 10] = [0x1]

take\`{4} [1 .. 15] = [0x1, 0x2, 0x3, 0x4]



## Arithmetic operations

### Addition

1 + 1 = 0
because size of 1 here is one Bit and the result is mod 2^1.

1 + (1:[8]) = 0x02
because size of second 1 here is 8 bit and the result is mod 2^8.


254 + 1 = ?
255 + 1 = ?


### Subtraction
3 - 5 
=0x06

because 3 - 5 = -2
-2 mod 2^^3 = 6


min 5 (-2) = 5  Why?

### Power
2 ^^ 3 = 0


### Adding two sequences

[1 .. 10] + [10, 9 .. 1]

=[0xb, 0xb, 0xb, 0xb, 0xb, 0xb, 0xb, 0xb, 0xb, 0xb]



## Polynomial Arithmetic

* Cryptol allows us to write polynomials
* Arithmetic operations can be done directly on the polynomials
* For our easy understanding we will set the output to binary using the command `:set base=2`

Example,
Cryptol> <| x^^6 + x^^4 + x^^2 + x^^1 + 1 |>

the out is
0b1010111

### pmult
pmult <| x^^3 + x^^2 + x + 1 |> <| x^^2 + x + 1 |>

The output is
0b101101

which is
<| x^^5 + x^^3 + x^^2 + 1 |>

### pdiv
* Returns the quotient
* Uses the long division method

TODO: is it important that it does long division?

#### Example
pdiv <| x^^3 + x^^2 + x + 1 |> <| x^^2 + x + 1 |>

the output is
0b0010

which is
<|x|>

### pmod
* Returns the remainder

#### Example
pmod <| x^^3 + x^^2 + x + 1 |> <| x^^2 + x + 1 |>

the output is
0b01

which is 
<|1|>

#### Example
pmod (pmult <| x^^3 + x^^2 + x + 1 |> <| x^^2 + x + 1 |>) <| x^^8 + x^^4 + x^^3 + x + 1 |>

the output is 
0b00101101

which is
<|x^^5 + x^^3 + x^^2 + 1|>


## Functions

* Basic structure of functional programming.
* Groups of declarations are organized based on indentation.
* Functions can take function(s) as argument.
* Can return only one value.

add (x, y) = x + y 
Here add is a function that takes one argument which is a tuple.

add x y = x + y
Here add is a function that takes two arguments.

### Currying
TODO

### Lambda expressions
* Functions small enough that we do not need a separate definition
* Instead of the name we use the backslash or “whack” character, ‘\’.
* the equals sign is replaced by an arrow ‘->’.

Increment function
Cryptol> increment 9 where increment x = x + 1

Same thing without function definition
Cryptol> (\x -> x + 1) 9

Q: Why not use the equal sign here instead of '->' ?

## Control Structures

### If-Then-Else
x = if y % 2 == 0 then 22 else 33

x = if y % 2 == 0 then 1
     | y % 3 == 0 then 2
     | y % 5 == 0 then 3
    else 7


### where
* Used to define body of a function

### Loops
* No while or for
* Use either recursion or recurrence



Ref:
https://github.com/GaloisInc/cryptol
https://github.com/dylanmc/CryptoBook
https://github.com/GaloisInc/ICryptol/blob/master/examples/AES.ipynb
