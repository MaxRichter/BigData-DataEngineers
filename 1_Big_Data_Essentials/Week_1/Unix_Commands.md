# Unix Command Line Interface (CLI)

* ls [OPTION]... [FILE]... - Lists the contents of a directory
    * ls -l (long)
    * ls -lh (human readable format)
    * ls -lSh (sorted by size)
* cd [-L|-P] directory - change directory
    * cd anaconda/
    * cd / (one step up)
    * cd .. (one step back)
    * cd ~ (home directory)
* pwd [OPTION] - print the whole pathname of the current directory
* du [OPTION]... [FILE]... - estimates and displays the disk space used by the files
    * du -h (size in human readable format)
* df [OPTION]... [FILE]... - reports the amount of available disk space being used by the file systems
    * df -h (size in human readable format)
* man - is the interface used to view the system’s reference manuals
    * man pwd
    * to exit manual type “q”
* mkdir [OPTION]... DIRECTOR... - “make directory”, creates directories on a file system
    * mkdir my_new_dir
* cp [OPTION]... SOURCE... DIRECTORY - makes copies of files and directories
    * cp file.txt copy_of_file.txt
    * cp -r (copy of a directory)
    * cp -r my_new_dir/ copy_of_my_new_dir
* mv [OPTION]... SOURCE... DIRECTORY (moves or renames files)
    * mv copy_of_file.txt my_new_dir/file.txt
    * mv -r (move a directory)
* rm [OPTION]... FILE... - removes (deletes) files or directories
    * rm copy_of_file.txt
    * rm -r copy_of_my_new_dir/
* touch [OPTION]... FILE... - changes (update) file timestamp
    * ls -la (list and showing last file access)
    * touch file.txt (change timestamp)
    * ls -la (timestamp for this file has changed)
    * touch new_empty_file.txt (new empty file created in the current directory)
* cat [OPTION]... [FILE]... 
    * Displays the contents of a file at the command line
    * Copies or append text files into a document
    * cat new_file.txt → text…
    * cat file.txt multilined_file.txt (print output of several files)
* head [OPTION]... [FILE]... - prints the first part of files
    * head multilined_file.txt
    * head -n (n is integer and is number of lines to print)
    * head -7 multilned_file.txt
* tail [OPTION]... [FILE]... - prints the last part of files (10)
    * tail multilined_file.txt
* more [FILE]... - displays text, one screen at a time
* less [FILE]... - displays text, allows scrolling (fast for large files)
* wc [OPTION]... [FILE]... - “word count” prints a count of lines, words, and characters for each file
    * wc multilined_file.txt
    * wc -c (only characters) etc.
* grep [OPTIONS] PATTERN [FILE]... - “global regular expression print” processes text and prints any lines which match a specified pattern
    * grep ‘15’ multilined_file.txt (searches for 15 in file)
* vim [OPTIONS] [filelist] - an advanced text editor
    * to start vim: “vim filename”
    * to edit a file: “:i” (insert)
    * to save file and quit vim: “:wq”
    * to quit vim without saving the file: “:!q”
    * .vimrc file with vim configuration
        * colorscheme darkblue
        * set tabstop=4
        * set shiftwidth=4
        * set smarttab
        * set expandtab
* cut [OPTIONS]... [FILE]... - drops sections of each line of a file/files
    * cut -d “ “ -f1 multilined_file.txt (-delimiter; -f: column)
* tr [-Ccsu] string1 string2 - translates one set of characters to another
    * cat multilined_file.txt | tr “ “ “.” (replaces whitespace with “.”; first commands reads in the file and then tr is applied on it using |)
    * cat multilined_file.txt | tr -d “ “ (drops whitespaces)
* sort [OPTION]... [FILE]... - sorts the contents of a text file
    * sort -r (reverse order)
    * sort -r -k (k is the column)
    * sort -k2,2 -r multilined_file.txt
    * sort -r -k -n (n is for numeric searching)
    * sort -k2,2 -n -r multilined_file.txt
* awk [-F fs] [-v var=value] [‘prog’ | -f progfile] [FILE]...
    * interpreted programming language text processing
    * awk -F ‘ ‘ ‘{print $1}’ multilined_file.txt (print first field with $1)
    * awk -F ‘ ‘ ‘{print $1”s”, $2*10}’ multilined_file.txt (add “s” to column 1 and multiply column 2 by 10)
* operators
    * \> overwrites the file if it exists or creates it if it does not exist
        * ls > list_of_my_filex.txt
        * cat list_of_my_files.txt
    * \>> appends to a file or creates the file if it does not exist
        * ls >> list_of_my_files_txt
    * | - pipe operator, output of first command acts as an input to the second command
        * cat multilined_file.txt | sort | uniq -c (how many times each string is repeated in our file)
        * cat multilined_file.txt | cut -d “ “ -f1 | sort | uniq -c
    * & - make the command run in the background
    * || - execute the second command only, if the execution of first command succeeds
    * && - execute second command only if the execution of the first command fails
    * free [OPTIONS] - displays the total amount of free and used memory
        * free -g (size in gigabytes)
    * top [OPTIONS] - provides a dynamic real-time view of a running system
    * ps [OPTIONS] - provides snapshot of the status of currently running processes
        * ps a (print out all processes)
        * kill [-s] [-l] %pid - sends a signal to a process
        * ps au (to get PID)
        * kill -9 25960
        * ps au (check if process is really killed)
    * nice [OPTION] [COMMAND [ARG]...] - runs a command with a modified scheduling priority
