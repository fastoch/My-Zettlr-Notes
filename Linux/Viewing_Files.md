## cat

The `cat` command is ideal for displaying small files: `cat filename_or_filepath`  
Use the `-n` option to display line numbers: `cat -n filename`  

`cat` can also be used to con**cat**enate files: `cat filename1 filename2 > output_file`  

### Redirection

Use `>` to redirect the output of a command to the specified file.  
It will create the file if not existing, or will overwrite the contents of the file if already existing.  

Use `>>` to append the result (add it at the end) instead of overwriting any pre-existing content.

## less (is more)

If the output of a text file exceeds the terminal window, it's best to use `less` or `more` instead of `cat`.  
Both tools are known as "pagers", they take the input from a file and send it to the terminal one page at a time.  

`less` extends the capabilities of `more` and is probably the most advanced reading tool that Linux provides.  
The syntax is `less filename`  
To exit the file reading, type **q**.

`less` is what is used when we display man pages.

While in the file, you can use the arrow keys to move one line up or down.  
To forward one page, press **Ctrl + F**, and **Ctrl + B** to move one page backward.  
To go to the beginning of a file, press lowercase **g**, and uppercase **G** to go to the very end.  

To search the current file for a specific string:
- move at the beginning by pressing g
- type a forward slash / followed by the string you're looking for
- then navigate between matches with n (next occurrence) or N (previous occurrence)

## tail, head, watch



