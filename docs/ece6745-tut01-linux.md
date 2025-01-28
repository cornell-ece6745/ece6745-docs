
Tutorial 1: Linux Development Environment
==========================================================================

The laboratory assignments for this course are designed assuming you will
be using a Linux (or UNIX-like) operating system for development. Basic
Linux knowledge is essential to successfully complete this work and a
more in-depth understanding enhances productivity. This tutorial covers
the computing resources to be used in the course and offers a brisk
introduction to the Linux operating system for first time users including
some details specific to this course.

Before you begin, make sure that you have **logged into the `ecelinux`
servers** as described in the remote access tutorial. You will need to
open a terminal and be ready to work at the Linux command line using VS
Code. To follow along with the tutorial, type the commands without the
`%` character. In addition to working through the commands in the
tutorial, you should also try the more open-ended activities.

Before you begin, make sure that you have **sourced the
`setup-ece2300.sh` script** as described in the remote access tutorial.
Sourcing the setup script sets up the environment required for this
tutorial.

1. The Linux Command Line
--------------------------------------------------------------------------

In this section, we introduce the basics of working at the Linux command
line. Our goal is to get you comfortable with commands required to
complete the laboratory assignments. The shell is the original Linux user
interface which is a text-based command-line interpreter. The default
shell on the `ecelinux` machines is Bash. While there are other shells
such as `sh`, `csh`, and `tcsh`, for this course we will always assume
you are using Bash. As mentioned above, we use the `%` character to
indicate commands that should be entered at the Linux command line, but
you should not include the actual `%` character when typing in the
commands on your own. To make it easier to cut-and-paste commands from
this tutorial document onto the command line, you can tell Bash to ignore
the `%` character using the following command:

```bash
% alias %=""
```

Now you can cut-and-paste a sequence of commands from this tutorial
document and Bash will not get confused by the `%` character which begins
each line.

### 1.1. Hello World

We begin with the ubiquitous "Hello, World" example. To display the
message "Hello, World" we will use the `echo` command. The `echo` command
simply "echoes'' its input to the console.

```bash
% echo "Hello, World"
```

The string we provide to the `echo` command is called a \IT{command
  line argument}. We use command line arguments to tell commands what
they should operate on. Although simple, the `echo` command can very
useful for creating simple text files, displaying environment variables,
and general debugging.

!!! question "Activity 1: Experiment with `echo`"

    Experiment with using the `echo` command to display different
    messages.

### 1.2. Manual Pages

You can learn more about any Linux command by using the `man` command.
Try using this to learn more about the `echo` command.

```bash
% man echo
```

You can use the up/down keys to scroll the manual one line at a time, the
space bar to scroll down one page at a time, and the `q` key to quit
viewing the manual. You can even learn about the `man` command itself by
using `man man`. As you follow the tutorial, feel free to use the `man`
command to learn more about the commands we cover.

!!! question "Activity 2: Experiment with `man`"

    Use the `man` command to learn more about the `cat` command.

### 1.3. Create, View, and List Files

We can use the `echo` command and a feature called _ecommand output
redirection_ to create simple text files. We will discuss command output
redirection in more detail later in the tutorial. Command output
redirection uses the `>` operator to take the output from one command and
"redirect" it to a file. The following commands will create a new file
named `ece2300-tut01.txt` that simply contains the text "Digital Logic and
Computer Organization"

```bash
% echo "Digital Logic and Computer Organization" > ece2300-tut01.txt
```

We can use the `cat` command to quickly display the contents of a
file.

```bash
% cat ece2300-tut01.txt
```

For larger files, `cat` will output the entire file to the console so it
may be hard to read the file as it streams past. We can use the `less`
command to show one screen-full of text at a time. You can use the
up/down keys to scroll the file one line at a time, the space bar to
scroll down one page at a time, and the `q` key to quit viewing the file.

```bash
% less ece2300-tut01.txt
```

