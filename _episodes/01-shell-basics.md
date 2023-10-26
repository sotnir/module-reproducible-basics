---
title: "Command line/shell"
teaching: 150
exercises: 30
questions:
- "How does using the command line/shell efficiently increase the reproducibility of neuroimaging studies?"
- "How can we ensure that our scripts *do the right thing*?"
objectives:
- "Understand basic differences among available shells and their use(s) in
 neuroimaging toolkits"
- "Use a shell as a working medium"
- "Explain the role of some of the most important environment variables"
- "Provide hints on efficient use of the collected shell history of commands"
- "Explain how to make shell scripts more robust and less dangerous"
- "Introduce basics of runtime and unit testing"
keypoints:
- "There are a number of incompatible shells; different
  neuroimaging tools may use specific shells and thus provide
  instructions that are not compatible with your current shell."
- "A command line shell is a powerful tool and learning additional
  'tricks' can help make its use much more efficient, less
  error-prone, and thus more reproducible."
- "Shell scripting is the most accessible tool to automate execution of
  arbitrary set of commands; this avoids manual retyping of the
  same commands and in turn avoids typos and erroneous analyses."
- "Environment variables play a big role in defining script behavior."
- "You can write automated tests for your commands to ensure correct execution."
- "Shell scripts are powerful, but -- if misused -- can cause big problems."
---

> ## You can skip this lesson if you can answer these questions:
>
>  - What factors affect the execution of a given typed command in shell?
>  - How can you script the execution of a list of commands given user input
>    arguments?
>  - How can you guarantee that your script was executed correctly and will not
>    fail during execution?
>  - How can you use text editors to edit your current command line?
>  - How can you quickly recover a sequence of commands you ran in shell
>    (e.g., an analysis from a year ago)?
{: .challenge}


