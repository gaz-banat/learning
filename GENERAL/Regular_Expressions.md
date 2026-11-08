

CHARACTER CLASS

1
[abc]
a, b, or c (simple class)
2
[^abc]
Any character except a, b, or c (negation)
3
[a-zA-Z]
a through z or A through Z, inclusive (range)
4
[a-d[m-p]]
a through d, or m through p: [a-dm-p] (union)
5
[a-z&&[def]]
d, e, or f (intersection)
6
[a-z&&[^bc]]
a through z, except for b and c: [ad-z] (subtraction)
7
[a-z&&[^m-p]]
a through z, and not m through p: [a-lq-z](subtraction)



QUANTIFIER

X?
X occurs once or not at all
X+
X occurs once or more times
X*
X occurs zero or more times
X{n}
X occurs n times only
X{n,}
X occurs n or more times
X{y,z}
X occurs at least y times but less than z times



Set(Value)?  	- 	The word ‘Value’ may occur once or not at all. The ‘?’ quantifier works on previous ‘token’. Here the token is a capturing group enclosed in parenthesis.
Things?		-	The letter ’s’ may occur one or not at all. The ‘?’ quantifier works on a previous ‘token’. Here the token is the letter ’s’





METACHARACTERS

.
Any character (may or may not match terminator)
\d
Any digits, short of [0-9]
\D
Any non-digit, short for [^0-9]
\s
Any whitespace character, short for [\t\n\x0B\f\r]
\S
Any non-whitespace character, short for [^\s]
\w
Any word character, short for [a-zA-Z_0-9]
\W
Any non-word character, short for [^\w]
\b
A word boundary
\B
A non word boundary



Subexpression
Matches
^
Matches the beginning of the line.
$
Matches the end of the line.
.
Matches any single character except newline. Using m option allows it to match the newline as well.
[...]
Matches any single character in brackets.
[^...]
Matches any single character not in brackets.
\A
Beginning of the entire string.
\z
End of the entire string.
\Z
End of the entire string except allowable final line terminator.
re*
Matches 0 or more occurrences of the preceding expression.
re+
Matches 1 or more of the previous thing.
re?
Matches 0 or 1 occurrence of the preceding expression.
re{ n}
Matches exactly n number of occurrences of the preceding expression.
re{ n,}
Matches n or more occurrences of the preceding expression.
re{ n, m}
Matches at least n and at most m occurrences of the preceding expression.
a| b
Matches either a or b.
(re)
Groups regular expressions and remembers the matched text.
(?: re)
Groups regular expressions without remembering the matched text.
(?> re)
Matches the independent pattern without backtracking.
\w
Matches the word characters.
\W
Matches the nonword characters.
\s
Matches the whitespace. Equivalent to [\t\n\r\f].
\S
Matches the nonwhitespace.
\d
Matches the digits. Equivalent to [0-9].
\D
Matches the nondigits.
\A
Matches the beginning of the string.
\Z
Matches the end of the string. If a newline exists, it matches just before newline.
\z
Matches the end of the string.
\G
Matches the point where the last match finished.
\n
Back-reference to capture group number "n".
\b
Matches the word boundaries when outside the brackets. Matches the backspace (0x08) when inside the brackets.
\B
Matches the nonword boundaries.
\n, \t, etc.
Matches newlines, carriage returns, tabs, etc.
\Q
Escape (quote) all characters up to \E.
\E
Ends quoting begun with \Q.






Boundary matchers
^	The beginning of a line
$	The end of a line
\b	A word boundary
\B	A non-word boundary
\A	The beginning of the input
\G	The end of the previous match
\Z	The end of the input but for the final terminator, if any
\z	The end of the input
 
Greedy quantifiers
X?		X, once or not at all
X*		X, zero or more times
X+		X, one or more times
X{n}		X, exactly n times
X{n,}	X, at least n times
X{n,m}	X, at least n but not more than m times
 
Reluctant quantifiers
X??		X, once or not at all
X*?		X, zero or more times
X+?		X, one or more times
X{n}?	X, exactly n times
X{n,}?	X, at least n times
X{n,m}?	X, at least n but not more than m times
 
Possessive quantifiers
X?+		X, once or not at all
X*+		X, zero or more times
X++		X, one or more times
X{n}+	X, exactly n times
X{n,}+	X, at least n times
X{n,m}+	X, at least n but not more than m times
 
Logical operators
XY		X followed by Y
X|Y		Either X or Y
(X)		X, as a capturing group
 
Back references
\n		Whatever the nth capturing group matched
\k<name>	Whatever the named-capturing group "name" matched
 
Quotation
\		Nothing, but quotes the following character
\Q		Nothing, but quotes all characters until \E
\E		Nothing, but ends quoting started by \Q
 
Special constructs (named-capturing and non-capturing)
(?<name>X)				X, as a named-capturing group
(?:X)					X, as a non-capturing group
(?idmsuxU-idmsuxU) 		Nothing, but turns match flags i d m s u x U on - off
(?idmsux-idmsux:X)  		X, as a non-capturing group with the given flags i d m s u x on - off
(?=X)					X, via zero-width positive lookahead
(?!X)					X, via zero-width negative lookahead
(?<=X)					X, via zero-width positive lookbehind
(?<!X)					X, via zero-width negative lookbehind
(?>X)					X, as an independent, non-capturing group
