---
layout: post
tags: project
title: Expression Evaluation Part 1
---


# Introduction

Probably my favorite aspect of computer science is all the hardware you get to write
programs for.  I love the idea of writing small programs that can get a lot done by 
considering all the little details of a programming language.  I also like the idea
of building something that can really take advantage of the beefiest hardware out there 
(though I don't have much experience with that side of things). 

When I took my first programming class, I remember specifically learning about memory management
in C++.  I thought, "Why would I ever need to even think about memory when my computer has 16gb of
it?"  Well, not thinking about memory management is how you end up writing bad and inefficient 
programs, and what's the fun in doing that?

What is this all building up to?  It's mostly to say that I really like working with lower
level langauges and getting down into the how-and-why of programs.  This is what really 
attracted me to C originally and why I love to use it despite the fact that it's orders of
magnitude more difficult than something like Python.  

Born from this interest in the lower levels of programming (and my degree in linguistics) is
the natural interest in compilers, interpreters, and evaluators.  There's something so interesting
about using our big brains to make a dumb computer box do math, analyze language, or write other
computer programs for us.  It's also pretty crazy learning about the kind of complexity that goes 
into making even the most basic calculator.  

At the heart of this all, how is it that we can make a computer understand something as simple as
2 + 2 = 4? What goes into that process and how does it all work?  I'm by no means an expert at this
but I've been working on a fun little evaluator and I've learned a lot, so why not share it?  

This first post will be a high level overview of the whole process; how it works and what has to be 
done to get a working program.  The next post will cover my implementation of it in C and the usual
dev diary type things.  A 3rd and final post will be my attempt at porting this over into other languages
(maybe something like python, Rust, or Java). 

Enjoy!


# Translating to Postfix

There are two major steps that allow a computer to evaluate an expression.  The first is to translate
the incoming infix expression into a more computer-readable postfix expression.
Postfix notation (or reverse polish notation) places the operators after the operands.  An infix expression
like "1\*2+3" becomes "12\*3+" in postfix.  This greatly simplifies the evaluation process and makes it worth
the trouble.  You'll see how that works in the next section.  Postfix also removes the need to have parenthesis,
since postfix alone keeps track of the order of operations.  

For now, we'll go over the process of converting an infix expression to a postfix expression.  The algorithm 
used for this is called the "Shunting yard" algorithm, named after the way a train shunting yard operates
It was created by the famous Edsgar Dijkstra.  The core of this algorithm, and the entire expression evaluation 
process, is a stack.  We can push and pop different operators and operands to the stack to place them in just 
the right order. Here is a very general overview of how the algorithm works:

    1. Start at the first token of an expression
    2. -> If that token is an operand, add it to the output queue
       -> If that token is an operator, push it to the stack
       -> If that token is a left paren, push it to te stack
       -> If that token is a right paren, dump the stack up to the left paren
    3. After all tokens have been read, dump the stack to the output
    4. Return the new postfix expression

One very important thing to mention: Whatever you do with the operators must take operator precedence into account. 
Operator precedence tells the program that \*/ should be evaluated before +-, essentially PEMDAS (or whatever you might
have learned).  Without this, the postfix translator will not work properly.  I'll explain how to implement this in 
when I go over the actual C code for this project. 

Here is that algorithm applied to a simple expression, "(1+2)\*3-4"

    1. TOKEN: '('
    Left paren, push to stack
    2. TOKEN: '1'
    Operand, add to output
    3. TOKEN: '+'
    Operator, push to stack
    4. TOKEN: '2'
    Operand, add to output
    5. TOKEN: ')'
    Right paren, dump stack up to '(' to output

At this point, the stack is empty and our output looks like "12+"

    6. TOKEN: '*'
    Operator, push to stack
    7. TOKEN: '3'
    Operand, add to output
    8. TOKEN: '-'
    Operand, current operator at top of stack 
    has higher precedence.
    - pop stack and add to output
    - push '-' to operator stack
    9. TOKEN: 4
    Operator, add to output
    ---
    All tokens parsed, dump operator stack to output

The final result of this would be "12+3\*4-"


# Evaluating the Expression

Once the expression has been translated into postfix notation, the evaluation process is extremely straightforward by 
comparison.  The evaluation step
is made so straightforward because of all the work we did in the first step.  This should make it clear why we had to go 
through the process of converting
the infix expression into postfix.  

We can evaluate a given postfix expression using this algorithm:

    1. Read in a token from the expression 
    
    2. -> If that token is a number, push it onto the stack
       |
       -> If that token is an operator, pop the required number 
          of tokens from the stack and evaluate, then push the 
          result back onto the stack.
    
    3. Pop the final element from the array and return it

This algorithm shows why postfix notation is so important.  Because each operator follows its operands, we can easily
push the operands onto a stack until we hit their operator.  We then just pop off enough operands from the stack and
plug those into some function that corresponds to the operator.  Here's an example of it at work on an actual
expression, lets say "123\*+":

    1. TOKEN: 1
    - Push to operand stack
    2. TOKEN: 2
    - Push to operand stack
    3. TOKEN: 3
    - Push to operand stack
    4. TOKEN: *
    - Pop 3 and 2, solve for 3 * 2 and 
      push 6 onto the operand stack
    5. TOKEN: +
    - Pop 6 and 1, solve for 6 + 1 and
      push 7 back onto the stack
    6. END OF EXPRESSION
    - Pop 7 from the stack and return it.


# Conclusion

This was a somewhat in-depth overview of everything you need to know before writing an expression evaluator. 
The next part will cover the implementation in C and will most likely review these points as they come up in
the code.  

