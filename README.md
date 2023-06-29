# FortranCompiler
## Idea of splitting :

The line tokens = re.findall('\w+|\W', text) uses regular expressions to split the input text into a list of tokens. Here's how it works:

\w+ matches one or more word characters (letters, digits, or underscores).
\W matches any non-word character.
| is the OR operator in regular expressions, so \w+|\W matches either one or more word characters or any non-word character.
By using re.findall(pattern, text), the function finds all occurrences of the pattern in the text and returns them as a list of strings. Each string represents a token in the input text.

For example, given the input text "program my_program integer x = 10;", the resulting list of tokens would be ['program', 'my_program', 'integer', 'x', '=', '10', ';'].

The purpose of splitting the text into tokens is to analyze and classify each token according to its type, as defined by the Token_type enum. This classification helps in further processing or understanding the structure and meaning of the program code





Some special parts in scanner code :
Handling comments by ignoring what is after! :
In following lines of code:
Please note as Fortan keep appending string to comment section until newline is entered that why \n condition was put and at the same time we are ignoring the spaces as not to count as token type.




![pic1](https://github.com/Zeinahesham308/FortranCompiler/assets/108671897/ec92bba7-22d3-4c33-bab6-06171211b929)


