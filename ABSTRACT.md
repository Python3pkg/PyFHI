This file is essentially just a copy and paste of my 
[original blog article](https://baas-becking.biology.utah.edu/cjpython/98-python-file-handling-improved-pyfhi)
for PyFHI. The difference between this file and said blog article is that the
improvements to Python file handling are found in [README](READEME.md) so this
file will mostly focus on the reasoning for why PyFHI needed to be created and
why it is important. We begin with a review of what file handles are and open
issues with them.

# File Handles
--------------

As a point of review, files are data on a storage device such as a hard 
drive or flash drive. Files are frequently found in two formats, text or 
binary. Simply put, text files contain human readable data and binary files 
contain data solely readable by computer (or computer science demi-gods). 
Python provides file handles which greatly simplify the access to files for 
programmers. A file handle can be declared/opened as simply as:

    file_handle = open(file_name)
    
Once a handle is open, interacting with the file is easy. A few examples
follow: 

    lines = file_handles.readlines() # Stores the entire string as a single 
    file
 
    for line in file_handle: # Reads the file one line at a time

        # Do stuff with each line of the file

 
    file_handle.close() # Close the file
    
Compared to many other languages, file handles in Python are simple, intuitive,
and versatile. What could possible be wrong with such a system?

# The Problem With Python File Handles
--------------------------------------

We'll start this journey with older Python file handle issues that have been
fixed in order to make sense of the current problems and why they're relevant.
Additionally, discussing older problems may teach some readers problems and
fixes they were previously unaware of. 

## Problem #1: Crashes

When Python has a file handle open when a script crashes/throws an error, the
program is abruptly aborted and the file is not properly closed. The file
handle is now useless but will occupy memory and potentially cause variable
name clashes. Python does have a garbage collector to reuse unused memory but
this may be a while so it is good programming practice to not rely on the
garbage collector and consume memory needlessly. One fix to this problem is to
use a *try...finally* block:

    try:

        file_handle = open(file_name)

        # Do stuff

    finally:

        file_handle.close() 

This guarantees that if a Python program throws an error from within the try
block, the finally block be executed. In this instance, we use the finally
block to close all our file handles. Problem fixed, right?

Well, while the above code is functional, it's not always very pretty and we
have to manually remember to put the closing of each file in the finally 
block. We also have to use a new try...finally block whenever we close a file
and then open another one. In summary, this goes against one of Python's core
philosophies of being simple to read and clean-looking. Guido van Rossum and
Nick Coghlan proposed the "with" statement in
[PEP 343](https://www.python.org/dev/peps/pep-0343/). The with statement
simplifies and cleans up the previous block of code as follows:

    with open(file_name) as file_handle:

        # Do stuff

Notice how we don't even have a file_handle.close() statement. This is because
if any error is ever thrown inside the with block, or when the with block is
finished, Python will close all file handles cleanly. This later fix is by far
more common and recommended in all code. Unfortunately, it is not applicable
in all situations.

## Problem #2: File Checking and the Race Condition

Many Python programs, or just programs in general, check whether certain files
given to the program by the user to read or manipulate even exist. This is
especially important for programs performing computationally intensive work
(such as bioinformatics) because it is pointless to spend hundreds of CPU
hours just to discover that a user doesn't have permission to write the output
file in the directory they provided. As such, many programs attempt to open
files immediately to ensure they're usable:

    with open(file_name) as file_handle:

        pass 

The program then opens the file in a *with* statement again when it comes time
to actually use the file.

The problem with this approach is that a file can be moved or deleted between
when the file was verified and when the file is actually used. This is a
problem known as the
[race condition](https://en.wikipedia.org/wiki/Race_condition).
In a nutshell, the race condition is when two processes need to use the 
same data but one process modifies, locks, or deletes the data before the other
process can access it. For small files (on modern computers, I would deem 
less than 0.5 Gb small enough, but it's up to you), there is a simple fix: 

    with open(file_name) as file_handle:

        lines = file_handle.realines() 

This reads the entire file into a single variable while inside a *with*
statement. Thus the whole file's contents are always available to the program
forevermore and any errors with access will result in Python cleaning up after
itself. However, this fix is not practical for large files such as metagenomes
(one can mess with file locking but this is never really guaranteed).

## Problem #3: Keeping the Code Pretty

One way to keep a hold of a large file from the beginning of the program is to
open it and keep it open which can be done as follows: 

    with open(file_name) as file_handle:

        # Rest of your program here 

This is fully functional but causes the entire program's code to become
indented once per file. This quickly results in ugly, hard to follow code. For
example, if I want to keep three files open throughout my program:

    with open(file_name1) as file_handle1:

        with open(file_name2) as file_handle2:

            with open(file_name3) as file_handle3:

                # Rest of program here 

Using a *try...finally* statement may result in fewer indents but still 
requires all the code to be indented once and for the program author to 
manually close all file handles in the finally block. If we just open the 
files in the most basal form: 

    file_handle = open(file_name) 

    # Do program stuff 

    file_handle.close() 

then we run back into the original problem with Python file handles. How do we
make a program keep a hold of a large file, clean-up cleanly, and look pretty?

# PyFHI To The Rescue
---------------------

As is explained in the [README](README.md), PyFHI effectively fixes all the
issues and even adds functionality to traditional file handles. This is done
through the *Open* class and *filecloser* wrapper/function:

## Open Class

The *Open* class is a fully-featured Python File Object wrapper that exactly
mimics the API of Python File Objects. This means that users can open files
with *Open* and use it as they would a normal file handle. This makes it easier
to update old scripts to use PyFHI and keeps the program just as readable as
before. The *Open* class also introduces a static method *close_all* to close
all open file handles. Finally, as detailed in the [README](README.md), *Open*
improves on many other aspects of Python file handles.

## filecloser Function

The function *filecloser* is a wrapper, meaning that it is a function that
returns a modified function. Specifically, *filecloser* inserts a given
function into a *try...finally* block where the *finally* block simply calls
*Open.close_all()*. Therefore, if your program exits or crashes anywhere
within a function, all your files will be closed cleanly.

# Conclusion
------------

As with all things computers, there are non-intuitive intricacies that
prevent optimal function. In the case of Python file handles, it has been
previously impossible to both maintain access to a file and crash a program
cleanly without reading the entire file into memory; this fix is obviously
impractical for large files associated with many applications including
bioinformatics. PyFHI has efficiently solved all these problems and added
functionality to Python file handling for very little effort on the part of
the developer.