# Shell Scripts

* **Scrip** - a test file containing a sequence of commands executed as a program

> Assume we have a shell script named basic

* Starts with `#!/bin/bash` (she-bang slash bin slash bash slash) - shebang line
* To run script `basic`: `./basic`.
	* Must put path to file since bash will only look through your `$PATH`
	* Can also use `$(pwd)/basic`

## Command line arguments

* `$ ./script arg1 arg2`
	* $1 contains arg1
	* $2 contains arg2
	* ...
* `${#}` is the number of args to the script
* `$0` is the name of the script (what comes before the first command which is also `$1`)

## Example

> Assume we have a script isItAWord

```sh
#!/bin/bash

# Search for the first arg in the dictionary
# Redirect output to blackhole
# egrep is smart enough to know $1 should be expanded
egrep "^$1$" /usr/share/dict/words > /dev/null

if [ $? -eq 0 ]; then
	echo $1 is not a good password
else
	echo $1 might be a good password
fi
```

* **Notes**: User needs to escape whatever special characters that get used
* **Notes**: Every command exits with a status code. Zero for success, non-zero for failure
	* `$?` is the status code for the last process
* **Notes**: The square bracket are themselves a program try `[ 1 -ne 2 ] ; echo $?`
	* The white space in the square brackets are **important**
	* Name of program is `[`, the reset are all arguments
* Take a look at the shell scripts dir

## Scripts

* See 1189/lectures/..../goodPasswordUsage
	* the functions (subroutines) are not type safe
* All variables are of type string

### More examples

```sh
if [ condition ]; then
	block
elif [ condition ]; then
	block
else
	block
fi
```

```sh
# No spaces between variable assignment
x=1
while [ ${x} -le $1 ]; do
	echo ${x}
	x=$((x + 1))
done
```

```sh
for x in a b c d; do echo ${x}; done

for x in a b c d; do
  echo ${x}
done
```

```sh
# Rename all .cpp files to .cc

# Use globbing patterns to find files
for name in *.cpp; do
	# Remove the trailing pp and add a c
	mv ${name} ${name%pp}c
done
```
