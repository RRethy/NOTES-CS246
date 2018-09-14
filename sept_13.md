# Sept 13 - CS246

## Using output of program as arg to another program

* Eg. `echo "Today is $(date) and I am $(whoami)`
  * `date` and `whoami` are both commands
* Using double quotes for arguments
  1. echo Today is $(date) and I am $(whoami)
    * This is multiple arguments but they still all get printed
  2. echo "Today is $(date) and I am $(whoami)"
    * Single arg
  3. echo 'Today is $(date)'
    * Output: Today is $(date)
    * Single quotes will suppress embedded commands
  4. echo `date`
    * It will print the date. `date` is treated as a command and is evaluated
    * Older syntax, won't be used much

## Searching within text files

`man grep`

* tool: egrep (grep -E)
* print every line in files provided as args that contain the regex pattern

### Regex additional examples

* Match zero or one of the precending identifier
  1. To match `cs246` and `cs 246`
    * `cs ?246`
  2. To match `cs246` and `246`
    * `(cs)?246`
3. Match zero or more of the preceding identifier
  * `*`
  * Not the same as globbing wildcard
  * `(cs)*246`
    * Match `cs246` `cscs246` `246`
4. Match any character (only a single char)
  * `.`
  * To match any number of anything
    * `.*`
5. To search for special identifiers you can escape with `\`
  * `\(\)` will match `()`
6. Match one or more occurences
  * `+`
  * `sc+` will match `sc` `sccc` `sccccc` but not `s`
7. Match the start of the line
  * `^`
  * `^tobe` will match `tobesssss` but not `sssstobe`
8. Match end of the line
  * `$`
  * `tobe$` will match `ddddddtobe$` but not `dddtobeddd`
  * `^tobe$` will match `tobe` but not `dddtobeddd`

### Examples

1. `grep searchtext myfile.txt`
  * Search for `searchtext` in myfile.txt
2. `grep "searchtext|Searchtext" myfile.txt`
  * Or `grep "(s|S)earchtext" myfile.txt`
  * Search for `searchtext` or `Searchtext` in myfile.txt
3. `grep "[sS]earchtext" mufile.txt`
  * Same as above, `[]` mean to match any char between them
  * `[^sS]` means to match not `s` and not `S`
3. `grep -i "searchtext" myfile.txt`
  * Same as above, but `-i` forces case-insensitivity
4. Print all words from /usr/share/dict/words that start with an e and consist of 5 characters
  * `egrep "^e.{4}$" /usr/share/dict/words`
  * `egrep "^e....$" /usr/share/dict/words`
5. Lines of even length
  * `egrep "^(..)*$" /usr/share/dict/words`
6. Filenames from cur directory whose name contains exactly one `a`
  * `ls | egrep "^[^a]*a[^a]*$"`

## File Permissions

* `ls -l` - long listing
  * Eg.
  ```
  -rw-r--r--  1 rethy  staff  2433 Sep 13 15:29 sept_13.md
  -rw-r--r--  1 rethy  staff  1351 Sep  9 15:37 sept_6.md
  ```
  * see below - shortcut - owner - group - size - date last modified - filename
  * `-rw-r--r--`
    * first bit - type of file - ordinary or directory (`-` for ordinary file, `d` for directory)
    * Nine following bits of three bits each (owner bits - single owner, group bits - all users that belong to the same group to which the file belongs, other bits - all other users not in group)
    * r (read) w (write) x (execute)
  * Ordinary files
    * `r` read the file, cat the file
    * `w` modify the contents
    * `x` can execute the program
  * Directories
    * `r` ls, tab completion
    * `w` add/remove files
    * `x` can search the directory, enter it, navigate into it

### Changing permissions

  * `chmod {mode} {file}+`
    * Add or remove mode to file
    * `u` (owner) `g` (group) `o` (other) `a` (all three)
    * `chmod a+x {file}+` - add executable permission for everyone
    * `chmod u-rw {file}+` - remove read/write permission from owner
    * `chmod u= {file}+` - remove all permissions from owner (still has permission to re-add these permissions)
