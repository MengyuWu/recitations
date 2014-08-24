# Recitation B #

## Makefiles ##

Make is a UNIX utility that follows a blueprint you create for compiling
programs. Calling `make` will automatically search your current directory for a
file called "Makefile" and use it to call various compiler commands according 
to the rules outlined therein. 

### Jae's myadd Makefile ###

Take Jae's Makefile piece by piece. It can be found in this git repository as
`sample-makefile`

```make
CC  = gcc
CXX = g++
```

Make has a some pre-configured rules for how to compile programs. For example 
itknows how to specify files as arguments to a compiler. However, you should 
tell it what compiler to use for C files and C++ files. Here, we set the
special make variables CC and CXX to gcc, the C-compiler, and g++, the c++
compiler.

```make
INCLUDES =

CFLAGS   = -g -Wall $(INCLUDES)
CXXFLAGS = -g -Wall $(INCLUDES)
```

Here we define our own variable, INCLUDES, which we can use for directories 
thatwe wish to include at the compilation step. An example value for INCLUDES 
could be `-I../myHeaders` which would tell the compiler to look in the myHeaders directory, located one directory above the current directory, during the compilation step for missing header files and other
sorts of relevant files. For this class, please do NOT use absolute paths in your Makefiles; we do not have the permissions to access your /home/your_uni directory (you'll learn about permissions later on).

After defining INCLUDES, we define the flags that we want each compiler to be
run with. In this case we include the `-g` flag for debugging and `-Wall` flag
to display all warnings. Lastly, we reference our variable INCLUDES to add 
those flags as well.

```make
LDFLAGS = -g
```

LDFLAGS are the flags that are appended to the compiler when using it for
linking. In this case we just want the debugging info to be included.

```make
LDLIBS =
```

LDLIBS will automatically be appended to the linker commands. These are flags
like `-lm` and function similarly to our INCLUDES variable but are added at a
different step. `m` denotes the math library.

That's about it for our variable declarations. The next step is to define
compile order and dependencies. The very first "target" or rule in your 
makefile gets built when you type `make` in this case the first target is:

```make
main: main.o myadd.o
```

Note that we did not specify the linking rule, because make follows an implied
linking rule:

    $(CC) $(LDFLAGS) <all-dependent-.o-files> $(LDLIBS)

Also note that make assumes that main depends on main.o, so we could omit it:

```make
main: myadd.o 
```

Basically what this rule says is make should produce an executable called 
"main" by linking myadd.o and main.o. This declares main.o and myadd.o as 
dependencies of main, meaning that if any of the dependencies (or their 
dependencies) change between the last time this target was run, it should 
re-run the outdated targets as well as this one.

The next target we declare is main.o:

```make
main.o: main.c myadd.h
```

This says that main.o depends on main.c (assumed) as well as myadd.h. See last
week's recitation notes to understand why main.o depends on myadd.h. We could
omit main.c as follows:

```make
main.o: myadd.h
```

Either way, we do not specify a rule here because make assumes the implicit
rule:

    $(CC) -c $(CFLAGS) <the-.c-file>

Lastly, we specify the target for myadd.o:

```make
myadd.o: myadd.c myadd.h
```

We'll include two phony targets. We tell make that they're "phony" so that it
doesn't attempt to use implicit rules or try to compile them. The first target
we make is "clean" which should remove all intermediate files. 
*Always include a clean* so that `make clean` can be used to remove 
intermediate files like object files, compiled code, etc. This should return 
your directory to just its source code that can generate all the other files. 
*Be careful:* Using `rm -f` will not prompt you to remove files. This is 
customary for `make clean` but it also means if you make a mistake in 
designing your rule it could remove files that youdidn't want to. There is no 
"trash" in UNIX - they'll be gone forever.

Lastly, we define a phony "all" target that just depends on the main and clean
targets. This will always remove all intermediate files and compiled files,
forcing make to recompile everything when main is called.

```make
.PHONY: clean
clean:
        rm -f *.o a.out core main

.PHONY: all
all: clean main
```



## Advanced Makefiles ##

This material is great to review, later, once you've worked with Make a little
more.

Makefiles aren't as hard as they look so long as you understand the three steps
to compilation. [(Source
here)](http://stackoverflow.com/questions/6264249/how-does-the-compilation-linking-process-work)

1. Pre-processing: This is when special instructions like `#include` are
processed.
2. Compilation: This is when your c code is processed into an object file in
binary form. At this point, undefined symbols are okay so long as they're
declared. That's why you `#include` header files - to declare functions and
variables that you don't define in that particular file. `gcc` lets you stop the
entire process at this point using the `-c` flag. Object files generated in this
step can be linked together to form an executable or put in a special archive
called a static library. Most error checking takes place at this stage.
3. Linking: The linker takes object files with binary symbols and produces
either a shared (AKA dynamic) library, or an executable. At this point it
replaces all the references to undefined symbols with the proper memory address.
Most often at this stage the only errors left come from double-defined symbols
or undefined symbols.

Now let's take a look at the Makefile's default rules and variables:

Compilation (includes pre-processing):
```make
n.o:
  $(CC) -c $(CPPFLAGS) $(CFLAGS) n.c
```
Linking:
```make
n:
  $(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)
```

Let's take a look at some of the variable definitions we've used in the past:

```make
CC  = gcc
INCLUDES =
CFLAGS   = -g -Wall $(INCLUDES)
LDFLAGS = -g
LDLIBS =
```

And here are some entries from running `man gcc`:

```
gcc [-c|-S|-E] [-std=standard]
           [-g] [-pg] [-Olevel]
           [-Wwarn...] [-pedantic]
           [-Idir...] [-Ldir...]
           [-Dmacro[=defn]...] [-Umacro]
           [-foption...] [-mmachine-option...]
           [-o outfile] [@file] infile...
-g  
           Produce debugging information in the operating system's native format (stabs, COFF, XCOFF, or DWARF 2).
           GDB can work with this debugging information.

           On most systems that use stabs format, -g enables use of extra debugging information that only GDB can
           use; this extra information makes debugging work better in GDB but will probably make other debuggers
           crash or refuse to read the program.

-Wall
           Turns on all optional warnings which are desirable for normal code.  At present this is -Wcomment,
           -Wtrigraphs, -Wmultichar and a warning about integer promotion causing a change of sign in "#if"
           expressions.  Note that many of the preprocessor's warnings are on by default and have no options to
           control them.

-I dir
           Add the directory dir to the list of directories to be searched for header files.  Directories named by -I
           are searched before the standard system include directories.  If the directory dir is a standard system
           include directory, the option is ignored to ensure that the default search order for system directories
           and the special treatment of system headers are not defeated .  If dir begins with "=", then the "=" will
           be replaced by the sysroot prefix; see --sysroot and -isysroot.

-Ldir
           Add directory dir to the list of directories to be searched for -l.

-llibrary
-l library
           Search the library named library when linking.
```

Given what we know about compilation, where might we want to use these flags and
for what purposes? The order we use these flags and the variables we insert them
in is **very** important.
