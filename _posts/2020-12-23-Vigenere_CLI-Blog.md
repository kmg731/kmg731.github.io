---
layout: post
tags: project
title:  "Vigenere Cypher in C"
---

# Introduction

[Original Project in Python](https://github.com/kmg731/vignereCipher)
[GitHub Repo for this project](https://github.com/kmg731/Vigenere-in-C)

About a year ago, I built a little python tool to encrypt and decrypt small messages using the Vigenere cypher.  
It worked out without any errors and was a pretty small program overall (only about 50 lines of code).  Since it
was a fun and interesting project, I've decided to rework it as a CLI applicaton written in C.  I've never made a 
straight CLI application, but hey, how hard could it be?  Prepare for an overly verbose article about something
that's probably pretty simple.  But hey, laugh at the places where I got stuck if you want.  

If you're unfamiliar with the Vigenere cipher, it is essentially an extension of the Caesar cipher.  In the Caesar 
cipher, each letter in a word is shifted up or down the alphabet by a fixed number.  

    ABCDEFGHIJKLMNOPQRSTUVWXYZ  - Normal alphabet, no shift
    FGHIJKLMNOPQRSTUVWXYZABCDE  - Caesar cipher, shift = 5

Say, for example, I wanted to 
encrypt the word "TOAD" using the Caesar cipher.  I would first select a shift value (let's use 5) and then select 
the letter 5 after the original letter.  Thus, "TOAD" would now become "YTFI."  The Caesar cipher is a bit simple.  
We can do better.

The Vigenere cipher uses the letters of a keyword to encrypt each letter of the main message.  If, for example, I want
to use the Vigenere cipher on the word "TOAD," I would first select a keyword to use in the encryption, let's go with
"FROG" for this one.  

-   First, you encrypt the first letter of "TOAD" using "F" as the shift value.  You get: Y
-   Next, do the same to "O" using "R".  you get: F
-   Repeat for "A," using "O": O
-   Finish with "D" and "G": J

The final encryption for "TOAD" using "FROG" as the key value is now "YFOJ" instead of "YTFI."
This is harder and less obvious to crack than the regular Caesar Cipher.  It is also not very hard to see how easy it
can be to implement this with code. 


# The Code Part

The Caesar cipher is quite easy to implement in code.  One nice feature of C is that chars can be evaluated as ints. 
As far as I know, the only way to use this int/char feature in Python is to use `ord()` and `chr()`.  This always felt
a bit cumbersome, so it's nice to be a bit more direct when modifying characters with numbers.  

Along with directly porting the funtcionality of the original Python program to an easier command line program, I'd
also like to add in some new functionality, mainly that of being able to encrypt an entire .txt file.  I'll have to 
play around and see how the command line handles files as arguments.  


## Structure

The first thing to think about is how I want the program to handle the encrypting.  I think the most straightforward
structure is to first create two functions for a Caesar cipher
and then write two more wrapper functions which give the Vigenere cypher.  I can also `inline` the original
Caesar encrypt/decrypt to make the process a bit faster.  

The string will also have to go through some kind of formatting funtion if we want this to scale up to full documents.
This means that, unfortunately, spaces and punctuation will have to be removed and the entire message will be output as
one single line of characters.  I can't really imagine some way to easily add the spaces back in without this ballooning
to something massive, so I'll leave it out.


## Functions

NOTE: I ran into a few bugs at this point.  I'll cover them in the Bugs section.  The code going forward will be 
correct and with the bugs fixed. 

So, how do we actually implement the Caesar encrypt/decrypt in C?  Thinking about all the addition and subtraction
can be a bit weird so I'll try to walk through it as best I can.

The formula for adding the shift to a character while also keeping it in the correct range is:

    (((char + shift - 97) % 26) + 97)

This first takes the numeric value of the char, adds the shift and then subtracts it from 97 (97 being the ascii
code for 'a'.  We could have just as easily written `char` + shift - 'a').  This will give us the position of the 
letter in the alphabet.  We perform modulus division on that number so it won't be out of range, and then add back
'a' to it to get the proper ascii character.  

One thing that didn't occur to me immediately was what parameters the Caesar encrypt/decrypt should take.  
Originally, it made the most sense to use `char*`, since 
the cipher is meant to take strings and encode or decode them.  Though, after thinking about it for a bit, it makes
more sense that they would take only `char`, since the main use will be to encrypt or decrypt a single `char` within
the wrapper function.  This should also increase speed since there is now no use for any loops and can be done with
just some simple arithmetic. We can also remove the function call to `strlen()` too.  Our function originally
went from:

    static inline void
    caesarEncode(char* message, int shift)
    {
      int len = strlen(message);
    
      for (int i = 0; i < len; ++i){
        message[i] = (((message[i] + shift - 97) % 26) + 97);
      }
    
      return message;
    }

to now only being:

    static inline char
    caesarEncode(char message, int shift)
    {
      return (((message + shift - 97) % 26) + 97);
    }

Obviously if we wanted to add functionality for specifically Caesar ciphers, we wouldn't be able to do this, but since
this is only a wrapper function to be used within another function, there is no reason to make it more complicated than
it has to be.  Since this function will be called so many times, it also should be as simple as possible.  Since this is 
very simple, we're also able to add `inline` to get it to run a bit faster too. `inline` will remove the funciton call 
overhead.  

Now, like I originally thought, you might be thinking that the `caesarDecode()` function wil be just as easy to write as 
`caesarEncode()`.  You're not entirely wrong, but there is one thing I overlooked that caused me a lot of pain when trying
to debug this.  
When I first wrote this program in Python, it never really occurred to me that if a letter wraps around the alphabet (let's 
say your message is just 'a' with the decode key 'b'), the modulo will break the function and not properly "wrap" everything
nicely around 26.  C and Python handle Modulo division differenly in regards to negative numbers.  In Python, a negative input
in a modulo equation is handled like this: x % n = z, where 0 <= z <= n.  This means that -1 % 26 = 25, not -1. 

In C however, modulo is just modulo.  There's no fancy wraparound functionality and -1 % 26 is just -1.  This causes problems 
for the cipher since we need it to wrap around the alphabet.  'a' - 1 needs to be 'z'.  It took awhile to figure this out both
in Python and in C, but the solution was simply to add my own function called `modWrap()` which solves the problem.  
Here's the function:

    int
    wrapMod(int a, int b)
    {
      if (a >= 0){
        return (a % b)
      } else {
        return (a % b + b) % b;
      }
    }

This is more verbose than it has to be though using the ternary operator here only makes it more difficult to read
than it should be.  

This little detour was all to explain that we need to change the way we handle the `caesarDecode()` function, since 
we're dealing with subtraction.  Put together, we get this:

    static inline char
    caesarDecode(char message, int shift)
    {
      return wrapMod((message - shift - 97), 26) + 97;
    }


## The Vigenere functions

Now on to the actual Vigenere cipher wrapper functions. It shouldn't be too hard to implement now that we've got everything
set up to process the message.  Let's first break down the steps for what should happen to a single character.

The main `encode()` function should contain a loop which calls `caesarEncode()` on each letter with each key from the 
larger passed string.  It's not very hard to keep the key value in the correct range for any lenght of message or key, 
just use `key[i % keyLen]`.  Also, don't forget to add `- 'a'` so the correct shift value is applied instead of the ASCII
value of the character (I made this mistake and spent a bit trying to figure out what was going wrong).  Adding it right to
the `key[]` value saves some time and keeps us from having to add something like `shift -=` 97= at the start of both
`caesarEncode()` and `caesarDecode()`.  

This is what I came up with for the main encode/decode functions:

    char*
    encode(char* message, char* key)
    {
      int msgLen = strLen(message);
      int keyLen = strlen(key);
    
      for (int i = 0; i < msgLen; ++i){
        message[i] = caesarEncode(message[i], key[i % keyLen] - 'a');
      }
    
      return message;
    }

I chose to declare `msgLen` and `keyLen` rather than using the clunkier (and probably slower) `strlen(var)` method, so
`strlen()` is only called once, rather than once per iteration.  The `decode()` function is nearly identical to the 
`encode()` function, though it obviously uses `caesarDecode()` instead. 


# File I/O: I'm probably doing this wrong

The next feature that I want to add is the ability to apply this encode/decode to an entire file.  There are a few
things to plan out before doing this.  How do I want the text itself to be handled?  Should it be one long string
of text or should spaces, tabs, and newline characters be preserved?  For this version, I think I just want to make
everything one long string to be deciphered.  Despite it being less visually appealing, it's sure harder to break
when you don't know where the start/end of a word is.  Also, since this is a plain .txt file, I might add some 80 
character column width in there so it at least comes out as a single block of text instead of one long line of 
text.  

I'm sure there's a better way to handle these kinds of command line arguments, but for now, I'll just add another option
for "-f" and a separate function to handle the file I/O.  Each relevant argv[] parameter will be passed in and the 
function can handle the rest from there.  

I've decided to split up the formatting and encoding/decoding functions into two separate functions.  The first one
`format()` will take in a file name and return a formatted buffer containg the contents of that file.  For now, the 
formatting just removes all spaces, but I'd like to use something that can search for a char in a string without
just looping through every char in that string.  

For now, this is what I've come up with:

    char*
    format(char* fileName)
    {
            FILE *fp;
            fp = fopen(fileName, "r");
    
            if(fp == NULL){
                    printf("Error: Could not open file\n");
                    return 0;
            }
    
            fseek(fp, 0, SEEK_END);
            long int buffSize = ftell(fp);
            fseek(fp, 0, SEEK_SET);
    
            char* buffer = calloc(buffSize + 1, sizeof(char));
    
            if (buffer == NULL){
                    printf("Error: Could not allocate buffer memory\n");
                    return 0;
            }
    
            fread(buffer, sizeof(char), buffSize, fp);
    
            char* formatBuffer = calloc(buffSize + 1, sizeof(char));
            int fBufCounter = 0;
    
            for (int i = 0; i < strlen(buffer); ++i){
                    if(buffer[i] >= 'a' && buffer[i] =< 'z'){
                            formatBuffer[fBufCounter] = buffer[i];
                            ++fBufCounter;
                    }
            }
    
            free(buffer);
            fclose(fp);
    
            return formatBuffer;
    }

This is probably the most complex part of the program (though it's not too bad).  First the funciton figures out
how big the incoming file is and allocates a buffer big enough to hold it all.  It then reads the contents of that file
into the buffer

The easiest way I found to handle formatting without having to modify the original buffer was to just make a new
buffer and copy the desired characters into that, rather than trying to remove spaces from the original.  This means
we just have to keep track of our position in the new buffer and loop through the original, skipping over any characters
we don't want. 

Originally I was going to use a function that would find a character
in a given string, but since I only wanted lowercase a-z characters, there is no reason to make it any harder than it
has to be.  If I were to check for all characters between 'a' and 'z' there would then be no reason to check if that 
character was something in the string "1234567890,./?><" etc.  Having it only add character between 'a' and 'z' should
be alright for the majority of documents. If it becomes an issue, I'll add in something to convert uppercase letters to 
lowercase letters, but for now I think this will work just fine.  This was one case where I really wanted to use 
something but after a bit of thinking found that there's really no benefit to using it. 

Also don't be like me and forget to make the formatting ranges correct.  Make sure they exclude anything outside of the 
'a' to 'z' range, not everything in that range.  Also make sure to use >= and <= so 'a' and 'z' are not formatted out.

And the actual encoding/decoding function looks like this:

    int
    processFile(char* fileName, char* option, char* key)
    {
            FILE *newFile;
            newFile = fopen("Vigenere_Output.txt", "w");
            if (newFile == NULL){
                    printf("Error: Could not open file\n");
                    return -1;
            }
            char* formatBuffer = format(fileName);
    
            if (strcmp("encode", option) == 0){
                    fputs(encode(formatBuffer, key), newFile);
            } else if (strcmp("decode", option) == 0) {
                    fputs(decode(formatBuffer, key), newFile);
            }
    
            free(formatBuffer);
            fclose(newFile);
    
            return 0;
    }

This is actually one of the more straightforward parts of the program.  The formtting was much harder with all the 
buffers.  This just opens a new file, formats the input, then encodes or decodes based off of what the user input
on the command line.  Then it frees the formatBuffer and closes the file.  The only real problem I ran into when 
working on this part was the file name.  If I entered it in as a raw string of text, it worked fine.  As soon as 
I started putting in variables or function returns into there, it would break and not open the file.  

<span class="underline">Example</span>: If I used `fopen(strcat(option, fileName), "w"))` the file would not open and the program would exit without
notice.  If I instead used what is shown `fopen("Vigenere_Output.txt", "w")` it will work fine with no problem. 
I'm not exactly sure how to fix this as of right now, or why it's doing this, but this seems like an alright work around
for the time being.  If it needs to be fixed in the future, I will but it's not that big of a deal if it stays like this.


# Conclusion

This was a fun project and a nice introduction to writing out my thought process as I'm programming. I think it helped
to push me to do things that I might not be completely familiar with and try new things.  If there's something I don't
understand, I can try writing it out here to teach myself along with anyone reading.  It also helps me look more 
critically at what I'm writing.  For this project especially, having to go line-by-line and explain why I did something
or how it works really helped me to consider everything I wrote and why I was writing it. 

For the project, it was easier than I'd thought.  I remember first making this in Python and thinking that it was all
complex string manipulation and crazy modulo division.  Redoing it in C showed me that it's really not that hard at all
and, in fact, my original code was more complex than it had to be (Maybe I should go back and update that original 
repo so it's actually readable).

Overall, I enjoyed this project.  It didn't take very long, there were no real major bugs and for the most part it was 
pretty smooth sailing.  