> ## For Windows OS users
>  If you do not have a remote or virtual
>  environment with Unix or Linux system, you can
>  [enable a "native" Linux shell on your Windows 10 system](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
{: .callout}

## What is a “shell”?

*Shell* commonly refers to the UNIX shell environment, which in its
core function provides users with a CLI (command line interface) to
manipulate "environment variables" and to execute external
commands. Because desired actions are expressed as typed commands, it
is possible to script (program) sets of those commands to be
(re-)executed repetitively or conditionally. For example, it provides constructs
for loops, functions, and conditions.  So, in contrast to GUIs (graphical
user interfaces), such automation via scripting is a native feature of
a CLI shell.  Unlike GUI-integrated environments with lots of
functionality exposed in menu items and icons, shell is truly a "black
box", with lots of powerful underlying features integral to efficient use.
Since manipulating files is one of the main tasks in a shell, a shell usually
comes with common commands (such as `cp`, `mv`, etc.) built in
or provided by an additional package (e.g., `coreutils` in Debian).

In this lesson, we first familiarize ourselves with basic (and
at times advanced) features of a shell and shell scripting, then
review a few key aspects related to reproducibility.
We will examine best practices for controlling command execution: knowing
which command actually was run, what conditions could have potentially
affected its execution, inspecting the available history of the
commands, and verifying that a script did not complete while hiding a
failed interim execution.

> ## External teaching materials
>
> Before going through the rest of this lesson, you should learn the
> basics of shell usage and scripting. The following lesson provides a
> good overview of all basic concepts.  Even if you are familiar with
> shell and shell scripting, please review the materials of the lesson and
> try to complete all exercises in it, especially if you do not know
> correct answers right away:
>
>   - [Software Carpentry: The Unix Shell (full: 1 h 35 min, review: 20 min)](http://swcarpentry.github.io/shell-novice/)
{: .callout}

> ## Additional materials
>
> If you are interested in knowing more about the history and features of
> various shells, please review the materials under following external links:
>
>  - [Wikipedia:Unix shell](https://en.wikipedia.org/wiki/Unix_shell)
>  - [Data Science at the Command Line](http://datascienceatthecommandline.com) --
>   contains a list of command line tools useful for “data science”
{: .callout}


## Commonly used shells and their relevance to existing neuroimaging projects

- **sh** - a [POSIX](https://en.wikipedia.org/wiki/POSIX)-compliant shell; this is a generic name and doesn’t refer to a specific project.
   - most portable shell (since it's the standard)
   - many FSL scripts use this shell
- **ksh** - KornSHell; based on older bash, but also became a root for tcsh, zsh, and others.
- **dash** - Debian Almquist SHell; an implementation of a POSIX-compliant shell (**sh**).
   - you will not see it used directly in a shebang
- **bash** - Bourne Again SHell; default shell for Mac OS and many Linux distributions.
   - (optionally) POSIX-compliant but with additional features from **ksh** and **csh**
   - next most portable and most popular for shell scripting (after
     **sh**) and generally available (in contrast to **zsh**)
- **csh/tcsh** - C SHell/Tenex C SHell; shell that aims to provide an interface similar to the C programming language.
   - heavily used by FreeSurfer and AFNI (primarily @ scripts)
- **zsh** - powerful, but not POSIX/bash-compatible; inspired by many features from **ksh** and **tcsh**.
   - rarely used for generic scripting

> ## References
> Additional materials
> - [Wikipedia:Comparison of command shells](https://en.wikipedia.org/wiki/Comparison_of_command_shells)
{: .callout}

## Challenges

> ## How can you determine what shell you're currently in?
> ~~~
> % echo $SHELL
> ~~~
> {: .bash}
{: .solution}

> ## How do you change the **current shell** of your current session?
> Executing the name of the shell starts it. For example:
> ~~~
> % tcsh
> ~~~
> {: .bash}
> would enter a new `tcsh` session.  You can exit it and return to
> your previous shell by typing `exit` or just pressing `Ctrl-d`.
{: .solution}

> ## How do you change **the login shell** (the one you enter when you login) for your account?
> ~~~
> % chsh
> ~~~
> {: .bash}
{: .solution}


> ## What is a shebang?
> It is the first line in the script, which starts with `#!` and is
> followed by the command interpreting the script; e.g.,
> if a file `blah` begins with the following:
> ~~~
> #!/bin/bash
> echo "Running this script using bash"
> ~~~
> {: .bash}
> then running `./blah` is analogous to calling `/bin/bash ./blah` .
> The string "#!" is said out loud
> as "hash-bang" and therefore is shortened to "shebang".
{: .solution}


> ## Exercise: supplying options to a shebang
>
> To help answer the question, determine which of the following shebangs would be correct and what their effect would be.
>
> 1. `#!/bin/bash`
> 2. `#!/bin/bash -e`
> 3. `#!/bin/bash -ex`
> 4. `#!/bin/bash -e -x`
>
> > ## Solution
> > A shebang can support one option, so 1-3 are all correct. The `-ex` flag
> > instructs the script to exit immediately if a command returns a non-zero
> > exit code (`-e`), and to print commands and their arguments as they're
> > executed (`-x`).
> {: .solution}
{: .challenge}


## Environment variables

Environment variables are not a feature of a shell per se. Every
process on any operating system inherits some "environment variables"
from its parent process. A shell just streamlines manipulation of those
environments and also uses some of them to guide its own
operation. Let's overview the most commonly used
environment variables. These variables are important because they
impact what external commands and libraries you are using.

### PATH - determines full path to the command to be executed

Whenever a command is run without providing the full path on the
filesystem, the shell consults the `PATH` environment variable to determine
where to look for the command. You may have multiple
implementations or versions of the same command available at different
locations, which may be specified within the PATH variable (separated
with a colon). Although this is a very simple concept, it is a workhorse
for "overlay distributions" (such as [conda](https://conda.io)). It is
also a workhorse for "overlay environments" such as [virtualenv](https://virtualenv.pypa.io)
in Python, or [modules](http://modules.sourceforge.net/) on servers.
It can also be a source of much confusion in cases where an unintended
command is run. This is why any tool which aims to capture the
state of the computational environment for later re-execution
needs to store the value of the PATH variable to guarantee that
given the same set of files, the same commands are executed. For example,
we may have two different versions of FSL installed in different locations;
without specifying the path to a particular installation of FSL, we may
unintentionally run a different version than intended and end up with different results.

> ## How can you determine the full path of a command?
>
> The `which` command can show which program will actually be used you
> run a command. For example:
> ~~~
> $ which fsl
> /usr/local/fsl
> ~~~
> {: .bash}
> **Note:** Do not confuse this with the `locate` command, which (if available) would
> find a file containing the specified word somewhere in the file name/path.
{: .solution}


> ## Beware of built-in ("builtin") commands
>
> Some commands might be implemented by a shell itself, and that
> implementation may differ from the one provided by another
> set of tools.
>
> Note that `which` is not a builtin command in bash (but is in
> zsh), meaning that in bash you would not be able to "resolve"
> builtin commands such as `pwd`.
>
> ~~~
> % pwd -h      # bash builtin
> bash: pwd: -h: invalid option
> pwd: usage: pwd [-LP]
>
> % which pwd
> /bin/pwd
>
> % /bin/pwd -h   # provided by coreutils
> /bin/pwd: invalid option -- 'h'
> Try '/bin/pwd --help' for more information.
> ~~~
> {: .bash}
{: .callout}


> ## Exercise: add a new path where the shell will look for commands such that...
>
> 1. those commands take precedence over identically named commands
> available elsewhere on the `PATH`?
>
> 2. those commands are run only if not found elsewhere
> on the `PATH`? (rarely needed/used case)
>
> > ## Solution
> > For a new path /a/b/c:
> > 1. Use `PATH=/a/b/c:$PATH`
> > 2. Use `PATH=$PATH:/a/b/c`
> {: .solution}
{: .challenge}

> ## Exercise: determine the environment variables used by a process
>
> Since each process inherits and possibly changes environment
> variables so that its child processes inherit them in turn,
> it is often important to be able to inspect them.  
> Given a `PID` of a currently running process (e.g., the `$$` variable in POSIX shell contains a
> `PID` of your active shell), how can you determine its environment variables?
>
> > ## Solution
> > 1. By looking into `/proc/PID/environ` file on Unix/Linux
> > systems.  Try finding the entries in that file separated with the byte
> > `0`. Use the `tr` command to separate them with a line.
> > 2. `ps e PID` will list all environment variables along with their
> > values
> > 3. The `e` shortcut in `htop` will show the environment variables of the
> > selected process
> {: .solution}
{: .challenge}

> ## Why is ${variable} is preferable over $variable?
>
> Using `${variable}` avoids ambiguity by safely concatenating a variable with another string.  
> For instance, if you had a variable `filename` that contains the value
> `preciousfile`, `$filename_modified` would refer to the value of the
> possibly undefined `filename_modified` variable; on the other hand, `${filename}_modified`
> will produce the desired value of `preciousfile_modified`.
> ~~~
> % filename="previousfile"
> % filename_modified="somerandomstring"
> % echo $filename
> previousfile
> % echo $filename_modified
> somerandomstring
> % echo ${filename}_modified
> previousfile_modified
> ~~~
> {: .bash}
{: .solution}


### LD_LIBRARY_PATH - determine which dynamic library is used

To improve maintainability and to make distributions smaller, most
programs use dynamic linking to reuse common functions provided by
shared libraries. The particular list of dynamic libraries that an
executable needs is often stored without full paths as well. Thus,
`ld.so` (e.g., `/lib/ld-linux.so.2` on recent Debian systems), which
takes care of executing those binaries, needs to determine which
particular libraries to load. 

The `LD_LIBRARY_PATH` environment variable resolves paths for loading dynamic
libraries in the same way the `PATH` variable resolves
paths for the execution of commands. Unlike `PATH`, however, `ld.so` does assume a list of
default paths (e.g., `/lib`, then `/usr/lib` on Linux systems, as
defined in `/etc/ld.so.conf` file(s)). Consequently, you may not even have
explicitly set it in your environment!

> ## How can you discover which library is used?
>
> `ldd EXEC` and `ldd LIBRARY` list libraries of a given binary. If a
> library is linked, a full path is provided if found
> using `ld`'s default paths or the `LD_LIBRARY_PATH` variable. For example:
> ~~~
> % ldd /usr/lib/afni/bin/afni | head
>	linux-vdso.so.1 (0x00007fffd41ca000)
>	libXm.so.4 => /usr/lib/x86_64-linux-gnu/libXm.so.4 (0x00007fd9b2075000)
>	libmri.so => /usr/lib/afni/lib/libmri.so (0x00007fd9b1410000)
>   ...
> ~~~
> {: .bash}
{: .callout}


> ## Swiss army knife to inspect execution on Linux systems
>
> [strace](https://en.wikipedia.org/wiki/Strace) traces "system calls"
> -- the calls your program makes to the core of the operating system (i.e., kernel).
> This way you can discover what files any given program tries to
> access or open for writing, which other commands it tries to run,
> etc. Try running `strace -e open` and provide some command to be
> executed.
>
{: .callout}


> ## Possible conflicts
>
> It is possible for `PATH` to point to one environment while
> `LD_LIBRARY_PATH` points to libraries from another environment,
> which can cause either incorrect or hard-to-diagnose
> behavior later on. In general, you should avoid manually manipulating
> these two variables.
{: .callout}


### PYTHONPATH - determine which Python module will be used

The idea of controlling path resolution via environment variables
also applies to language-specific domains. For example, Python consults
the `PYTHONPATH` variable to determine search paths for Python
modules.

> ## Possible side-effect
>
> Having a mix of system-wide and user-specific installed
> applications/modules with custom installations in virtualenv environments
> can cause unexpected use of modules.
>
> You can use `python -c 'import sys; print(sys.path)'` to output a
> list of paths your current default Python process will consult
> to find Python libraries.
>
{: .callout}


### Additional considerations

> ## Exercise: "exported" vs. "local" variables
>
> Variables can be "exported" so they will be inherited by any new
> child process (e.g., when you run a new command in a shell). Otherwise,
> the variable will be "local", and will not be inherited by child processes.
>
> 1. How can you determine if a variable was exported or not?
> 2. How do you produce a list of all local environments (present in your shell but
>  not exported)?
>
> > ## Solution
> >
> > 1. Only exported variables will be output by the `export` command. Alternatively,
> >    you can use `declare -p` to list all variables prepended with
> >    a specific attribute:
> >    ~~~
> >    % LOCAL_VARIABLE="just for now"
> >    % export EXPORTED_VARIABLE="long live the king"
> >    % declare -p | grep \_VARIABLE
> >    declare -x EXPORTED_VARIABLE="long live the king"
> >    declare -- LOCAL_VARIABLE="just for now"
> >    ~~~
> >    {: .bash}
> > 2. Extrapolate from 1.: 
> >    ~~~
> >    declare -p | grep -e '^declare --'
> >    ~~~
> >    {: .bash}
> {: .solution}
{: .challenge}


## Efficient use of the interactive shell

A shell can simplify common operations and effectively boost your efficiency 
once becoming familiar with its features and configuring properly.

### What are "aliases"?

Aliases are shortcuts for commonly used commands and can add
options to calls for most common commands.  

> ## Additional materials
> Please review useful aliases presented in
> [30 Handy Bash Shell Aliases For Linux / Unix / Mac OS X](https://www.cyberciti.biz/tips/bash-aliases-mac-centos-linux-unix.html).
{: .callout}

> ## Should aliases defined in your `~/.bashrc` be used in your scripts?
>
> No. Since `~/.bashrc` is read only for interactive sessions,
> aliases placed there will not be available in your scripts'
> environment. Even if they were available after some
> manipulation, it would be highly inadvisable to use them, since
> that would render your scripts not portable across machines/users.
{: .solution}

### Editing command line

`bash` and other shells use the `readline` library for basic navigation
and manipulation of the command line entry.  That library provides two
major modes of operation which are inspired by
[two philosophically different editors](https://en.wikipedia.org/wiki/Editor_war)
-- `emacs` and `vim` .

Use **set -o emacs** to enter emacs mode (default) and **set -o vi** to
enter vi mode. Subsequent discussion and examples refer to the default,
**emacs** mode. Learning navigation shortcuts can increase your
efficiency with the shell tenfold, so let's review most common ones
to edit the command line text:

`Ctrl-a` | Go to the beginning of the line you are currently typing on
`Ctrl-e` | Go to the end of the line you are currently typing on
`Ctrl-l` | Clear the screen (similar to the clear command)
`Ctrl-u` | Remove text on the line before the cursor position
`Ctrl-h` | Remove preceding symbol (same as backspace)
`Ctrl-w` | Delete the word before the cursor
`Ctrl-k` | Remove text on the line after the cursor position
`Ctrl-t` | Swap the last two characters before the cursor
`Alt-t`  | Swap the last two words before the cursor
`Alt-f`  | Move cursor forward one word on the current line
`Alt-b`  | Move cursor backward one word on the current line
`Tab`    | Auto-complete files, folders, and command names

> ## Hints
> 
> - If the `Alt-` combination does not work, you can temporarily work around that
>   by hitting the `Esc` key once, instead of holding `Alt` before pressing the
>   following command character.
> - Although many navigational commands can be achieved also by using
>   "arrow keys" on your keyboard, sometimes using their `Ctrl-`
>   counterparts is more efficient since it doesn't require you to move
>   away your hands from the main alphanumeric portion of the keyboard.
> - Many people find the need to use `Ctrl` key more often than
>   `CapsLock` (which was originally used to assist with FORTRAN and other languages
>   where all keywords had to be CAPITALIZED). You can
>   [change your environment settings](https://www.emacswiki.org/emacs/MovingTheCtrlKey)
>   to either swap them or to make `CapsLock` into another `Ctrl` key.
{: .callout}

If you need a more powerful way to edit your current command line, use

`Ctrl-x Ctrl-e` (or `Alt-e` in **zsh**) | Edit command line text in the editor (as defined by `VISUAL` environment variable)


Some shortcuts can not only edit command line text, but also control the execution of processes:

`Ctrl-c` | Kill currently running process
`Ctrl-d` | Exit current shell
`Ctrl-z` | Suspend currently running process; `fg` restores it, and `bg` places it into background execution

> ## Interrogating shell options with `set -o`
>
> Shells provide a set of configurable options which can be enabled or
> disabled using the `set` command.  Use `set -o` to print the current settings
> you have in your shell, and then navigate `man bash` to find their
> extended description.
>
> When using `man`, you can search the manual page by using the shortcut `/` and
> typing `o option-name`. You can type 'n' for the "next" and 'p' for
> "previous" finding to identify the corresponding section.
> For example, use `set -o noclobber` which can be used to forbid
> overwriting of previously existing files. `>|` could be used to
> explicitly instruct the overwriting of an already existing file. "A shell
> redirect ate my results file" should no longer be given as a valid excuse.
>
{: .challenge}

### Shell history

By default, a shell stores in memory a history of the commands you
have run. You can access this log using the `history` command. When you exit
the shell, those history lines are appended to a file (by default in
`~/.bash_history` for bash shell). This not
only allows you to quickly recall commands you have run recently, but
can effectively provide a "lab notebook" of the actions you have
performed. The shell history can be very useful for two reasons. First, it can provide
a skeleton for your script and help you realize that automating
your shell commands is worth the effort. Second, it helps you determine exactly
which command you ran to perform any given operation.

> ## Eternal history
>
> Unfortunately, by default shell history is truncated to the 1000 last
> commands, so you cannot use it as your "eternal lab notebook" without
> some tuning.  Since this is a common problem, solutions
> exist, so please review available approaches:
> - [shell-chronicle](https://github.com/con/shell-chronicle)
> - [tune up of PROMPT_COMMAND](https://debian-administration.org/article/543/Bash_eternal_history)
>   to record each command as soon as it finishes running
> - adjustment of `HISTSIZE` and `HISTCONTROL` settings,
>   e.g. [1](http://www.pointsoftware.ch/howto-bash-audit-command-logger/)
>   or [2](http://superuser.com/questions/479726/how-to-get-infinite-command-history-in-bash)
{: .callout}

Some of the main keyboard shortcuts to navigate shell history are:

`Ctrl-p` | Previous line in the history
`Ctrl-n` | Next line in the history
`Ctrl-r` | Bring up next match backwards in shell history

You can hit `Ctrl-r` ("reverse-i-search") and start typing some portion of the command you
remember running. Subsequent use of `Ctrl-r` will bring up the next match, and so
on. You will leave "search" mode as soon as you use some other
command line navigation command (e.g., `Ctrl-e`).

`Alt-.` | Insert the final argument of the previous command.

Subsequent use of `Alt-.` will bring up the last argument of the previous command,
and so on.

> ## Exercise: History navigation
>
> Inspect your shell command history you have run so far:
> 1. Use `history` and `uniq` commands to figure out what which command you run the most
> 2. Experiment with `Ctrl-r` to find the commands next to the most
>    popular command
{: .challenge}


## Hints for correct/robust scripting in shell

### Fail early

By default, your shell script might execute even if some
command within it fails.  This might lead to some very bad side effects:

- operating on incorrect results (e.g., if the command to reproduce
  results failed, but script continued)
- polluting the terminal screen (or log file) with output hiding a
  point of failure

This is why it's generally advisable to use `set -e` in scripts
to instruct the shell to exit with non-0 exit code as soon as a command fails.

> ## Note on special commands
> POSIX defines [some commands as "special"](https://www.gnu.org/software/bash/manual/html_node/Special-Builtins.html#Special-Builtins),
> such that failure to execute would cause the entire script to exit, even
> without `set -e`, if they returned a non-0 exit code: `break`, `:`, `.`,
> `continue`, `eval`, `exec`, `exit`, `export`, `readonly`, `return`,
> `set`, `shift`, `trap`, and `unset`.
{: .callout}

If you expect some command to fail and that's okay, handle its
failing execution explicitly; e.g., via:

~~~
% command_ok_to_fail || echo "As expected command_ok_to_fail failed"
~~~
{: .bash}

or just
~~~
% command_ok_to_fail || :
~~~
{: .bash}


### Use only defined variables

By default, POSIX shell and bash treat undefined variables as variables
containing an empty string:

~~~
> echo ">$undefined<"
><
~~~
{: .bash}

which can lead to many undesired and non-reproducible
or undesirable side-effects:

- "using" mistyped variable names
- "using" variables that were not defined yet due to the logic
  of the script; for example, imagine the effects of `sudo rm -rf ${PREFIX}/` if the
  `PREFIX` variable was not defined for some reason (**Do not copy this into your terminal!**)

The `set -u` option instructs the shell to fail if an undefined variable is
used.

If you intend to use some variable that might still be undefined
you can either use `${var:-DEFAULT}` to provide an explicit `DEFAULT`
value or define it on the condition that it doesn't already exist; e.g.:

~~~
% : ${notyetdefined:=1}
% echo ${notyetdefined}
1
~~~
{: .bash}


> ## set -eu
> Include `set -eu` toward the beginning of  your shell script.
> This command sets both "fail early" modes for extra protection to make your
> scripts more deterministic and thus reproducible.
{: .callout}


## (Unit) Testing

### Run-time testing

To some degree you can consider the `set -u` feature to be a "run-time
test" -- i.e., "test if variable is defined, and if not, fail". In fact, **bash**
and other shells provide a command called `test`, which
can perform various basic checks and return with a non-0 exit code if the
condition is not satisfied. For undefined variables, use `test -v`:

~~~
% test -v undefined
% echo $?
1
~~~
{: .bash}

See the “CONDITIONAL EXPRESSIONS" section of the `man bash` page for more
conditions, such as:

`-a file`   | True if file exists
`-w file`   | True if file exists and is writable
`-z string` | True if the length of string is non-zero

Instead of calling the `test` command, you can use
`[ TEST-EXPRESSION ]` syntax, so `test -v undefined` is identical to
`[ -v undefined ]`.

With `set -e` the whole operation of your script can be stated to be
somewhat tested -- the script will fail as soon as any command fails.
Using such tests/assertions in your code can help guarantee that
your script performs as expected.

> ## Exercise: TODO, under construction.
{: .challenge}

### Unit-testing

[Unit-testing](https://en.wikipedia.org/wiki/Unit_testing) is a
powerful paradigm to verify that pieces of your code (units) operate
correctly in various scenarios, and that these assumptions are represented in
the code. In fact,  everyone does at least some
"testing" by simply running their code/script on an input and checking
that the output matches their expectations. Unit-testing just takes this
workflow one step further: code such tests in a separate file so you can run
them all at once later on (e.g., whenever you change your script) to verify
that your script still performs correctly. In the simplest case, you can
just copy your test commands into a separate script that would fail if
any command within it fails, and therefore effectively testing your target
script(s).

For example, the following script could be used to test basic correct operations
of AFNI's `1dsum` command:

~~~
tfile=$(mktemp)              # create a temporary random file name
printf "1\n1.5\n" >| $tfile  # populate file with known data
result=`1dsum $tfile`        # compute result
[ "$result" = "2.5" ]        # compare result with target value
rm $tfile                    # cleanup
~~~
{: .bash}

Although it looks trivially simple, this is a powerful basic test
to guarantee that `1dsum` is available, that it is installed
correctly, and that it operates correctly on
typical files stored on the file system.

To have better management over a collection of such tests, testing
frameworks were developed for shell scripts. Notable ones are:

- [shunit2: xUnit based unit testing for Unix shell scripts](https://code.google.com/archive/p/shunit2/)
- [Bats: Bash Automated Testing System](https://github.com/bats-core/bats-core)

In general, they provide helpers with the means to execute tests.
Helpers then report which ones passed and failed as they run a collection
of tests.

> ## Exercise: use existing testing framework to evaluate script
>
> Choose shunit2 or bats (or both) and
>
> 1. Re-write the above test for `1dsum` using one of the frameworks.  If
> you do not have AFNI available, you can test generic `bc` or `dc`
> command line calculators that may be available on your system.
> 2. Add additional tests to "document" behavior of `1dsum` whenever
>   - input file is empty
>   - multiple files are provided
>   - some values are negative
{: .challenge}


> ## Testing frameworks
>
> Although we've focused on testing shell scripts, testing frameworks exist
> nearly for every programming and scripting language/environment (see [Wikipedia: List of unit testing frameworks](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks).
We recommend extending this testing framework to code you write at all stages
of your analysis pipeline.
{: .callout}
