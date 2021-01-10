---
layout: post
tags: project
title: Expression Evaluation Part 2
---


# Introduction

This is part two of a three part series on expression evaluation.  In this part, I'll 
go over the actual code implementation of an expression evaluator in C.  Part three will
(hopefully) be about porting this over to different languages.  

I'm by no means an expert on this.  It took me a few weeks to just wrap my head around the
whole process and then a bit longer to actually get everything working.  This implementation
is by no means perfect.  There should really be some kind of token system so this will take 
more numbers than just 0-9.  There should be associativity so we could add more operators 
instead of just having add, subtract, multiply, and divide.  The evaluation section should
probably not use so many if-else statements.   

The code I wrote is a modified version of the code provided at the end of Ch.4 in Robert LaFore's 
book *Data Structures and Algorithms in Java*.  I started with a direct translation of the code, 
but that ended up being a complete failure so I had to heavily modify it so it would actually function.


# Writing the translation functions

I should start by saying that I won't be going over how to write a stack/queue library.  They're not 
extrememly difficult to implement and this isn't really meant to be about how to write those.  If you'd 
like to see them, check my github page for the full library files.

The first step to getting a computer to solve an equation is to translate it into postfix.  To do this, 
we'll implement a scaled down and modified version of the shunting yard algorithm we talked about in the
last post.  

The translator itself is composed of just two functions, `translate()` and `processOp()`.  `translate()`
is, obviously, the main function for this process.  The biggest challenge when trying to sort this out 
was not so much in the implementation, but moreso in C itself.  Unlike in most languages, C does not have
the \\+= operator overloaded to work with strings.  \\+= will very nicely add and set ints, but there's no 
way to use them with strings and get the functionality you'd expect in Java, Python or C++.  Because of this,
I decided that my two hours of work would not go to waste and I would use the queue library I wrote to 
make it a bit easier on myself.  There is definitely a better way to do this but global variables scare me and
I'm not about to add in more loops and parameters to some of these functions. 

Here is the actual implementation of the `translate()` function:

    
    char*
    translate(char* expression, int len){
    
            // The main stack 
            Stack* st = createStack();
            Queue* q = createQueue();
            char* output = (char*)calloc(1, sizeof(char) * len);
            char parenChar;
    
            for (int i = 0; i < len; ++i){
                    // TODO: Add error checking for mismatched parens
                    char ch = expression[i];
    
    
                    switch(ch){
                    case '+':
                    case '-':
                            if(S_isEmpty(st)){
                                    push(st, ch);
                                    break;
                            }
                            processOp(st, q, ch, 1);
                            break;
    
    
                    case '*':
                    case '/':
                            if (S_isEmpty(st)){
                                    push(st, ch);
                                    break;
                            }
                            processOp(st, q, ch, 2);
                            break;
    
    
                    case '(':
                            push(st, ch);
                            break;
                    case ')':
                            while((!(S_isEmpty(st)) && (parenChar = pop(st)) != '(')){
                                    enqueue(q, parenChar);
                            }
                            // TODO: add error for mismatched parens
    
                            break;
    
                    default:
                            enqueue(q, ch);
    
                    }
            }
    
            // Dump the contents of the stack to the queue
            while(!(S_isEmpty(st))){
                    enqueue(q, pop(st));
            }
    
            // Dequeue the output queue to an output string
            int i = 0;
            while(!(Q_isEmpty(q))){
                    output[i] = dequeue(q);
                    ++i;
            }
    
            return output;
    
    }

This function starts out as you would expect.  We first declare all of our necessary variables with the exception
of one, `parenChar`.  This is declared up here because of a very weird exception I didn't learn about until just
now, C will not let you declare a variable inside of a `case` statement. I'm not sure why, but this gets around
that.  The next step is to iterate through each character of the output string.  Ideally, this would be an array of 
tokens but for this simple implementation it is just a regular string.  
We then run the current character through a `switch/case` block.  For any operand, we just add it to the output
queue.  If the current character is an operator, we've got to go through some extra steps to handle operator precedence.

A quick detour for `processOp()`:

    void
    processOp(Stack* opStack, Queue* q, char incoming, int incPrecedence){
    
            char opTop = S_peek(opStack);
            int topPrecedence;
    
            if (opTop == '*' || opTop == '/'){
                    topPrecedence = 2;
            } else if (opTop == '+' || opTop == '-') {
                    topPrecedence = 1;
            } else {
                    topPrecedence = 0;
            }
    
            if (incPrecedence > topPrecedence ){
                    push(opStack, incoming);
    
    
            } else if (incPrecedence < topPrecedence ){
                    char toAdd = pop(opStack);
                    enqueue(q, toAdd);
                    push(opStack,incoming);
            }
    }

This is where the next function comes in.  `processOp()` takes in four arguments: the main stack from the `translate()`
function, the output queue from `translate()`, the incoming operator, and the precedence of that incoming operator.  
This function takes the current top of the operator stack (`opTop`), sets its precedence, then compares that to the
incoming precedence.  The part that took me much longer than I thought to figure out was the precedence comparison.
For some reason, I kept going back and forth on what this was actually supposed to do.  Finally, I sat down and just
worked it out.  If the incoming precedence is greater than the precedence of the operator at the top of the stack, 
just push the operator to the stack.  This means that if the current top is '+' and the incoming is '**' the order
of operations is maintained if we just push '**' to the stack.  

