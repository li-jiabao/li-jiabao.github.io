
# Bitwise and Bit Shift Operators

The Java programming language also provides operators that perform bitwise and bit shift operations on integral types. The operators discussed in this section are less commonly used. Therefore, their coverage is brief; the intent is to simply make you aware that these operators exist.

The unary bitwise complement operator "`~`" inverts a bit pattern; it can be applied to any of the integral types, making every "0" a "1" and every "1" a "0". For example, a `byte` contains 8 bits; applying this operator to a value whose bit pattern is "00000000" would change its pattern to "11111111".

The signed left shift operator "`&lt;&lt;`" shifts a bit pattern to the left, and the signed right shift operator "`&gt;&gt;`" shifts a bit pattern to the right. The bit pattern is given by the left-hand operand, and the number of positions to shift by the right-hand operand. The unsigned right shift operator "`&gt;&gt;&gt;`" shifts a zero into the leftmost position, while the leftmost position after `"&gt;&gt;"` depends on sign extension.

The bitwise `&amp;` operator performs a bitwise AND operation.

The bitwise `^` operator performs a bitwise exclusive OR operation.

The bitwise `|` operator performs a bitwise inclusive OR operation.

The following program, 
[`BitDemo`](examples/BitDemo.java), uses the bitwise AND operator to print the number "2" to standard output.

```


class BitDemo {
    public static void main(String[] args) {
        int bitmask = 0x000F;
        int val = 0x2222;
        // prints "2"
        System.out.println(val &amp; bitmask);
    }
}

```