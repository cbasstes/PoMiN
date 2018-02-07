# PoMiN
PoMiN: a Post-Minkowskian N-Body Solver

## Description

N-body code which solves Hamilton's equations for a system of particles using
the N-body Hamiltonian in the following paper:

https://arxiv.org/abs/1003.0561[Ledvinka, T., Schafer, G., Bicak, J., Phys. Rev. Lett. 100, 251101 (2008)]

The Hamiltonian presented in the paper above models a system of weakly
gravitating point particles. It is fully special relativistic, but includes
only first-order effects in (Newton's constant) G from General Relativity.

The code uses a fourth-order Runge-Kutta integrator, and has a computational
complexity that scales as N^2. The code implements a basic, global adaptive
timestepping scheme, based on the distances and velocities of the two closest
particles. Support for parallel computing has not yet been implemented.

For more, see link:doc/pomin.pdf[pomin.pdf].

pomin takes input from the standard input (stdin) and outputs to the
standard output (stdout); both of which are in the comma-separated values (CSV)
format (see the *INPUT* and *OUTPUT* sections).

For more information about using stdin and stdout, see the *NOTES* section below.
Also see the *EXAMPLES* section for some useful uses.

## Options

-d
    Print debug (e.g., auxiliary values and metadata) information to stdout.

-v
    Print verbose (e.g., wall-time) information to stderr.

-p <parts>
    Use parts of the Hamiltonian instead of 123; that is, which of the three parts of the Hamiltonian to use;
        1    to use H1 only
        12   to use H1+H2
        13   to use H1+H3 or
        123  to use H1+H2+H3 (default)


## Input

Input must be read from stdin as one line in the following format:

    description,start-time,end-time,timestep,iterations,courant-number,gravitational-constant,speed-of-
    light,mass_1,qx_1,qy_1,qz_1,px_1,py_1,pz_1,mass_2,qx_2,...


INPUT,                  TYPE,    DESCRIPTION
description,            string,  ignored by program--except it cannot be empty
start-time,             float,   time to start at (physics time)
end-time,               float,   time to end at (physics time)
timestep,               float,   (maximum) length of each computational iteration
iterations,             integer, upper bound of iterations (0 = unbounded)
courant-number,         float,   adaptive-timestep parameter (0 = off)
gravitational-constant, float,   you know: G
speed-of-light,         float,   that'd be: c
mass_1,                 float,   mass of particle 1
qx_1,                   float,   x-position of particle 1
...,                    ...,     ...
py_3,                   float,   y-momentum of particle 3
...,                    ...,     ...

NOTE: the 'courant-number' is the ratio of the change in distance between the
two closest particles (in a single timestep) to the distance between those
particles.

WARNING: the 'description' field cannot be empty due to the strtok(1) function
parsing; however, using a unique value here is helpful when selecting input to
use with, e.g., grep(1).

To generate a header line for n particles, run the “header” make(1) target:

    make header N=n

The pomin code will automatically re-run itself if it detects another line of input
(i.e., recursion).


## Output

Output is written to stdout (except for errors, which are written to stderr) in
this form:

    iteration_#,time,qx_1,qy_1,qz_1,px_1,py_1,pz_1,qx_2,qy_2,...

### Debug

If the debug flag is given, some metadata and other variables are also printed:

The above output will be prefixed with the time and date (formatted in
accordance with ISO 8601: YYYY-MM-DDTHH:MM:SS) of when the program was executed
along with the git(1) hash at that time (with an idicator of whether the
working directory was clean or not) and which mathematical precision was used
(double or quad). Furthermore, the standard output will be interlaced with the
following information (identified by headers):

    H1,H2,H3,dH1,dH2,dH3,qdotx_1,qdoty_1,qdotz_1,qdotx_2,...

where 'H' and 'dH' are the values of the Hamiltonian (H1+H2+H3) and its
derivative at that given time, followed by the components of the derivative of
each particle's position along the three Cartesian axes.

### Notes

+stdin+ (standard in) and +stdout+ (standard out) are special names for two
ordinary files. To get input to (and output from) pomin all you have
to do is write to (and read) these files; imagine your data travelling like
this:

    stdin --> pomin --> stdout

Now, by default, +stdin+ is associated with your keyboard and +stdout+ is
associated with your monitor. But typing the input by hand only to have it
display on screen isn't very useful; so we redirect these “file descriptors“ to
other files. For example, to redirect the output (+stdout+) to a file we write

    pomin > file

That right angle-bracket (>) will take the output of a command and write it to
a file; this is known as “redirection.”

Now we only need to get our input (+stdin+) from someplace better. To do this,
we use a “pipe” (a vertical bar: |) to take the output of one command and give
it as input to another command like so:

    cat file | pomin

The cat command takes the contents of a file and writes it to +stdout+ -- then
the pipe takes this output and writes it to the stdin of the next program;
essentially creating a cohesive stream of data:

              /\-- pipe --\/
    cat --> stdout       stdin --> pomin

All of this sounds like more work than necessary but, in fact, it makes the
program much more robust and useful.


### Obtainting

The Git repository can be obtained through the SSH protocol with

    git clone git@github.com:jchristopherfeng/PoMiN.git


### Compiling

To compile run

    make [double | quad]

where double (the default) or quad is the desired precision. Then, to install
the program on an operating system that abides by the Filesystem Heirarchy
Standard (FHS; e.g., a GNU/Linux distribution) run make install and the
executible will be put into /usr/local/bin/ and this man page will be put into
/usr/local/man/man1/ so that you can run the program from any directory and
view its manual at any time with man pomin


### Scripts

There are several files in the ./scripts directory that can be used to automate
various tasks. Each file can be executed with the -h flag to show usage and
help information; e.g., python ./script/animate.py -h


## Examples

To use a file named +input.csv+ that has exactly one line of input (as described
in the *INPUT* section) and write the output to a file called +output.csv+, try

    cat input.csv | pomin > output.csv

To use as input a line from a file (of any size) called +input.csv+ with “test
one” in the description field and only save every other output line (and the
header, line #1):

    grep “test one” cat input.csv | pomin | awk 'NR == 1 || NR % 2 == 0' > output.csv

To run all of the lines (except the header) from a file and split their outputs
into seperate files:

    tail -n +2 input.csv | pomin | csplit --prefix="output/test_" -sz - '/^iteration/' '{*}'

To print only the first and last lines of output:

    pomin | sed -e 1b -e '$!d'


## Authors

Developed in the Center for Relativity at the University of Texas at Austin, Department of Physics.

Mark Baumann <markb@physics.utexas.edu> +
Joel S. Doss <jdoss@uoregon.edu> +
Justin C. Feng <jcfeng@physics.utexas.edu> +
Bryton T.D. Hall <email@bryton.io> + 
Lucas Spencer


## License

This program is licensed under the MIT License. See LICENSE file.


## See Also
stdin(3), stdout(3), stderr(3), grep(1), cat(1), echo(1), awk(1), sed(1), head(1), tail(1), csplit(1)
