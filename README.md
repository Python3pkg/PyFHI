# PyFHI
-------

!!!DO NOT USE!!!

PyFHI was written before I was aware of such concepts as garbage collecting
and libraries like [contextlib.ExitStack](https://docs.python.org/3/library/contextlib.html#contextlib.ExitStack). PyFHI is short-sighted
and can result in undesireable behaviors like premature closing of all file
handles. See [this Reddit post](https://www.reddit.com/r/Python/comments/3o1clp/pyfhi_python_file_handling_improved/)
for details on the dangers adn alternatives to PyFHI.

# Old Docs
----------

PyFHI (Python File Handling Improved) improves how Python handles files and
such that all files are closed in case a program crashes without the need
of *with* statements, code is less indented and more practical, and script
portability between Python 2 and Python 3 is improved. More details on
PyFHI's functionality can be found in [ABSTRACT](ABSTRACT.md) or [my blog]
(https://baas-becking.biology.utah.edu/cjuserprofilelayout/987:ahyer?view=profile).

The official source code repository is hosted on [GitHub]
(https://github.com/TheOneHyer/PyFHI).

## Pronunciation
----------------

Officially: "pie-fi" [pahy-fahy]

Colloquially: "Pi-Fi" as in "Wi-Fi" but with a "Pi"

## Installation Instructions
----------------------------

pip install pyfhi

## Usage
--------

PyFHI contains two parts: the *Open* class and *filecloser* function/wrapper.
It is the combination of these two elements that give PyFHI the benefits listed
above. The *Open* class is a wrapper of Python File Objects, so it can 
do anything a Python file handle can normally do. Below is an explanation of 
the various benefits of PyFHI beyond the use of normal file handles and how to 
use them.

### Closing All Open File Handles

PyFHI's *Open* class tracks all open file handles. It also contains a static
method called *close_all* that closes all open file handles. Thus, in order to
close all open file handles, one simply needs to type:

    Open.close_all()
    
### Python 2 and Python 3 Cross-Compatibility

Python 2 uses the *next* method to get the next line ine a file while Python
3 uses *'__next__'* to accomplish the same task. This difference only matters
if a developer is calling this method manually (rare). However, if a 
programmer is calling these methods manually, then the program will crash if it
is run with a version of Python other than intended. *Open* offers both 
methods and each method will detect the runtime Python version and execute 
the correct method call.

### File Printing

If one prints a file variable they get something like this:

    >>>print(file_handle)
    
    <open file 'file_name', mode 'r' at 0x7f2f7d48c540>
   
Which is ugly and mostly useless. The same thing in PyFHI yields:

    >>> print(file_handle)

    File Name: [file_name]

    File Mode: r

    File Encoding: UTF-8

    File Closed: False

### File Closing

The cornerstone of PyFHI is it's ability to cleanly hold onto and track 
file handles throughout a program and to close them all cleanly upon 
termination, even if the program crashed. This is accomplished by tracking 
all open file handles with the "Open" class and removing closed file handles. 
As mentioned prior, *Open* has a static method to close all open file 
handles. The *filecloser* function wraps a function such that when the 
function ends, or if there is crash, *Open*'s *close_all* method is called, 
cleanly closing all open file handles. A more detailed explanation of why this
is relevant can found in [ABSTRACT](ABSTRACT.md).

In order to reap the benefits of PyFHI file closing, a developer must do the 
following three things:

1. Add the following import statements to their script (note the capital "O"):
  * from pyfhi import filecloser
  * from pyfhi import Open
2. Changes all instances of opening files with *open* to *Open*. Note that 
this doesn't work/apply to things like *os.open*, just plain *open*.
3. Add *@filecloser* before any function to close all files after the 
function is complete.

Two important points concerning PyFHI usage follow:

#### Opening Files

One of the goals of PyFHI was to clean up code and avoid needless indents. As
such, PyFHI was constructed such that "with" statements are no longer 
necessary. So instead of opening three files like this:

    with open([file1]) as fh1:

        with open([file2]) as fh2:

            with open([file3]) as fh3:

                # Do stuff here

One can simply do:

    fh1 = Open([file1]) # Note the capital "O"

    fh2 = Open([file2])

    fh3 = Open([file3])

To get the same results. Best part is that all the files will still be deleted
cleanly if there is a crash anywhere in the program (assuming *filecloser* is
used)!

#### Full-Program Utilization/Wrapping

The greatest benefit of PyFHI is the ability to close all files opened by a 
program no matter where in the program a crash occurred. In order to 
accomplish this, a developer must put the bulk of their program in a function
so that the *filecloser* wrapper can wrap the program. This is best/most 
universally done by writing your script in a function called "main()" and 
calling it at the end of the script as so:

    def main():

        # Do your main program stuff

    if __name__ == '__main__':

        main()
        
Then wrap this method as shown in Usage Example.

### Usage Example

    from pyfhi import filecloser
    from pyfhi import Open

    @filecloser
    def main():

        # Do your main program stuff

    if __name__ == '__main__':
        main()

## Open Issues
--------------

There are no known open issues with PyFHI. Please submit all bugs to
theonehyer@gmail.com.

## License
----------

GNU General Public License v3 (GPLv3)

See LICENSE for details

## Authors
----------

Alex Hyer (theonehyer@gmail.com)