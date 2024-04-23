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


:: note that everything in quotes should work
:: note that I shuffled the lines multiple times so parenthesis positions should be tested by that -> it seems to be irrelevant since the results are all the same.
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