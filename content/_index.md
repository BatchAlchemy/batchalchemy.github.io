---
title: 'Batch Syntax Research Summary'
date: 2024-04-23T13:39:28+02:00
draft: false
# bookComments: false
# bookSearchExclude: false
---
# Batch Syntax Research Summary

## Introduction (WIP)
- motivated because there is no documentation
- we aim to help to strengthen the understanding behind it
- we definitely have not found out every syntax detail of Batch, just some interesting things to know about / consider
- disclaimer: Batch syntax is weird, so keep everything with a grain of salt
- note: everything we talk about is considered to be done in the context of Batch mode (there is a difference between Batch mode and interactive mode, look it up *here*)

# Findings on the Batch Syntax

## CALL & valid local filenames

This experiment was motivated by the idea that parenthesis characters may appear in Windows filenames. At the same time they get used in syntactical ways in Batch scripts, such as statements concerning the `if` and `else` keywords.

We decided to use the ``call`` key keyword to test what filenames are to be considered valid command names by the Windows Command Prompt (cmd.exe).

For that, we created eleven Batch files, namely `x)`, `).bat`, `)x.bat`, `x(.bat`, `x.bat(`, `(x.bat`, `()`, `().bat`, `(x).bat`, `x.bat)` and `(.bat`, all of which contain the following Batch code:

```batch=
@echo inside %0     :: prints the filename of the called file on screen 
@exit /b            :: exits called batch and return to callee 
```

This code is meant to display which called file the callee calls.

Speaking of the callee, we use the Batch code below for the execution of the experiment. This Batch gets evoked from the Windows Command Prompt to display the results. The comments next to the lines of code represent the result of the corresponding line of code. 

```batch=
call x).bat		:: works; prints `inside x).bat`
call ).bat		:: works; prints `inside ).bat`
call )x.bat		:: works; prints `inside )x.bat`

call x(.bat		:: failure; only `x` gets interpreted as a command_name
call x.bat(		:: failure; only `x.bat` gets interpreted as a command_name

call (x.bat		:: failure; does nothing
call (.bat		:: failure; does nothing
call x.bat)		:: failure; instead tries to open file in another way cuz file ending

call ()			:: syntax error; `)` cannot be used here
call ().bat		:: syntax error; `)` cannot be used here
call (x).bat           :: syntax error; `.bat` cannot be used here
```

As one can see, the first three cases (**ll. 1-3**) get correctly executed. The corresponding files exist and the closing parentheses to the left of the file extension seem to work without issue.

A little bit more interesting are the next two cases (**ll. 5-6**): Here, the opening parenthesis seems to delimit the called filename to the right. Because these files do not exist (yet), we get the following two errors from the Windows Command Prompt:

```bash=
C:\Users\defaultuser>call x(.bat
The command "x" is either misspelled or could not be found.

C:\Users\defaultuser>call x.bat(
The command "x.bat" is either misspelled or could not be found.
```

To check if they map as one might think, we create two more files, namely `x` and `x.bat`.

First, the code of the extensionless file `x` will be:
```batch=
@echo inside %0
@echo Actual filename: x
@exit /b
```

And the content of `x.bat` will be mostly the same; it just differs in the hard-coded actual filename, which is to be printed:
```batch=
@echo inside %0
@echo Actual filename: x.bat
@exit /b
```

Now executing our caller Batch file again, we get the following output from the original ll. 5-6:
```bash=
C:\Users\defaultuser>call x(.bat
inside x
Actual filename: x.bat

C:\Users\defaultuser>call x.bat(
inside x.bat
Actual filename: x.bat
```

The result shows that the closing parenthesis delimits the filename to be called to the right and that the `x.bat` has priority when either filename is being interpreted by the Windows Command Prompt.

This is backed by the fact that upon removing ``x.bat`` from the working directory, the extensionless `x` will still not be called by the calling lines of code in either case:
```bash=
C:\Users\defaultuser>call x(.bat
The command "x" is either misspelled or could not be found.

C:\Users\defaultuser>call x.bat(
The command "x.bat" is either misspelled or could not be found.
```

