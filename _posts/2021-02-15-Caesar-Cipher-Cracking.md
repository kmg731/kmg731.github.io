---
layout: post
tags: project
title: Caesar Cipher Cracking
---


# Introduction

This is less of a full fledged project and more of a little thing that I did because
I thought it would be interesting.  It's an extra little module to eventually integrate
into the Caesar/Vigenere cipher I made a few months back.

I learned most of how this works from [this blog post](https://mitchellkember.com/blog/post/caesar-cipher/).  The code for the article was
originally done in Haskell, but I did it on my own in C (as usual), which led to having
to rethink a few of the implementations.
I'll go over the process I followed, any new info I found, and what I learned from the project.

Overall, it's a really easy tool to make and shouldn't take more than an hour or two to
finish.  It took me about 3 hours, but I kept getting weird segfaults on the encrypt/decrypt functions
which took longer to diagnose than actually writing the main cracking function.


# Cracking the Code

The caesar cipher is a fun little thing to play around with but doesn't exactly have much practical
use because of how easy it is to crack.  It's not very difficult to imagine how to crack any given
code.  You'd only have to take the code (or a subset of the code depending on how long it is) and
run through each of the 25 possible shifts until you hit one that makes some sense.  It wouldn't be
very exciting, but I don't think it would take longer than 20 minutes to break any given code.

We're going to use a variation on this method, which involves doing a character frequency analysis
on each of the shifts for a given message and determine which of those character frequencies best
fits the known character frequency of English.

Since we've got computers, we can make this process a little more fun.  Our implementation will follow
that rule, but also use a fun statistics trick to get the computer to pick the cipher key that results
in actual English, rather than making us do it.

I should mention that this method of cracking works best the longer the text is.  There's a very low chance
it will work on a single word, like "text," but with something as long as an average email or letter, it'll
work fine.


## Structure

We're going to start off the cracking with the "try every shift key" approach.  We'll run a character frequency
analysis each of the keys and keep that handy for the next step.
The character frequency analysis sounds like it'd be really complicated, but it's really just saying "find how
many times a given character appears in some text."  The implementation is very easy, even in C:

    static float*
    characterFrequency(char* string){
            float* charFreq = (float*)calloc(1, sizeof(float) * 26);
            int len = strlen(string);

            strclean(string);
            for (int i = 0; i < strlen(string); ++i){
                    if (islower(string[i])){
                            ++charFreq[string[i] - 'a'];
                    } else if (isupper(string[i])){
                            ++charFreq[string[i] - 'A'];
                    }
            }

            for (int i = 0; i < 26; ++i){
                    charFreq[i] = (charFreq[i] / len) * 100;
            }

            return charFreq;
    }

All this function is really doing is pulling out the characters of the alphabet and adding 1 to the
corresponding entry in the `charFreq` array, which gives us the corresponding frequency of any character a-z
in a string.  Once we've figured out how many of each character occurs in a given string, we can then
turn those entries into a percentage of the total string.  This is required for the next part.

`strclean()` is just a function that removes all non-alphabet characters from a given string.  We need this since there's
no reason to count the frequency of spaces or punctuation.

The next function to go over is `chiSqr()` which is a probability function that takes in two datasets (expected and
observed) and returns some value for how well the two datasets match.  This is obviously very useful in finding the
shift key used since the character frequency that best fits the known character frequency of English will be the
shift key we're looking for.

    static float
    chiSqr(float* os){
            float totalSum;

            for (int i = 0; i < 26; ++i){
                    totalSum = totalSum + ( pow(os[i] - FREQ_TABLE[i], 2.0) / FREQ_TABLE[i]);
            }

            return totalSum;
    }

We can get away with cutting this function down, since neither dataset will ever exceed 26 characters and
the expected set is known (shown by the `FREQ_TABLE` constant).  If you were going to implement a proper
`chiSqr` function, it might be a bit more complex, but this works for our purposes.

The actual cracking function is something that I've found quite amusing.  I'm not sure it's the best or most
efficient, but the way everything comes together is really satisfying.  Here's the final function:

    int
    crack (char* string){
            float shiftTable[26];

            for (int i = 0; i < 26; ++i){
                    shiftTable[i] = chiSqr(characterFrequency(decrypt(string, i)));
            }

            return findMin(shiftTable, 26);
    }

This function takes in some encrypted string and returns the shift key used to encrypt it.  A better function
might be one that returns the fully decrypted string, but I thought this was a bit more general.  This allows
you to crack some string or file with only `decrypt(string, crack(string))`, which I think works quite well.


# Final Thoughts

This was a fun little exercise to try out.  This was the first time I'd tried some kind of codebreaking project
like this and it's been really cool. I like that there's more math in this than what I usually do so I'm excited
to try out some more codebreaking projects.  The next thing I'll be working on is a program that cracks the
Vigenere cipher.  That way, my Vigenere cipher will have a full toolset to not only make and decrypt Vigenere
ciphers, but also crack any you may come across.

The primary bottleneck of this program is the `characterFrequency()` function, which slows the program down
significantly the larger a document gets.  Though, I don't see any way this could be improved since, by
definition, character frequency has to visit every element in a given dataset, giving it a linear runtime.
One optimization that I did find was that I was using `strclean()` too many times, which slowed everything down
greatly.  In trying to make a general Caesar encrypt/decrypt function, I used `strclean()` in both functions,
without noticing that that was adding an extra O(n) function into something that only needed to be done once.
I think, overall, it's better to have to functions that are meant just for a specific domain, than having two
more general functions that slow down your program overall.

The biggest difficulty in making this was the stupid Caesar encrypt/decrypt functions.  For whatever reason,
when I wrote the Caesar encrypt/decrypt functions in the same way as the Vigenere encrypt/decrypt functions,
I'd get segfaults and nothing would work.  I eventually gave up and made it return a new string containing the
encrypted string which, when doing something like this doesn't seem memory efficient *at all*, but it works for
now and I'll try to optimize it as best I can going forward.  I don't know how much memory management really
matters for a program that only runs for maybe a second, but I suppose it's always good to employ proper
memory management practices.  At the very least, it'll give me something to work on with this project.

The next project I'll be doing is definitely a Vigenere Cipher cracking tool.  I think it'll be fun to explore
these a bit more and compare the difficulty of cracking the Caesar cipher to cracking the Vigenere cipher.
