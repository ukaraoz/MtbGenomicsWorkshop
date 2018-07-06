====================================
Advanced Beginner/Intermediate Shell
====================================

Updated June 2018 by Ulas Karaoz

Credits
----------
Original authors:

Jessica Mizzi, Titus Brown, and Jessica Mizzi (http://dib-training.readthedocs.io/en/pub/2016-01-13-adv-beg-shell.html).

Learning goals:
---------------
* expose you to a bunch of syntax around shell use & scripting.
* show you the proximal possibilities of shell use & scripting.
* give you some useful tricks
* provide opportunity for discussion, so please ask questions!

-----

.. important::

  Before starting:

  1. Make sure you have the test data. Download and unpack:
  
    `https://public.etherpad-mozilla.org/p/mtb_bioinformatics_shell <https://public.etherpad-mozilla.org/p/mtb_bioinformatics_shell>`_

  2. Set your current working directory to be the top level dir, 'data/'.
  
  3. We'll be posting code snippets to: `https://public.etherpad-mozilla.org/p/mtb_bioinformatics_shell <https://public.etherpad-mozilla.org/p/mtb_bioinformatics_shell>`_

----

Exploring directory structures
------------------------------

So, if we do 'ls', we see a bunch of stuff.  We didn't create this folder.
How do we figure out what's in it?

Here, 'find' is your first friend::

   find . -type d

This walks systematically (recursively) through all files underneath '.',
finds all directories (type d), and prints them (assumed, if not other
actions).

We'll come back to 'find' later, when we use it for finding files.

----

Renaming a bunch of files
-------------------------

Let's go into the IlluminaReads directory::

  cd IlluminaReads

and take a look with `ls`.

For our first task, let's pretend that we want to change the extension of all of the fastq
files from *.fastq* to *.fq*.  Here, we get to use two commands - 'for' and 'basename'.

'for' lets you do something to every file in a list.  To see it in action::

  for i in *.fastq
  do
     echo $i
  done

This is running the command 'echo' for every value of the variable 'i', which
is set (one by one) to all the values in the expression '*.fastq'.

If we want to get rid of the extension '.fastq', we can use the 'basename'
command::

  for i in *.fastq
  do
     basename $i .fastq
  done

Now, this doesn't actually rename the files - it just prints out the name,
with the suffix '.fastq' removed.  To rename the files, we need to capture
the new name in a variable::

  for i in *.fastq
  do
     newname=$(basename $i .fastq).fq
     echo $newname
  done
  
What ``$( ... )`` does is run the command in the middle, and then replace the
``$( )`` with the value of running the command.

Now we have the old name ($i) and the new name ($newname) and we're ready
to write the rename command -- ::

  for i in *.fastq
  do
     newname=$(basename $i .fastq).fq
     echo mv $i $newname
  done

Question: why did we use 'echo' here?

Now that we're pretty sure it all looks good, let's run it for real::

  for i in *.fastq
  do
     newname=$(basename $i .fastq).fq
     mv $i $newname
  done

We have renamed all the files!

.. info:: You may see backquotes used instead of ``$(...)``. Same thing.

----

Let's also get rid of the annoying '_001' that's at the end of the
files.  basename is all fine and good with the end of files, but what
do we do about things in the middle? Now we get to use another useful command -- 'cut'.

What 'cut' does is slide and dice strings.  So, for example, ::

   echo hello, world | cut -c5-

will give you 'o, world'.

'cut' expects to take a bunch of lines of input from a file. By
default it is happy to take them in from stdin ("standard input"), so
you can specify '-' and give it some input via a pipe, which is what
we're doing with echo:

We're taking the output of 'echo hello, world' and sending it to the
input of cut with the ``|`` command ('pipe').

You've probably already seen this with head or tail, but many UNIX
commands take stdin and stdout.

Let's construct the cut command we want to use.  If we look at the names of
the files, and we want to remove 001 only, we can see that each filename
has a bunch of fields separated by '_'.  So we can ask 'cut' to pay attention
to the first four fields, and omit the fifth, around the separator (or
delimiter) '_'::

   echo S194_L001_R1_001.fq | cut -d_ -f1-4

That looks about right -- let's put it into a for loop::

  for i in *.fq
  do
     echo $i | cut -d_ -f1-4
  done

Good - now assign it to a variable and append an ending::

  for i in *.fq
  do
     newname=$(echo $i | cut -d_ -f1-4).fq
     echo $newname
  done
  
and now construct the 'mv' command::

  for i in *.fq
  do
     newname=$(echo $i | cut -d_ -f1-4).fq
     echo mv $i $newname
  done
                
and if that looks right, run it::

  for i in *.fq
  do
     newname=$(echo $i | cut -d_ -f1-4).fq
     mv $i $newname
  done

You've renamed all your files.

----

Let's do something quite useful - subset a bunch of FASTQ files.

If you look at one of the FASTQ files with head, ::

  head S188_L001_R1.fq

you'll see that it's full of FASTQ sequencing records.  Often we want
to run a bioinformatices pipeline on some small set of records first,
before running it on the full set, just to make sure all the commands work.
So we would like to subset all of these files without modifying the originals.

First, let's make sure the originals are read-only::

  chmod u-w *.fq

Now, let's make a 'subset' directory::

  mkdir subset

Now, to subset each file, we want to run a 'head' with an argument
that is the total number of lines we want to take.  In this case, it
should be a multiple of 4, because FASTQ records have 4 lines each.
So let's plan to take the first 100 lines of each file by using 'head
-400'.

The for loop will now look something like::

  for i in *.fq
  do
     echo "head -400 $i > subset/$i"
  done

If that command looks right, run it for real::

  for i in *.fq
  do
     head -400 $i > subset/$i
  done

We now have our subsets.

------------

Exercise: Can you rename all of your files in subset/ to
have 'subset.fq' at the end?

(Work in small groups; start from working code; there are several ways
to do it, all that matters is getting there.)

Backtracking
------------

Variables:

You can use either $varname or ${varname}.  The latter is useful
when you want to construct a new filename, e.g.::

   MY${varname}SUBSET

would expand ${varname} and then put MY .. SUBSET on either end, while ::

   MY$varnameSUBSET

would try to put MY in front of $varnameSUBSET which won't work.

(Unknown/uncreated variables give nothing.)

---

We used "$varname" above - what happens if we use ''?

(Variables are interpreted inside of "", and not inside of ''.)

----

Pipes and redirection:
----------------------

To redirect stdin and stdout, you can use::

  > - send stdout to a file
  < - take stdin from a file
  | - take stdout from first command and make it stdin for second command
  >> - appends stdout to a previously-existing file

stderr (errors) can be redirected::

  2> - send stderr to a file

and you can also say::

  >& - to send all output to a file

Editing on the command line:
----------------------------

Most prompts support 'readline'-style editing. This uses `emacs <https://www.tldp.org/HOWTO/pdf/Emacs-Beginner-HOWTO.pdf>`_ control keys.

Type something out; then type CTRL-a.  Now type CTRL-e.  Beginning and end...

Up arrows to recall previous command, left/right arrows, etc.

----

Another useful command along with 'basename' is 'dirname'. Any idea what it does?

-----

Working with collections of files; conditionals
-----------------------------------------------

Let's go back to the 'data' directory and play around with loops some more. ::

  cd ..

'if' acts on things conditionally::

  for i in *
  do
     if [ -f $i ]; then
        echo $i is a file
     elif [ -d $i ]; then
        echo $i is a directory
     fi
  done

but what the heck is this ``[ ]`` notation?  That's actually running
the 'test' command; try 'help test | less' to see the docs.  This is a
weird syntax that lets you do all sorts of useful things with files --
We can use it to get rid of empty files::

  touch emptyfile.txt

to create an empty file, and then::

  for i in *
  do
     if [ \! -s $i ]; then
        echo rm $i
     fi
  done

...and as you can see here, I'm using '!' to say 'not'.

Executing things conditionally based on exit status
---------------------------------------------------

Let's create two scripts (you can use 'nano' here if you want) -- in
'success.sh', put::

  #! /bin/bash
  echo mesucceed
  exit 0

and in 'fail.sh', put::

  #! /bin/bash
  echo mefail
  exit 1

You can do this with 'here documents'. A 'here document' (or 'heredoc') is a way of getting text input into a script without having to feed it in from a separate file. ::

  cat > success.sh <<EOF
  #! /bin/bash
  echo mesucceed
  exit 0
  EOF
  cat > fail.sh <<EOF
  #! /bin/bash
  echo mefail
  exit 1
  EOF

Now make them executable::

  chmod +x success.sh fail.sh

(Somewhat counterintuitively, an exit status of 0 means "success" in UNIX.)

You can now use this to chain commands with ``&&`` and ``||`` -- ::

  ./success.sh && echo this succeeded || echo this failed
  ./fail.sh && echo this succeeded || echo this failed

You can do this with R and python scripts too -- in R, you set the
exit status of a script with ``quit(status=0, save='no')`` and in
Python with ``sys.exit(0)``.  Any failure of the script due to an
exception will automatically set the exit status to non-zero.

The exit status of the previous command can be examined with ``$?``::

  ./success.sh
  if [ $? -eq 0 ]; then echo succ; fi

  ./success.sh
  if [ $? -ne 0 ]; then echo fail; fi

Writing shell scripts
---------------------

Always put 'set -e' at the top.

Sometimes put 'set -x' at the top.

You can take in command line parameters with '$1', '$2', etc. '$*' gives
you all of them at once.

Other bits and pieces
-----------------------

* Scripts exit in a subshell and can't modify your environment variables. 
If you want to modify your environment, you need to use '.' or 'source'.

* Subshells are ways to group commands with ( ... ).

* You can use \ to do line continuation in scripts (in R and Python, too!)

* `screen <https://kb.iu.edu/d/acuy>`_ lets you use multiple windows at the same time. It is extremely useful.

* History tricks::

  !! - run previous command
  !-1 - run command-before-previous command (!-2 etc.)
  !$ - replace with the last word on the previous line
  !n - run the nth command in your 'history'


A general approach you can use
------------------------------

* Break your task down into multiple commands
* Put commands in shell scripts, run in serial
* Use intermediate i/o files to figure out what's going on!
* Use echo to debug!

The weird awesomeness that is 'find'
------------------------------------

Here is more useful things you can do with 'find' command:

Print all files::
  
   find . -type f

Print all files w/details::

   find . -type f -ls

Find all files not in git directories::

   find . -name .git -prune -o -type f -print

Find all directories in the current directory::

   find * -prune -type d -print

...and get their disk usage::

   find * -prune -type d -exec du -skh {} \;

Here, '-exec' runs the command specified up until the ``\;``, and replaces
the {} with the filename.

Same result, different command::

   find . -depth 1 -type d -exec du -skh {} \;

Find all files larger than 100k::

   find . -size +100k -print

Find all files that were changed within the last 10 minutes::

  find . -ctime -10m

(...and do things to them with -exec ;).

Run 'grep -l' to find all files containing the string 'CGTTATCCGGATTTATTGGGTTTA'::

  find . -type f -exec grep -q CGTTATCCGGATTTATTGGGTTTA {} \; -print

Note, you can use -a (and) and -o (or), along with ``\(`` and ``\)``,
to group conditions::

  find . \( \( -type f -size +100k \) -o \( -type f -size -1k \) \)  -print
  
...so it's basically all programming...

Note that you can 'exec' a Python, R, or shell script.

Exercise: how would you copy all files containing a specific string
('CGTTATCCGGATTTATTGGGTTTA', say) into a new directory? And what are the
pros (and cons) of your approach?

(Work in small groups; start from working code, say, the 'find'
command above; there are several ways to do it, all that matters is
getting there.)