Alternatively, if the current top is '\*' and the incoming operator is '+', just pushing it on would not maintain 
the order of operations.  We have to pop the operator from the stack, add it to the output queue, then push the
incoming operator to the stack.  This step makes sure that "2\*2+3" is translated as "22\*3+" rather than "223+\*"
which would be equivalent to "2\*(2+3)".  

Back to `translate()`:

The next thing to cover is how the parenthesis are handled.  If we find a left parenthesis, we just push it to the
stack.  We don't need to think about operator precedence, since anything inside the parenthesis will be evaluated 
on its own regardless of the stack contents before the parenthesis.  
Once the left parenthesis has been pushed to the stack, we continue on as normal.  The catch here is what happens
when we hit the right parenthesis.  When this happens, we dump the contents of the stack to the output until we
hit the left parenthesis.  This might be my favorite part of the program because it handles something that seems
very complex in a very simple way.  It's easy to visualize.  

Imagine we're trying to evaluate (2+2)\*3:

    1. TOKEN: '('
    -> Push to the stack and move to the next token
    2. TOKEN: '2'
    -> Add to output queue
    3. TOKEN: '+'
    -> Push to the operator stack
    4. TOKEN: '2'
    -> Add to output queue

Before we hit the righ parenthesis, let's see what the operator stack and output queue look like:

    Stack:
    | + |
    | ( |
     ---
    Queue:
     ---
     2 2
     ---

Now, let's process ')'

    TOKEN: ')'
    -> Right parenthesis, dump stack to output
       and discard '('

The output queue now contains "22+".  When the rest of the expression is processed, we're left with "22+3\*" which 
is the correct postfix translation for (2+2)\*3.

The final two steps don't need much explanation.  There are two while loops which handle the last steps of the 
translation.  The first loop dumps the final contents of the stack to the queue, and the second loop converts
the stack into a string.  I could probably add this as its own queue library function, but I'm just leaving it 
like it is for now. 

That covers translation, next is actually solving the equation.


# Evaluation

The evaluation step is much more straightforward than the previous step.  There is not nearly as much abstract
stuff you need to wrap your head around.  Evaluating an expression is almost like doing the translation but in 
reverse.  Rather than pushing operators to a stack, you instead push the operands to a stack and pop them off
when you hit an operator.  Here's how it works:

    1. Iterate through each element of the translated expression
    
    2. -> If that element is an operand, push it to the stack
    
       -> If that element is an operator, pop the necessary elements
          needed for that operator, do the operation, and push the 
          result of that operation back on the stack.
    
    3. Return the result 

This stack approach is why we had to go through the initial translation step.  Since operands preceed their operators
in postfix, we're able to quickly and easily use a stack to evaluate.  Here's an example of this evaluation algorithm
at work with the expression "123\*+"

    1. TOKEN: 1
    - Push onto operand stack
    2. TOKEN: 2
    - Push onto operand stack
    3. TOKEN: 3
    - Push onto operand stack
    4. TOKEN: *
    - Pop two operands from the stack and evaluate
    --> 3*2 = 6
    ---> Push 6 onto the stack
    5. TOKEN: +
    - Pop two operands from the stack and evaluate
    --> 6 + 1 = 7
    ---> Push 7 onto the stack
    6. END OF EXPRESSION
    - Pop and return the last number on the stack

It's worth taking a bit to think about how this process would be different if we stuck with infix instead of going 
through the translation process.  

I think my implementation of this step is what most would have come up with, maybe with the exception of the
`push(st, add(pop(st), pop(st)))`.  That probably isn't best practice, but I haven't been told otherwise, so 
yell at me if it's wrong. 

    int
    parse(char* expression){
            Stack* st = createStack();
    
            for (int i = 0; i < strlen(expression); ++i){
                    char ch = expression[i];
    
                    switch(ch){
    
                    case '+':
                            push(st, add(pop(st), pop(st)));
                            break;
                    case '-':
                            push(st, sub(pop(st), pop(st)));
                            break;
    
                    case '*':
                            push(st, mult(pop(st), pop(st)));
                            break;
                    case '/':
                            push(st, divide(pop(st), pop(st)));
                            break;
    
                    default:
                            push(st, ch - '0');
    
                    }
            }
    
            return pop(st);
    }

This function overall is very straightforward.  We've only got a stack and one case for each operator.  When the loop
enounters an operator, it pops two elements, then pushes the result of the operator back onto the stack.  


# Conclusion

This project was not as difficult as I originally anticipated, but it was also not very easy.  It took more research
than any other project and a few days of tinkering with the details to get everything working right.  If anything, 
the hardest part was just finding the right resources to get an understanding of how everything should work.  So
much of the stuff I would find would tell me that I needed to keep track of operator precedence and associativity
without giving any kind of concrete implementation.  
Eventually, I just went back to basics and read through the section in Lafore's book and the example java code gave
me a good jumping off point to see how this whole process is implemented in code.  

Hopefully I'll get around to making a part 3 for this.  I might have to take a break for a bit before I get the energy
to do this all again in Python or Rust.  

Thanks for reading!  

