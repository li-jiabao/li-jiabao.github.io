
# Unicode

**Unicode** is a computing industry standard designed to consistently and uniquely encode characters used in written languages throughout the world. The Unicode standard uses hexadecimal to express a character. For example, the value 0x0041 represents the Latin character A. The Unicode standard was initially designed using 16 bits to encode characters because the primary machines were 16-bit PCs.

When the specification for the Java language was created, the Unicode standard was accepted and the `char` primitive was defined as a 16-bit data type, with characters in the hexadecimal range from 0x0000 to 0xFFFF.

Because 16-bit encoding supports 2<sup>16</sup> (65,536) characters, which is insufficient to define all characters in use throughout the world, the Unicode standard was extended to 0x10FFFF, which supports over one million characters. The definition of a character in the Java programming language could not be changed from 16 bits to 32 bits without causing millions of Java applications to no longer run properly. To correct the definition, a scheme was developed to handle characters that could not be encoded in 16 bits.

The characters with values that are outside of the 16-bit range, and within the range from 0x10000 to 0x10FFFF, are called **supplementary characters** and are defined as a pair of `char` values.

This lesson includes the following sections:

- [Terminology](terminology.html) &#150; Code points and other terms are explained.
- [Supplementary Characters as Surrogates](supplementaryChars.html) &#150; 16-bit surrogates are used to implement supplementary characters, which cannot be implemented as a single primitive `char` data type.
- [Character and String API](characterClass.html) &#150; A listing of related API for the `Character`, `String`, and related classes.
- [Sample Usage](usage.html) &#150; Several useful code snippets are provided.
- [Design Considerations](design.html) &#150; Design considerations to keep in mind to ensure that your application will work with any language script.
- [More Information](info.html) &#150; A list of further resources are provided.
