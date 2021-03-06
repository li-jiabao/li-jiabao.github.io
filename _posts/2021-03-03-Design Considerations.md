---
title: Design Considerations
author: lijiabao
date: 2020-12-06 01:51:05.684598300 +0800
category: Internationalization
categories: Internationalization
tags: Internationalization
---

# Design Considerations

To write code that works seamlessly for any language using any script, there are a few things to keep in mind.
<th id="h1">Consideration</th><th id="h2">Reason</th>
<td headers="h1">Avoid methods that use the `char` data type.</td><td headers="h2">Avoid using the `char` primitive data type or methods that use the `char` data type, because code that uses that data type does not work for supplementary characters. For methods that take a `char` type parameter, use the corresponding `int` method, where available. For example, use the `Character.isDigit(int)` method rather than `Character.isDigit(char)` method.</td>
<td headers="h1">Use the `isValidCodePoint` method to verify code point values.</td><td headers="h2">A code point is defined as an `int` data type, which allows for values outside of the valid range of code point values from 0x0000 to 0x10FFFF. For performance reasons, the methods that take a code point value as a parameter do not check the validity of the parameter, but you can use the `isValidCodePoint` method to check the value.</td>
<td headers="h1">Use the `codePointCount` method to count characters.</td><td headers="h2">The `String.length()` method returns the number of code units, or 16-bit `char` values, in the string. If the string contains supplementary characters, the count can be misleading because it will not reflect the true number of code points. To get an accurate count of the number of characters (including supplementary characters), use the `codePointCount` method.</td>
<td headers="h1">Use the `String.toUpperCase(int codePoint)` and `String.toLowerCase(int codePoint)` methods rather than the `Character.toUpperCase(int codePoint)` or `Character.toLowerCase(int codePoint)` methods.</td><td headers="h2">While the `Character.toUpperCase(int)` and `Character.toLowerCase(int)` methods do work with code point values, there are some characters that cannot be converted on a one-to-one basis. The lowercase German character &#223;, for example, becomes two characters, SS, when converted to uppercase. Likewise, the small Greek Sigma character is different depending on the position in the string. The `Character.toUpperCase(int)` and `Character.toLowerCase(int)` methods cannot handle these types of cases; however, the `String.toUpperCase` and `String.toLowerCase` methods handle these cases correctly.</td>
<td headers="h1">Be careful when deleting characters.</td><td headers="h2">When invoking the `StringBuilder.deleteCharAt(int index)` or `StringBuffer.deleteCharAt(int index)` methods where the index points to a supplementary character, only the first half of that character (the first `char` value) is removed. First, invoke the `Character.charCount` method on the character to determine if one or two `char` values must be removed.</td>
<td headers="h1">Be careful when reversing characters in a sequence.</td><td headers="h2">When invoking the `StringBuffer.reverse()` or `StringBuilder.reverse()` methods on text that contains supplementary characters, the high and low surrogate pairs are reversed which results in incorrect and possibly invalid surrogate pairs.</td>
