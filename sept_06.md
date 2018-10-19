# Linux shell

## Shell

* interface to the operating system
* Two type of shells:
  1. **Graphical shell** - Click to select, drag and drop. Easy to learn but restrictive.
  2. **Command-line shell** - Type commands on a prompt. Powerful but steep learning curve.

## Unix

* built in 70s
* Came with command-line interface (shell)
* Stephen Bourne
* Other popular shells
  * C Shell (csh) -> Turbo C Shell (tcsh)
  * Korn Shell (ksh)
  * Original *Shell* became Bourne Again Shell (bash)
* Run `echo $0` to see which shell you are using. You should be using `bash` (or `zsh` which is mostly bash compatible)

## Linux file system

* Directories - just files that can contain other files
* Non-directory files - *ordinary files*
* File system is a tree structure
  * The root of the tree is the *root directory* (`/`)
* **Path**: Way to specify the location of any file in the file system
  * Eg. /usr/share/dict/words

## Commands

* `ls` - listings
  * lists all non-hidden files in the directory
  * **hidden files**: name starts with a dot
    * `ls -a` to view hidden files too (all files)
  * **Special Files**:
    * `.` is the current directory
    * `..` is the parent directory
* `pwd` - present working directory
* `cd` - change directory
  * `cd {path}` where {path} is a path to a directory
    * Allows absolute paths or relative paths
