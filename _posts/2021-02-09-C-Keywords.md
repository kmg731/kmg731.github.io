---
layout: post
tags: non-project 
title: C Keywords Explained
---
# Foreword

I recently interviewed for a job to do some C development.  In the screening, the interviewer
asked me to define some C keywords.  I knew that this was something I should know and was kicking
myself immediately.  Long story short, I bombed the interview.  This is my attempt at making it 
right by (hopefully) explaining the keywords that I had not heard of or wasn't completely sure on.

There's not much of a reason to define all 32 keywords, since we all know what keywords like "int,"
"double," and "char" do, but maybe not everyone knows what "extern," "inline," or "register," do.


# Keywords


## Auto

This is a storage-class specifier.  It declares how the variable it is attached to will be 
handled.  The `auto` keyword says that this will be allocated when it comes into scope and 
deallocated when it exits scope.  There is not much of a reason to ever use this, since the
default behavior for variables is `auto`


## Const

This keyword simply means that the contents stored inside the variable, whether that be a pointer,
a string, an int, float, double, etc. cannot be modified.  If a variable is declared as 
`const int x = 10` then `x` will always have the value of 10.  In the same way, if a pointer is 
declared as `const int* ptr = &x` that pointer may only ever hold the address of the variable `x`.


## Extern

Another storage-class specifier.  `extern` tells the compiler that this function or variable exists
somewhere else in the program.  Similar to the `auto` keyword, there's not much of a reason to use
it in your program since all functions in C are implicitly declared as `extern`.  


## Inline

This keyword tells the compiler to do an inline substitution on whatever it is you're declaring 
it with. This can have the advantage of speeding up your programs (as per the gcc entry on `inline`)
If I'm writing a program  that relies on heavy usage of a few small functions, I'll usually add 
`inline` to them to speed things up a bit.


## Restrict

This keyword can be used to add compiler optimizations when working with pointers.  `Restrict` tells 
the compiler that you, the developer, will make sure that the pointer declared with `restrict` will
not be aliased in any way.  This allows the compiler to optimize away certain safety checks and speed
up the code a bit.


## Register

Another storage-class specifier.  This one tells the compiler that you want this variable
to be stored in a CPU register rather than in memory.  This allows for much faster access
of that variable.  Though since the variable is stored in a CPU register and has no memory 
address, there is no way to declare a pointer to that variable.  This can prevent aliasing.


## Static

Another storage-class specifier.  When used with a variable inside of a function, it tells 
the compiler to keep this variable in memory and to not destroy it when the function ends.
A good example of how this is used would be the simple program:

    void addOne(){
      static int a = 0;
      printf("%i ", ++a);
    }
    
    int main(){
      addOne();
      addOne();
      addOne();
    
      return 0;
    }

With the `static` keyword, the output is `1 2 3`, without the static keyword, the output 
would just be `1 1 1`.

When used with a function, the static keyword works as the opposite of `extern`, it limits
the usage of the function or variable to just the file it was defined in.  This allows us
to declare different functions with the same name across multiple source files. 


## Volatile

This keyword tells the compiler that the piece of data it's attached to is, well, volatile and
should not be optimized in any way.  There may be times when the compilers optimizations break
your code (the domains seem to be peripheral registers, interrupt service routines, and 
some multithreading) and you'd like to turn off all optimizations for a specific variable or
set of variables.  