You can use the `ls` command to list the filenames of the files you
have created.

```bash
% ls
```

We can provide _command line options_ to the `ls` command to modify the
command's behavior. For example, we can use the `-1` (i.e., a dash
followed by the number one) command line option to list one file per
line, and we can we can use the `-l` (i.e., a dash followed by the letter
l) command line option to provide a longer listing with more information
about each file.

```bash
% ls -1
% ls -l
```

You should see the newly created `ece2300-tut01.txt` file along with some
additional directories or folders. We will discuss directories in the
next section. Use the following commands to create a few more files using
the `echo` command and command output redirection, and then list the
files again.

```bash
% echo "Application" > ece2300-tut01-layer1.txt
% echo "Algorithm"   > ece2300-tut01-layer2.txt
% ls -1
```

!!! question "Activity 3: Create New File"

    Create a new file named `ece2300-tut01-layer3.txt` which contains the
    third layer in the computing systems stack (i.e., programming
    language). Use `cat` and `less` to verify the file contents.

### 1.4. Create, Change, and List Directories

Obviously, having all files in a single location would be hard to manage
effectively. We can use _directories_ (also called folders) to logically
organize our files, just like one can use physical folders to organize
physical pieces of paper. The mechanism for organizing files and
directories is called the _file system_. When you first login to an
`ecelinux` machine, you will be in your _home directory_. This is your
own private space on the server that you can use to work on the
laboratory assignments and store your files. You can use the `pwd`
command to print the directory in which you are currently working, which
is known as the _current working directory_.

```bash
% pwd
/home/netid
```

