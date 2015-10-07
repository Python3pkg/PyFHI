# PyFHI

PyFHI (Python File Handling Improved) improves how Python handles files and
such that all files are closed in case a program crashes without the need
of "with" statements, code is less indented and more practical, and script
portability between Python 2 and Python 3 is improved. More details on
PyFHI's functionality can be found in ABSTRACT or [my blog]
(https://baas-becking.biology.utah.edu/cjuserprofilelayout/987:ahyer?view=profile).

The official source code repository is hosted on [GitHub]
(https://github.com/TheOneHyer/PyFHI).

## Pronunciation

Officially: "pie-fi" [pahy-fahy]

Colloquially: "Pi-Fi" as in "Wi-Fi" but with a "Pi"

## Installation Instructions

pip install pyfhi

## Usage

In order to reap the benefits of PyFHI, a developer must do the following 
three things:

1. Add the following import statements to their script (note the capital "O"):
  * from pyfhi import filecloser
  * from pyfhi import Open
2. Changes all instances of opening files with "open" to "Open". Note that 
this doesn't work/apply to things like "os.open", just plain "open".
3. Add "@filecloser" before any function to close all files after the 
function is complete.

Three important points concerning PyFHI usage follow:

### Opening Files

One of the goals of PyFHI was to clean up code and avoid endless indents. As
such, PyFHI was constructed such that "with" statements are no longer 
necessary. So instead of opening three files like this:

> with open([file1]) as fh1:
>
>   with open([file2]) as fh2:
>
>       with open([file3]) as fh3:
>
>           # Do stuff here

One can simply do:

> fh1 = Open([file1]) # Note the capital "O"
>
> fh2 = Open([file2])
>
> fh3 = Open([file3])

To get the same results. Best part is that all the files will still be deleted
cleanly if there is a crash anywhere in the program!

### Full-Program Utilization/Wrapping

The greatest benefit of PyFHI is the ability to close all files opened by a 
program no matter where in the program a crash occurred. In order to 
accomplish this, a developer must put the bulk of their program in a function
so that the "filecloser" wrapper can wrap the program. This is best/most 
universally done by writing your script in a function called "main()" and 
calling it at the end of the script as so:

> def main():
>
>   # Do your main program stuff
>
> if __name__ == '__main__':
>
>   main()

### Usage Example

> from pyfhi import filecloser
>
> from pyfhi import Open
>
>
> @filecloser
>
> def main():
>
>   # Do your main program stuff
>
> if __name__ == '__main__':
>
>   main()

## License

GNU General Public License v3 (GPLv3)

See LICENSE for details

## Authors

Alex Hyer (theonehyer@gmail.com)