The next three cases (ll. 8-10) fail to call the files as well. As seen before, the closing parenthesis in **lines 8 and 9** delimit the filenames to the right, now yielding empty filenames (because the files' names on disk start with the closing parenthesis). Hence, the Windows Command Prompt interprets these two lines of code simply as a `call` command with empty arguments, yielding in an empty output with no file being called:
```bash=
C:\Users\defaultuser>call (x.bat

C:\Users\defaultuser>call (.bat

```

**Line 10** also yields an empty output, but because of the placement of the opening parenthesis (which seemingly does not work as a delimiter as seen before), it is considered part of the file extension. As a result, for us, the Windows OS prompted us with a popup window asking what application we'd like to open `.bat)` files with.

Considering the last three lines of code (**ll. 12-14**), syntax errors get reported by the Windows Command Prompt. In the first two cases, it seems as if the `call` command gets interpreted again as if it had no arguments. After that interpretation, now having nothing within the pair of parenthesis yields a syntactic error, claiming that `)` cannot be used in that place. Note that the Batch does not abort, but instead just skips the syntactically erroneous line of code. Speaking of the very last case, now having a non-empty parentheses, the output claims that `.bat` suddenly became the syntax problem. Again call is probably being interpreted without having any arguments, the parentheses get interpreted as a code block and thus the file extension is out of place.

***Conclusion***:
*Filenames containing opening parenthesis can lead to unintentional interpretation by the Windows Command Prompt, although their existence is inherently permitted within the Windows OS. Also, there can be a difference between how a Batch thinks it got evoked and how it is actually being evoked.*

***Future research***
*Future research of interpretation of Windows filenames in connexion with Batch code blocks like the ones within if-else statements*


## The correlation of IF and ELSE (WIP)

## SET variable names, whitespaces, delimiters & more
This experimentation was motivated during general research on the assignment command. As we've gathered more information about its general working, it became evident that further inspection on variable names, whitespaces and delimiters were needed, as well as the differences between the different options/switches.

First, the set command interprets variables and their values differently, depending on the used switches. Take a look at the following Batch script. The comments describe the results of the set commands.

```batch=
:: this sets a variable named `var1 ` (note the whitespace) to the value ` 1    `  (note the whitespaces below too, e.g. by selecting the text)
@set var1 = 1    

:: this sets a variable named `var2` to the value `1` (note the whitespaces below, e.g. by selecting the text)
@set /a var2 = 1    

:: print this to show the effect of the different handling of whitespaces (the slash character just functions as a visual separator)
echo %var1 %/%var2%/
```
The most useful insight to gain here is that the `/a` switch ignores any trailing or leading whitespaces upon command parsing, even for the variable's name. Here is some proof, which we get by calling `set var` (select the text to highlight the whitespaces). This command displays all *variable=value* pairs of the current environment whose variable names start with `var`:
```batch=
C:\Users\defaultuser>set var
var1 = 1    
var2=1
```

Executing the script above in a fresh environment presents us with the output of line 8 as follows:
```batch=
C:\Users\defaultuser>echo  1    /1/
 1    /1/
```
Note that the displaying information of the executed commands has already substituted variables in it. For further clarification, we deviate a little bit from the actual visual appearance of the terminal's output in a way that we execute line 8 above in interactive mode. Note that in doing so, only the visual output inside the terminal changes; the logic or any semantics stay the same. For the same purpose of visibility, we replace every whitespace of the variables and their values with `_`:
```batch=
C:\Users\defaultuser>echo %var1_%/%var2%/
 _1____/1/
```

Also note that in the `/a` switch also forbids the usage of whitespace characters within a variable name, while the `\p` switch or assignments without the use of any special flags allows these variables:
```batch=
C:\Users\defaultuser>set var 3 = 1    
set var 3 = 1

C:\Users\defaultuser>set /a var 3 = 1    
Missing operator.

C:\Users\defaultuser>set /p var 3 = Expecting user input... 2    

C:\Users\defaultuser>set var
var 3 = 2    
var1 = 1    
var2=1
```

For the sake of completeness, here is the Batch file that evoked the output from above.
```batch=
set var 3 = 1    
set /a var 3 = 1    
set /p var 3 = Expecting user input...

set var
```

Speaking of the `/p` switch, the value of the set command (in this case functioning as the text prompting for user input or data redirection) ignores any leading whitespace characters. Interestingly, the three most common delimiting characters (namely `,`, `;` and `=`\*), which often function very similar to whitespaces in the context of Batch commands, do not fulfill their usual job when calling `set /p`:


\*Note that that the Batch syntax forbids prompts of assignment commands which start with an `=` character, either with or without leading whitespaces:
```batch=
set /p var4 =    =Expecting user input.

echo "still executing"
```

This Batch displays a syntax error upon execution - before any input was given via the terminal. Typical for Batch, execution continues anyhow.
```batch=
C:\Users\defaultuser>set /p var4 = =Expecting user input.
The syntax of the command is incorrect.

C:\Users\defaultuser>echo "still executing"
"still executing"
```

Now we dive a little bit deeper into the concept of delimiting characters for assigned variable names. Again, taking the typical non-whitespace delimiting characters of Batch (namely `,`, `;` and `=`\*) to craft some weird commands. We use the following lines of Batch code to test which of these function as whitespaces. 
```batch=
@set,var5, =1    :: this sets a variable named `,var5, ` to the value `1`
@set;va r5;=2    :: this sets a variable named `;va r5;` to the value `2`
@set $va=r5=3    :: this sets a variable named `$va` to the value `r5=3`
```
Let's display the results using the `set` command. Here is the corresponding excerpt of our set of environment variables:
```batch=
C:\Users\defaultuser>set
$va=r5=3
,var5, =1
;va r5;=2 
[...]
```

Since the displaying information via `set` gives no hint for the third line of code or the first line of the results of `set` respectively (both meaning the `$va=r5=3` case), we use the following lines of code for variable inspection:

The resulting displaying information is as follows:
```batch=
C:\Users\defaultuser>echo %,var5, %
1

C:\Users\defaultuser>echo %;va r5;%
2

C:\Users\defaultuser>echo %$va%
r5=3

C:\Users\defaultuser>echo %var5%
ECHO is on.

C:\Users\defaultuser>echo %va r5%
ECHO is on.

C:\Users\defaultuser>echo %$va=r5%
ECHO is on.
```

Let's talk about the results. For now, we reference the the resulting code lines above. Line 1 to 5 tell us, that whitespaces, commas and semicolons are valid parts of variable names for the standard usage of the set command. If we take a look back at the original Batch script, we can see, that `,` and `;` interestingly fulfill two functionalities at the same time: They are valid parts of variable names *AND* they separate the command name from its first argument (working as a whitespace). This is especially interesting, as in the case of `@set $va=r5=3`, the whitespace functions just as a separator, not as part of the variable's name.

Lines 8 & 9 simply show that the first `=` character was taken as the operator of the assignment operation. Lines 11, 14 and 17 show that the other imaginable variable names are not set, since they got expanded to nothing, thus leaving just the `echo` command without arguments for execution.

Note that trying to use `=` as both a separator and as part of a variable name leads to a syntax error.
```batch=
C:\Users\defaultuser>set=var6=1
The syntax of the command is incorrect.
```

A last thing to note here about the prompt switch: The `/p` option seems to not just naively print out the entire rest of the line as a prompt, as one can see in the case where the prompting does not get answered by user interaction but an empty stream instead. Maybe not that interesting though.
```batch=
set /p var7=Expecting stream input...  <nul
```

Which gets evoked in the Windows Command Prompt as follows (the command has finished execution since it received a stream, but no variable was set). Also note the additional `0` which originated from the `<nul` stream.
```batch=
C:\Users\defaultuser>set /p var7=Expecting stream input...  0<nul
Expecting stream input...
```

At last, interestingly, in the context of the arithmetic switch `/a`, an assignment without stating a variable name is syntactically completely valid:
```batch=
set /a 2+3
```

Upon executing it from a Batch, this shows no output and no variable is being set (well, of course not - there is no variable name defined). However, when this line of code gets executed in **interactive** mode, it actually displays the arithmetic result:
```batch=
C:\Users\defaultuser>set /a 2+3
5
```

***Conclusion***:
- *Batch normally assigns leading and trailing whitespaces to set variables and their values respectively (except for the separating whitespace between the `set` command name and its first argument). Interestingly, the characters `,` and `;` function as both a separation character between the `set` command name and its first argument AND as part of the variable name. This is not the case for `=`, which instead leads to a syntax error.*
- *Moreover, `/p` switch assignments ignore leading whitespace characters for their prompt text. They yield syntax errors when a prompt starts with an `=` character; and leading `,` and `;` characters (which often count as leading whitespace characters for in other Batch command contexts) become part of the displayed prompt.*
- *When using the `/a` switch, leading and trailing whitespace characters of both, the variable name AND its value get ignored and whitespaces within a variable name is syntactically not allowed. Note that the latter is valid in the no-switch and `/p` cases. An arithmetic assignment usage without a variable name being stated is syntactically valid. Variable names during an arithmetic assignment must not contain typical arithmetic characters such as `/`, `*`, `-`, `+`; meanwhile, in non-switch and /p-switch case, they are allowed, even as leading or trailing characters.*