You should see output similar to what is shown above, but instead of
`netid` it should show your actual NetID. The `pwd` command shows a
_directory path_. A directory path is a list of nested directory names;
it describes a "path" to get to a specific file or directory. So the
above path indicates that there is a toplevel directory named `home` that
contains a directory named `netid`. This is the directory path to your
home directory. As an aside, notice that Linux uses a forward slash (`/`)
to separate directories, while Windows uses a back slash (`\`) for the
same purpose.

We can use the `mkdir` command to make new directories. The following
command will make a new directory named `ece2300` within your home
directory.

```bash
% mkdir ece2300
```

We can use the `cd` command to change our current working directory. The
following command will change the current working directory to be the
newly created `ece2300` directory, before displaying the current working
directory with the `pwd` command.

```bash
% cd ece2300
% pwd
/home/netid/ece2300
```

Use the `mkdir`, `cd`, and `pwd` commands to make another directory.

```bash
% mkdir tut01
% cd tut01
% pwd
/home/netid/ece2300/tut01
```

We sometimes say that `tut01` is a subdirectory or a child directory of
the `ece2300` directory. We might also say that the `ece2300` directory
is the parent directory of the `tut01` directory.

There are some important shortcuts that we can use with the `cd` command
to simplify navigating the file system. The special directory named `.`
(i.e., one dot) always refers to the current working directory. The
special directory named `..` (i.e., two dots) always refers to the parent
of the current working directory. The special directory named `~` (i.e.,
a tilde character) always refers to your home directory. The special
directory named `/` (e.g., single forward slash) always refers to the
highest-level root directory. The following commands illustrate how to
navigate up and down the directory hierarchy we have just created.

```bash
% pwd
/home/netid/ece2300/tut01
% cd .
% pwd
/home/netid/ece2300/tut01
% cd ..
% pwd
/home/netid/ece2300
% cd ..
% pwd
/home/netid
% cd ece2300/tut01
% pwd
/home/netid/ece2300/tut01
% cd
% pwd
/home/netid
% cd /
% pwd
/
% cd ~/ece2300
% pwd
/home/netid/ece2300
```

Notice how we can use the `cd` command to change the working directory to
another arbitrary directory by simply using a directory path (e.g.,
`ece2300/tut01`). These are called _relative paths_ because the path is
relative to your current working directory. You can also use an _absolute
path_ which always starts with the _root directory_ to concretely specify
a directory irrespective of the current working directory. A relative
path is analogous to directions to reach a destination from your current
location (e.g., How do I get to the coffee shop from my current
location?), while an absolute path is analogous to directions to reach a
destination from a centralized location (e.g., How do I get to the coffee
shop from the center of town?).

```bash
% pwd
/home/netid/ece2300
% cd /home/netid/ece2300/tut01
% pwd
/home/netid/ece2300/tut01
% cd
% pwd
/home/netid
```

This example illustrates one more useful shortcut. The `cd` command with
no command line arguments always changes the current working directory to
your home directory. We can use the `ls` command to list files as well as
directories. Use the following commands to create a new file and
directory in the `ece2300/tut01` subdirectory, and then list the file and
directory.

```bash
% cd ~/ece2300/tut01
% echo "Computer Systems Programming" > ece2300-tut01.txt
% mkdir dirA
% ls -1
```

You should see both the `dirA` subdirectory and the newly created
`ece2300-tut01.txt` file listed. Feel free to use the `cat` command to
verify the file contents of the newly created file. We can use the `tree`
command to recursively list the contents of a directory. The following
commands create a few more directories before displaying the directory
hierarchy.

```bash
% cd ~/ece2300/tut01
% mkdir -p dirB/dirB_1
% mkdir -p dirB/dirB_2
% mkdir -p dirC/dirC_1
% cd ~/ece2300/tut01
% tree
.
+-- dirA
+-- dirB
|   |-- dirB_1
|   `-- dirB_2
|-- dirC
|   `-- dirC_1
`-- ece2300-tut01.txt
```

Note that we are using the `-p` command line option with the `mkdir`
command to make multiple nested directories in a single step.

!!! question "Activity 4: Creatring Directories and Files"

    Experiment with creating additional directories and files within the
    `ece2300/tut01` subdirectory. Try creating deeper hierarchies with
    three or even four levels of nesting using the `-p` option to the
    `mkdir` command. Experiment with using the `.` and `..` special
    directories. Use the `tree` command to display your newly created
    directory hierarchy.

### 1.5. Copy, Move, and Remove Files and Directories}

We can use the `cp` command to copy files. The first argument is the
name of the file you want to copy, and the second argument is the new
name to give to the copy. The following commands will make two copies of
the files we created in the previous section.

```bash
% cd ~/ece2300/tut01
% cp ece2300-tut01.txt ece2300-tut01-a.txt
% cp ece2300-tut01.txt ece2300-tut01-b.txt
% ls -1
```

We can also copy one or more files into a subdirectory by using multiple
source files and a final destination directory as the arguments to the
`cp` command.

```bash
% cd ~/ece2300/tut01
% cp ece2300-tut01.txt dirA
% cp ece2300-tut01-a.txt ece2300-tut01-b.txt dirA
% tree
```

We can use the `-r` command line option to enable the `cp` command
to recursively copy an entire directory.

```bash
% cd ~/ece2300/tut01
% tree
% cp -r dirA dirD
% tree
```

If we want to move a file or directory, we can use the `mv` command.
As with the `cp` command, the first argument is the name of the file
you want to move and the second argument is the new name of the file.

```bash
% cd ~/ece2300/tut01
% mv ece2300-tut01.txt ece2300-tut01-c.txt
% ls -1
```

Again, similar to the `cp` command, we can also move one or more files
into a subdirectory by using multiple source files and a final
destination directory as the arguments to the `mv` command.

```bash
% cd ~/ece2300/tut01
% tree
% mv ece2300-tut01-a.txt dirB
% mv ece2300-tut01-b.txt ece2300-tut01-c.txt dirB
% tree
```

We do not need to use the `-r` command line option to move an entire
directory at once.

```bash
% cd ~/ece2300/tut01
% tree
% mv dirD dirE
% tree
```

The following example illustrates how we can use the special `.`
directory to move files from a subdirectory into the current working
directory.

```bash
% cd ~/ece2300/tut01
% tree
% mv dirE/ece2300-tut01.txt .
% tree
```

We can use the `rm` command to remove files. The following command
removes a file from within the `ece2300/tut01` subdirectory.

```bash
% cd ~/ece2300/tut01
% ls -1
% rm ece2300-tut01.txt
% ls -1
```

To clean up, we might want to remove the files we created in your home
directory earlier in this tutorial.

```bash
% cd
% rm ece2300-tut01.txt
% rm ece2300-tut01-layer1.txt
% rm ece2300-tut01-layer2.txt
% rm ece2300-tut01-layer3.txt
```

We can use the `-r` command line option with the `rm` command to remove
entire directories, but please be careful because it is relatively easy
to permanently delete many files at once. See Section 2.3 for a useful
command that you might want to use instead of the `rm` command to avoid
accidentally deleting important work.

```bash
% cd ~/ece2300/tut01
% ls -1
% rm -r dirA dirB dirC dirE
% ls -1
```

!!! question "Activity 5: Copy, Move, and Remove Directories and Files"

    Creating additional directories and files within the `ece2300/tut01`
    subdirectory, and then use the `cp`, `mv`, and `rm` commands to copy,
    move, and remove the newly created directories and files. Use the
    `ls` and `tree` commands to display your file and directory
    organization.

### 1.6. Using `wget` to Download Files

We can use the `wget` command to download files from the internet. For
now, this is a useful way to retrieve a text file that we can use in the
following examples.

```bash
% cd ~/ece2300/tut01
% wget http://www.csl.cornell.edu/courses/ece2300/overview.txt
% cat overview.txt
```

### 1.7. Using `grep` to Search Files

We can use the `grep` command to search and display lines of a file that
contain a particular pattern. The `grep` command can be useful for
quickly searching the contents of the source files in your laboratory
assignments. The command takes the pattern and the files to search as
command line arguments. The following command searches "digital logic" in
the `overview.txt` file downloaded in the previous section.

```bash
% cd ~/ece2300/tut01
% grep "digital logic" overview.txt
```

You should see just the lines within the `overview.txt` file that contain
the words "digital logic". We can use the `--line-number` command line
option with the `grep` command to display the line number of each match.

```bash
% cd ~/ece2300/tut01
% grep --line-number "digital logic" overview.txt
```

We can use the `-r` command line option to recursively search all files
within a given directory hierarchy. In the following example, we create a
subdirectory, copy the `overview.txt` file, and illustrate how we can use
the `grep` command to recursively search for the word "digital logic".

```bash
% cd ~/ece2300/tut01
% mkdir dirA
% cp overview.txt dirA
% grep -r --line-number "digital logic" .
```

Notice how we specify a directory as a command line argument (in this
case the special `.` directory) to search the current working directory.
You should see the three lines from both copies of the `overview.txt`
file. The `grep` command also shows which file contains the match.

As another example, we will search two special files named
`/proc/cpuinfo` and `proc/meminfo`. These files are present on every
modern Linux system, and they contain information about the processor and
memory hardware in that system. The following command first uses the
`less` command so you can browse the file, and then uses the `grep`
command to search for `processor` in the `/proc/cpuinfo` file. Recall
that with the `less` command, we use the up/down keys to scroll the file
one line at a time, the space bar to scroll down one page at a time, and
the `q` key to quit viewing the file.

```bash
% cd ~/ece2300/tut01
% less /proc/cpuinfo
% grep "processor" /proc/cpuinfo
```

It should be pretty clear that you are using a system with multiple
processors. You can also search to find out which company makes the
processors and what clock frequency they are running at:

```bash
% cd ~/ece2300/tut01
% grep "vendor_id" /proc/cpuinfo
% grep "cpu MHz" /proc/cpuinfo
```

We can find out how much memory is in the system by searching for
`MemTotal` in the `/proc/meminfo` file.

```bash
% cd ~/ece2300/tut01
% grep "MemTotal" /proc/meminfo
```

!!! question "Activity 6: Experimenting with `grep`"

    Try using `grep` to search for the words "computer" in the
    `overview.txt` file.

### 1.8. Using `find` to Find Files

We can use the `find` command to recursively search a directory hierarchy
for files or directories that match a specified criteria. While the
`grep` command is useful for searching file contents, the `find` command
is useful for quickly searching the file and directory names in your
laboratory assignments. The `find` command is very powerful, so we will
just show a very simple example. First, we create a few new files and
directories.

```bash
% cd ~/ece2300/tut01
% mkdir -p dirB/dirB_1
% mkdir -p dirB/dirB_2
% mkdir -p dirC/dirC_1
% echo "test" > dirA/file0.txt
% echo "test" > dirA/file1.txt
% echo "test" > dirB/dirB_1/file0.txt
% echo "test" > dirB/dirB_1/file1.txt
% echo "test" > dirB/dirB_2/file0.txt
% tree
```

We will now use the `find` command to find all files named `file0.txt`.
The `find` command takes one command line argument to specify where we
should search and a series of command line options to describe what files
and directories we are trying to find. We can also use command line
options to describe what action we would like to take when we find the
desired files and directories. In this example, we use the `--name`
command line option to specify that we are searching for files with a
specific name. We can also use more complicated patterns to search for
all files with a specific filename prefix or extension.

```bash
% cd ~/ece2300/tut01
% find . -name "file0.txt"
```

Notice that we are using the special `.` directory to tell the `find`
command to search the current working directory and all subdirectories.
The `find` command always searches recursively.

!!! question "Activity 7: Experimenting with `find`"

    Create additional files named `file2.txt` in some of the
    subdirectories we have already created. Use the `find` command to
    search for files named `file2.txt`.

### 1.9. Using `tar` to Archive Files

We can use the `tar` command to "pack" files and directories into a
simple compressed _archive_, and also to "unpack" these files and
directories from the archive. This kind of archive is sometimes called a
_tarball_. Most open-source software is distributed in this compressed
form. It makes it easy to distribute code among collaborators and it is
also useful to create backups of files. We can use the following command
to create an archive of our tutorial directory and then remove the
tutorial directory.

```bash
% cd ~/ece2300
% tar -czvf tut01.tgz tut01
% rm -r tut01
% ls -l
```

Several command line options listed together as a single option
(`-czvf`), where `c` specifies we want to create an archive, `z`
specifies we should use "gzip" compression, `v` specifies verbose mode,
and `f` specifies we will provide filenames to archive. The first command
line argument is the name of the archive to create, and the second
command line argument is the directory to archive. We can now extract the
contents of the archive to recreate the tutorial directory. We also
remove the archive.

```bash
% cd ~/ece2300
% tar -xzvf tut01.tgz
% rm tut01.tgz
% tree tut01
```

Note that we use the `x` command line option with the `tar` command to
specify that we intend to extract the archive.

!!! question "Activity 8: Experimenting with `tar`"

    Create an example directory within the `ece2300/tut01` subdirectory.
    Copy the `overview.txt` file and rename it to add example files to
    your new directory. Use the `tar` command to create and extract an
    archive of just this one new directory.

### 1.10. Using `top` to View Running Processes

You can use the `top` command to view what commands are currently running
on the Linux system in realtime. This can be useful to see if there are
many commands running which are causing the system to be sluggish. When
finished you can use the `q` character to quit.

```bash
% top
```

The first line of the `top` display shows the number of users currently
logged into the system, and the _load average_. The load average
indicates how "overloaded" the system was over the last one, five, and 15
minutes. If the load average is greater than the number of processors in
the system, it means your system will probably be sluggish. You can
always try logging out and then back into the `ecelinux` servers to see
if you get assigned to a different server in the cluster.

### 1.11. Environment Variables

In the previous sections, we have been using the Bash shell to run
various commands, but the Bash shell is actually a full-featured
programming language. One aspect of the shell that is similar in spirit
to popular programming languages, is the ability to write and read
_environment variables_. The following commands illustrate how to write
an environment variable named `ece2300_tut01_layer1`, and how to read this
environment variable using the `echo` command.

```bash
% ece2300_tut01_layer1="application"
% echo ${ece2300_tut01_layer1}
```

Keep in mind that the names of environment variables can only contain
letters, numbers, and underscores. Notice how we use the `${}` syntax to
read an environment variable. There are a few built-in environment
variables that might be useful:

```bash
% echo ${HOSTNAME}
% echo ${HOME}
% echo ${PWD}
```

We often use the `HOME` environment variable in directory paths like
this:

```bash
% cd ${HOME}/ece2300
```

The `PWD` environment variable always holds the current working
directory. We can use environment variables as options to commands other
than `echo`. A common example is to use an environment variable to
"remember" a specific directory location, which we can quickly return to
with the `cd` command like this:

```bash
% cd ${HOME}/ece2300/tut01
% TUT01=${PWD}
% cd
% pwd
/home/netid
% cd ${TUT01}
% pwd
/home/netid/ece2300/tut01
```

!!! question "Activity 9: Experimenting with Environment Variables"

    Create a new environment variable named `ece2300_tut01_layer2` and
    write it with the second layer in the computer systems stack (i.e.,
    algorithm). Use the `echo` command to display this environment
    variable. Experiment with creating a new subdirectory within
    `ece2300/tut01` and then using an environment variable to "remember"
    that location.

### 1.12. Command Output Redirection

We have already seen using the `echo` command and command output
redirection to create simple text files. Here is another example:

```bash
% cd ${HOME}/ece2300/tut01
% echo "Application" > computing-stack.txt
% cat computing-stack.txt
```

The `>` operator tells the Bash shell to take the output from the command
on the left and overwrite the file named on the right. We can use any
command on the left. For example, we can save the output from the `pwd`
command or the `man` command to a file for future reference.

```bash
% cd ${HOME}/ece2300/tut01
% pwd > cmd-output.txt
% cat cmd-output.txt
% man pwd > cmd-output.txt
% cat cmd-output.txt
```

We can also use the `>>` operator which tells the Bash shell to take the
output from the command on the left and append the file named on the
right. We can use this to create multiline text files:

```bash
% cd ${HOME}/ece2300/tut01
% echo "Application"           > computing-stack.txt
% echo "Algorithm"            >> computing-stack.txt
% echo "Programming Language" >> computing-stack.txt
% echo "Operating System"     >> computing-stack.txt
% cat computing-stack.txt
```

!!! question "Activity 10: Experimenting with Output Redirection"

    Add the remaining levels of the computing stack (i.e., compiler,
    instruction-set architecture, microarchitecture,
    register-transfer-level, gate-level, circuits, devices, technology)
    to the `computing-stack.txt` text file. Use the `cat` command to
    verify the file contents.

### 1.13. Command Chaining

We can use the `&&` operator to specify two commands that we want to
chaining together. The second command will only execute if the first
command succeeds. Below is an example.

```bash
% cd ${HOME}/ece2300/tut01 && cat computing-stack.txt
```

!!! question "Activity 11: Experimenting with Command Chaining"

    Create a single-line command that combines creating a new directory
    with the `mkdir` command and then immediately changes into the
    directory using the `cd` command.

### 1.14. Command Pipelining

The Bash shell allows you to run multiple commands simultaneously, with
the output of one command becoming the input to the next command. We can
use this to assemble "pipelines"; we "pipe" the output of one command to
another command for further actions using the `|` operator.

The following example uses the `grep` command to search the special
`proc/cpuinfo` file for lines containing the word "processor" and then
pipes the result to the `wc` command. The `wc` command counts the number
of characters, words, or lines of its input. We use the `-l` command line
option with the `wc` command to count the number of lines.

```bash
% grep processor /proc/cpuinfo | wc -l
```

This is a great example of the Linux philosophy of providing many simple
commands that can be combined to create more powerful functionality.
Essentially the pipeline we have created is a command that tells us the
number of processors in our system.

As another example, we will pipe the output of the `last` command to the
`grep` command. The `last` command lists the names of all of the users
that have logged into the system since the system was rebooted. We can
use `grep` to search for your NetID and thus quickly see how when you
previously have logged into this system.

```bash
% last | grep netid
```

We can create even longer pipelines. The following pipeline will report
the number of times you have logged into the system since it was
rebooted.

```bash
% last | grep netid | wc -l
```

!!! question "Activity 12: Experimenting with Command Pipelines"

    Use the `cat` command with the `overview.txt` file and pipe the
    output to the `grep` command to search for the word ""memories".
    While this is not as fast as using `grep` directly on the file, it
    does illustrate how many commands (e.g., `grep`) can take their input
    specified as a command line argument or through a pipe.

### 1.14. Bash Shell Scripts

If you find yourself continually having to use the same complex commands
over and over, consider creating a Bash shell script to automatically
execute those commands. A Bash shell script is just a text file with a
list of commands that you can run using the `source` command.

For example, let's create a Bash shell script to automatically grep for
information about the processor and memory in a single step. Use VS Code
to open a new file called `get-processor-memory-info.sh` (note that by
convention we usually use the `.sh` extension for Bash shell scripts).

```
% cd ${HOME}/ece2300/tut01
% code get-processor-memory-info.sh
```

Then enter the following commands into this new Bash shell script.

```
grep "processor" /proc/cpuinfo
grep "vendor_id" /proc/cpuinfo
grep "cpu MHz"   /proc/cpuinfo
grep "MemTotal"  /proc/meminfo
```

Then save the Bash shell script and execute it using the `source`
command.

```
% cd ${HOME}/ece2300/tut01
% source get-processor-memory-info.sh
```

!!! question "Activity 13: Experimenting with Bash Shell Scripts"

    Create a new Bash shell script named `wget-and-grep.sh`. The Bash
    shell script should use `wget` to get the `overview.txt` file for the
    course and then uses grep to find instances of the workd "digital
    logic" in this file. See earlier examples in this tutorial for the
    commands required for both steps. Then use `source` to execute the
    new Bash shell script.

### 1.15. Aliases, Wildcards, Command History, and Tab Completion

In this section, we describe some miscellaneous features of the Bash
shell which can potentially be quite useful in increasing your
productivity.

Aliases are a way to create short names for command sequences to make it
easier to quickly execute those command sequences in the future. For
example, assume that you frequently want to change to a specific
directory. We can create an alias to make this process take just two
keystrokes.

```bash
% alias ct="cd ${HOME}/ece2300/tut01"
% ct
% pwd
/home/academic/netid/ece2300/tut01
```

If you always want this alias to be available whenever you login to the
system, you can save it in your `.bashrc` file. The `.bashrc` is a
special Bash script that is run on every invocation of a Bash shell.

```bash
% echo "alias ct=\"cd ${HOME}/ece2300/tut01\"" >> ${HOME}/.bashrc
```

The reason we have to use a back slash (`\`) in front of the double
quotes is to make sure the `echo` command sees this command line argument
as one complete string.

Wildcards make it easy to manipulate many files and directories at once.
Whenever we specify a file or directory on the command line, we can often
use a wildcard instead. In a wildcard, the asterisk (`*`) will match any
sequence of characters. The following example illustrates how to list all
files that end in the suffix `.txt` and then copies all files that match
the wildcard from one directory to another.

```bash
% cd ${HOME}/ece2300/tut01
% ls *.txt
% cp dirA/file*.txt dirB
% tree
```

The Bash shell keeps a history of everything you do at the command line.
You can display the history with the `history` command. To rerun a
previous command, you can use the `!` operator and the corresponding
command number shown with the `history` command.

```bash
% history
```

You can pipe the output of the `history` command to the `grep` command to
see how you might have done something in the past.

```bash
% history | grep wc
```

If you press the up arrow key at the command line, the Bash shell will
show you the previous command you used. Continuing to press the up/down
keys will enable you to step through your history. It is very useful to
press the up arrow key once to rerun your last command.

The Bash shell supports tab completion. When you press the tab key twice
after entering the beginning of a filename or directory name, Bash will
try to automatically complete the filename or directory name. If there is
more than one match, Bash will show you all of these matches so you can
continue narrowing your search.

2. Course-Specific Linux Commands
--------------------------------------------------------------------------

In this section, we describe various aspects of the development
environment that are specific to the severs used in the course.

### 2.1. Course Setup Script

As discussed in the remote access tutorial, the very first thing you need
to do after logging into the `ecelinux` servers is source the course
setup script. This will ensure your environment is setup with everything
you need for working on the laboratory assignments. Enter the following
command on the command line:

```bash
% source setup-ece2300.sh
```

You should now see `ECE 2300` in your prompt which means your environment
is setup for the course.

It can be tedious to always remember to source the course setup script.
You can also use auto setup which will automatically source the course
setup for you when you open a terminal. Note that if the environment for
ECE 2300 conflicts with the environment required by a different course
then you will need to manually source the setup script when you are
working on this course. Enter the following command on the command line
to use auto setup:

```bash
% source setup-ece2300.sh --enable-auto-setup
```

Now close the terminal and log out completely from the `ecelinux`
servers. Log back in and you should see `ECE 2300` in the prompt meaning
your environment is automatically setup for the course. If at anytime you
need to disable auto setup you can use the following command:

```bash
% source setup-ece2300.sh --disable-auto-setup
```

Again, if for any reason running the setup script prevents you from using
tools for another course, you cannot use the auto setup. You will need to
run the setup script manually every time you want to work on this course.

### 2.2. Using `quota` to Check Your Space Usage

Students are allocated 10GB of storage on the servers. You can use the
following command to show much space you are using:

```bash
% quota -s
```

The `blocks` column is how much data you are using, and the `quota`
column is your quota. If you have exceed the 10GB quota, you can browse
your home directory and list the size of files and the contents of
directories with the `du` command:

```bash
% cd ${HOME}
% du -sh *
```

By recursively changing directories and examining the sizes of files and
directories you can figure out what you need to delete. We can pipe the
output of `du` to the `sort` and `head` commands to find the top 20
largest files and directories like this:

```bash
% cd ${HOME}
% du -xak . | sort -nr | head --lines=20
```

Or just use the following to generate a human readable summary of the
size of files/directories in the current working directory. Note that it
can take 20--30 seconds for this command to finish, so please be patient.

### 2.3. Using `trash` to Safely Remove Files

We have installed a simple program called `trash` which moves files you
wish to delete into a special subdirectory of your home directory located
at `${HOME}/tmp/trash`. The following commands create a file and then
deletes it using `trash`.

```bash
% cd ${HOME}
% echo "This file will be deleted." > testing.txt
% trash testing.txt
% echo "This file will also be deleted." > testing.txt
% trash testing.txt
% ls ${HOME}/tmp/trash
```

If you look in `${HOME}/tmp/trash` you will see subdirectories organized
by date. Look in the subdirectory with today's date and you should two
files corresponding to the two files you deleted. We highly recommend
always using the `trash` command instead of `rm` since this avoids
accidentally deleting your work.

