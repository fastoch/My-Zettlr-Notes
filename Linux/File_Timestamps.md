Every file on Linux has 3 timestamps:
- the access timestamp or **atime** is the last time the file was read (`ls- lu`)
- the modified timestamp or **mtime** is the last time the contents were modified (`ls -l`, `ls -lt`)
- the changed timestamp or **ctime** is the last time some metadata related to the file was changed (`ls -lc`)

You can display all 3 timestamps with the `stat` command: `stat <filename>`.

We can change the timestamp on a file using the `touch` command.
This command creates a file if it doesn't exist yet: `touch unexisting_file`
But if it exists, it will update all the file's timestamps to the computer's current time.

To only change the access time: `touch -a filename`
This will also change the ctime because atime is part of the file's metadata.

To change the mtime: `touch -m filename`
That will also change the ctime…

We can also set the atime or mtime to a specific time.
`touch -m -t 201806231340.52 filename`

To modify both the access and modification time: 
`touch -d "2016-12-25 16:32:30" filename`

To set the timestamps of a file A to those of file B:
`touch file_A -r file_B`

To modify only the ctime (change time) to a specific time, we need to use a trick:
- run `stat filename` and note the atime and mtime
- set the system date and time to the desired ctime
- run `touch filename` on the file you want to modify the ctime of
- then change the atime and mtime back to their initial values
- set the system date and time back to their actual values

Additional tip:
- `date` displays the current date and time for your area
- `date -u` displays the current UTC date and time (coordinated universal time)