---
description: >-
  This is a quick reference cheat sheet to getting started with linux bash shell
  scripting.
---

# Bash

### [\#](https://quickref.me/bash#getting-started)Getting started <a id="getting-started"></a>

#### hello.sh <a id="hello-sh"></a>

```text
#!/bin/bash

VAR="world"
echo "Hello $VAR!" # => Hello world!
```

Execute the script

```text
$ bash hello.sh
```

#### Variables <a id="variables"></a>

```text
NAME="John"

echo ${NAME}    # => John
echo $NAME      # => John
echo "$NAME"    # => John
echo '$NAME'    # => $NAME
echo "${NAME}!" # => John!

NAME = "John" # => Error (about space)
```

#### Comments <a id="comments"></a>

```text
# This is an inline Bash comment.
```

```text
: '
This is a
very neat comment
in bash
'
```

Multi-line comments use `:'` to open and `'` to close

#### Arguments <a id="arguments"></a>

| `$1` … `$9` | Parameter 1 ... 9 |
| :--- | :--- |
| `$0` | Name of the script itself |
| `$1` | First argument |
| `${10}` | Positional parameter 10 |
| `$#` | Number of arguments |
| `$$` | Process id of the shell |
| `$*` | All arguments |
| `$@` | All arguments, starting from first |
| `$-` | Current options |
| `$_` | Last argument of the previous command |

See: [Special parameters](http://wiki.bash-hackers.org/syntax/shellvars#special_parameters_and_shell_variables).

#### Functions <a id="functions"></a>

```text
get_name() {
    echo "John"
}

echo "You are $(get_name)"
```

See: [Functions](https://quickref.me/bash#functions-2)

#### Conditionals <a id="conditionals"></a>

```text
if [[ -z "$string" ]]; then
    echo "String is empty"
elif [[ -n "$string" ]]; then
    echo "String is not empty"
fi
```

See: [Conditionals](https://quickref.me/bash#conditionals-2)

#### Brace expansion <a id="brace-expansion"></a>

```text
echo {A,B}.js
```

| `{A,B}` | Same as `A B` |
| :--- | :--- |
| `{A,B}.js` | Same as `A.js B.js` |
| `{1..5}` | Same as `1 2 3 4 5` |

See: [Brace expansion](http://wiki.bash-hackers.org/syntax/expansion/brace)

#### Shell execution <a id="shell-execution"></a>

```text
echo "I'm in $(PWD)"
# Same
echo "I'm in `pwd`"
```

See: [Command substitution](http://wiki.bash-hackers.org/syntax/expansion/cmdsubst)

### [\#](https://quickref.me/bash#parameter-expansions)Parameter expansions <a id="parameter-expansions"></a>

#### Syntax <a id="syntax"></a>

| `${FOO%suffix}` | Remove suffix |
| :--- | :--- |
| `${FOO#prefix}` | Remove prefix |
| `${FOO%%suffix}` | Remove long suffix |
| `${FOO##prefix}` | Remove long prefix |
| `${FOO/from/to}` | Replace first match |
| `${FOO//from/to}` | Replace all |
| `${FOO/%from/to}` | Replace suffix |
| `${FOO/#from/to}` | Replace prefix |

**Substrings**

| `${FOO:0:3}` | Substring _\(position, length\)_ |
| :--- | :--- |
| `${FOO:(-3):3}` | Substring from the right |

**Length**

| `${#FOO}` | Length of `$FOO` |
| :--- | :--- |


**Default values**

| `${FOO:-val}` | `$FOO`, or `val` if unset |
| :--- | :--- |
| `${FOO:=val}` | Set `$FOO` to `val` if unset |
| `${FOO:+val}` | `val` if `$FOO` is set |
| `${FOO:?message}` | Show message and exit if `$FOO` is unset |

#### Substitution <a id="substitution"></a>

```text
echo ${food:-Cake}  #=> $food or "Cake"
```

```text
STR="/path/to/foo.cpp"
echo ${STR%.cpp}    # /path/to/foo
echo ${STR%.cpp}.o  # /path/to/foo.o
echo ${STR%/*}      # /path/to

echo ${STR##*.}     # cpp (extension)
echo ${STR##*/}     # foo.cpp (basepath)

echo ${STR#*/}      # path/to/foo.cpp
echo ${STR##*/}     # foo.cpp

echo ${STR/foo/bar} # /path/to/bar.cpp
```

#### Slicing <a id="slicing"></a>

```text
name="John"
echo ${name}           # => John
echo ${name:0:2}       # => Jo
echo ${name::2}        # => Jo
echo ${name::-1}       # => Joh
echo ${name:(-1)}      # => n
echo ${name:(-2)}      # => hn
echo ${name:(-2):2}    # => hn

length=2
echo ${name:0:length}  # => Jo
```

See: [Parameter expansion](http://wiki.bash-hackers.org/syntax/pe)

#### basepath & dirpath <a id="basepath-dirpath"></a>

```text
SRC="/path/to/foo.cpp"
```

```text
BASEPATH=${SRC##*/}   
echo $BASEPATH  # => "foo.cpp"


DIRPATH=${SRC%$BASEPATH}
echo $DIRPATH   # => "/path/to/"
```

#### Transform <a id="transform"></a>

```text
STR="HELLO WORLD!"
echo ${STR,}   # => hELLO WORLD!
echo ${STR,,}  # => hello world!

STR="hello world!"
echo ${STR^}   # => Hello world!
echo ${STR^^}  # => HELLO WORLD!

ARR=(hello World)
echo "${ARR[@],}" # => hello world
echo "${ARR[@]^}" # => Hello World
```

### [\#](https://quickref.me/bash#arrays)Arrays <a id="arrays"></a>

#### Defining arrays <a id="defining-arrays"></a>

```text
Fruits=('Apple' 'Banana' 'Orange')

Fruits[0]="Apple"
Fruits[1]="Banana"
Fruits[2]="Orange"

ARRAY2=(foo{1..2}) # => foo1 foo2
ARRAY3=({A..D})    # => A B C D

# declare construct
declare -a Numbers=(1 2 3 4 5 6)
```

#### Indexing <a id="indexing"></a>

| `${Fruits[0]}` | First element |
| :--- | :--- |
| `${Fruits[-1]}` | Last element |
| `${Fruits[*]}` | All elements |
| `${Fruits[@]}` | All elements |
| `${#Fruits[@]}` | Number of all |
| `${#Fruits}` | Length of 1st |
| `${#Fruits[3]}` | Length of nth |
| `${Fruits[@]:3:2}` | Range |
| `${!Fruits[@]}` | Keys of all |

#### Iteration <a id="iteration"></a>

```text
Fruits=('Apple' 'Banana' 'Orange')

for e in "${Fruits[@]}"; do
    echo $e
done
```

**With index**

```text
for i in "${!Fruits[@]}"; do
  printf "%s\t%s\n" "$i" "${Fruits[$i]}"
done

```

#### Operations <a id="operations"></a>

```text
Fruits=("${Fruits[@]}" "Watermelon")     # Push
Fruits+=('Watermelon')                   # Also Push
Fruits=( ${Fruits[@]/Ap*/} )             # Remove by regex match
unset Fruits[2]                          # Remove one item
Fruits=("${Fruits[@]}")                  # Duplicate
Fruits=("${Fruits[@]}" "${Veggies[@]}")  # Concatenate
lines=(`cat "logfile"`)                  # Read from file
```

#### Arrays as arguments <a id="arrays-as-arguments"></a>

```text
function extract()
{
    local -n myarray=$1
    local idx=$2
    echo "${myarray[$idx]}"
}
Fruits=('Apple' 'Banana' 'Orange')
extract Fruits 2     # => Orangle
```

### [\#](https://quickref.me/bash#dictionaries)Dictionaries <a id="dictionaries"></a>

#### Defining <a id="defining"></a>

```text
declare -A sounds
```

```text
sounds[dog]="bark"
sounds[cow]="moo"
sounds[bird]="tweet"
sounds[wolf]="howl"
```

#### Working with dictionaries <a id="working-with-dictionaries"></a>

```text
echo ${sounds[dog]} # Dog's sound
echo ${sounds[@]}   # All values
echo ${!sounds[@]}  # All keys
echo ${#sounds[@]}  # Number of elements
unset sounds[dog]   # Delete dog
```

#### Iteration <a id="iteration-2"></a>

```text
for val in "${sounds[@]}"; do
    echo $val
done
```

```text
for key in "${!sounds[@]}"; do
    echo $key
done
```

### [\#](https://quickref.me/bash#conditionals-2)Conditionals <a id="conditionals-2"></a>

#### Integer conditions <a id="integer-conditions"></a>

| `[[ NUM -eq NUM ]]` | Equal |
| :--- | :--- |
| `[[ NUM -ne NUM ]]` | Not equal |
| `[[ NUM -lt NUM ]]` | Less than |
| `[[ NUM -le NUM ]]` | Less than or equal |
| `[[ NUM -gt NUM ]]` | Greater than |
| `[[ NUM -ge NUM ]]` | Greater than or equal |
| `(( NUM < NUM ))` | Less than |
| `(( NUM <= NUM ))` | Less than or equal |
| `(( NUM > NUM ))` | Greater than |
| `(( NUM >= NUM ))` | Greater than or equal |

#### String conditions <a id="string-conditions"></a>

| `[[ -z STR ]]` | Empty string |
| :--- | :--- |
| `[[ -n STR ]]` | Not empty string |
| `[[ STR == STR ]]` | Equal |
| `[[ STR = STR ]]` | Equal \(Same above\) |
| `[[ STR < STR ]]` | Less than _\(ASCII\)_ |
| `[[ STR > STR ]]` | Greater than _\(ASCII\)_ |
| `[[ STR != STR ]]` | Not Equal |
| `[[ STR =~ STR ]]` | Regexp |

#### Example <a id="example"></a>

**String**

```text
if [[ -z "$string" ]]; then
    echo "String is empty"
elif [[ -n "$string" ]]; then
    echo "String is not empty"
else
    echo "This never happens"
fi
```

**Combinations**

```text
if [[ X && Y ]]; then
    ...
fi
```

**Equal**

```text
if [[ "$A" == "$B" ]]; then
    ...
fi
```

**Regex**

```text
if [[ '1. abc' =~ ([a-z]+) ]]; then
    echo ${BASH_REMATCH[1]}
fi
```

**Smaller**

```text
if (( $a < $b )); then
   echo "$a is smaller than $b"
fi
```

**Exists**

```text
if [[ -e "file.txt" ]]; then
    echo "file exists"
fi
```

#### File conditions <a id="file-conditions"></a>

| `[[ -e FILE ]]` | Exists |
| :--- | :--- |
| `[[ -d FILE ]]` | Directory |
| `[[ -f FILE ]]` | File |
| `[[ -h FILE ]]` | Symlink |
| `[[ -s FILE ]]` | Size is &gt; 0 bytes |
| `[[ -r FILE ]]` | Readable |
| `[[ -w FILE ]]` | Writable |
| `[[ -x FILE ]]` | Executable |
| `[[ f1 -nt f2 ]]` | f1 newer than f2 |
| `[[ f1 -ot f2 ]]` | f2 older than f1 |
| `[[ f1 -ef f2 ]]` | Same files |

#### More conditions <a id="more-conditions"></a>

| `[[ -o noclobber ]]` | If OPTION is enabled |
| :--- | :--- |
| `[[ ! EXPR ]]` | Not |
| `[[ X && Y ]]` | And |
| `[[ X || Y ]]` | Or |

#### logical and, or <a id="logical-and-or"></a>

```text
if [ "$1" = 'y' -a $2 -gt 0 ]; then
    echo "yes"
fi

if [ "$1" = 'n' -o $2 -lt 0 ]; then
    echo "no"
fi
```

### [\#](https://quickref.me/bash#loops)Loops <a id="loops"></a>

#### Basic for loop <a id="basic-for-loop"></a>

```text
for i in /etc/rc.*; do
    echo $i
done
```

#### C-like for loop <a id="c-like-for-loop"></a>

```text
for ((i = 0 ; i < 100 ; i++)); do
    echo $i
done
```

#### Ranges <a id="ranges"></a>

```text
for i in {1..5}; do
    echo "Welcome $i"
done
```

**With step size**

```text
for i in {5..50..5}; do
    echo "Welcome $i"
done
```

#### Auto increment <a id="auto-increment"></a>

```text
i=1
while [[ $i -lt 4 ]]; do
    echo "Number: $i"
    ((i++))
done
```

#### Auto decrement <a id="auto-decrement"></a>

```text
i=3
while [[ $i -gt 0 ]]; do
    echo "Number: $i"
    ((i--))
done
```

#### Continue <a id="continue"></a>

```text
for number in $(seq 1 3); do
    if [[ $number == 2 ]]; then
        continue;
    fi
    echo "$number"
done
```

#### Break <a id="break"></a>

```text
for number in $(seq 1 3); do
    if [[ $number == 2 ]]; then
        # Skip entire rest of loop.
        break;
    fi
    # This will only print 1
    echo "$number"
done
```

#### Until <a id="until"></a>

```text
count=0
until [ $count -gt 10 ]; do
    echo "$count"
    ((count++))
done
```

#### Forever <a id="forever"></a>

```text
while true; do
    # here is some code.
done
```

#### Forever \(shorthand\) <a id="forever-shorthand"></a>

```text
while :; do
    # here is some code.
done
```

#### Reading lines <a id="reading-lines"></a>

```text
cat file.txt | while read line; do
    echo $line
done
```

### [\#](https://quickref.me/bash#functions-2)Functions <a id="functions-2"></a>

#### Defining functions <a id="defining-functions"></a>

```text
myfunc() {
    echo "hello $1"
}
```

```text
# Same as above (alternate syntax)
function myfunc() {
    echo "hello $1"
}
```

```text
myfunc "John"
```

#### Returning values <a id="returning-values"></a>

```text
myfunc() {
    local myresult='some value'
    echo $myresult
}
```

```text
result="$(myfunc)"
```

#### Raising errors <a id="raising-errors"></a>

```text
myfunc() {
    return 1
}
```

```text
if myfunc; then
    echo "success"
else
    echo "failure"
fi
```

### [\#](https://quickref.me/bash#options)Options <a id="options"></a>

#### Options <a id="options-2"></a>

```text
# Avoid overlay files
# (echo "hi" > foo)
set -o noclobber

# Used to exit upon error
# avoiding cascading errors
set -o errexit   

# Unveils hidden failures
set -o pipefail  

# Exposes unset variables
set -o nounset
```

#### Glob options <a id="glob-options"></a>

```text
# Non-matching globs are removed  
# ('*.foo' => '')
shopt -s nullglob   

# Non-matching globs throw errors
shopt -s failglob  

# Case insensitive globs
shopt -s nocaseglob 

# Wildcards match dotfiles 
# ("*.sh" => ".foo.sh")
shopt -s dotglob    

# Allow ** for recursive matches 
# ('lib/**/*.rb' => 'lib/a/b/c.rb')
shopt -s globstar   
```

### [\#](https://quickref.me/bash#history)History <a id="history"></a>

#### Commands <a id="commands"></a>

| `history` | Show history |
| :--- | :--- |
| `shopt -s histverify` | Don't execute expanded result immediately |

#### Expansions <a id="expansions"></a>

| `!$` | Expand last parameter of most recent command |
| :--- | :--- |
| `!*` | Expand all parameters of most recent command |
| `!-n` | Expand `n`th most recent command |
| `!n` | Expand `n`th command in history |
| `!<command>` | Expand most recent invocation of command `<command>` |

#### Operations <a id="operations-2"></a>

| `!!` | Execute last command again |
| :--- | :--- |
| `!!:s/<FROM>/<TO>/` | Replace first occurrence of `<FROM>` to `<TO>` in most recent command |
| `!!:gs/<FROM>/<TO>/` | Replace all occurrences of `<FROM>` to `<TO>` in most recent command |
| `!$:t` | Expand only basename from last parameter of most recent command |
| `!$:h` | Expand only directory from last parameter of most recent command |

`!!` and `!$` can be replaced with any valid expansion.

#### Slices <a id="slices"></a>

| `!!:n` | Expand only `n`th token from most recent command \(command is `0`; first argument is `1`\) |
| :--- | :--- |
| `!^` | Expand first argument from most recent command |
| `!$` | Expand last token from most recent command |
| `!!:n-m` | Expand range of tokens from most recent command |
| `!!:n-$` | Expand `n`th token to last from most recent command |

`!!` can be replaced with any valid expansion i.e. `!cat`, `!-2`, `!42`, etc.

### [\#](https://quickref.me/bash#miscellaneous)Miscellaneous <a id="miscellaneous"></a>

#### Numeric calculations <a id="numeric-calculations"></a>

```text
$((a + 200))      # Add 200 to $a
```

```text
$(($RANDOM%200))  # Random number 0..199
```

#### Subshells <a id="subshells"></a>

```text
(cd somedir; echo "I'm now in $PWD")
pwd # still in first directory
```

#### Inspecting commands <a id="inspecting-commands"></a>

```text
command -V cd
#=> "cd is a function/alias/whatever"
```

#### Redirection <a id="redirection"></a>

```text
python hello.py > output.txt   # stdout to (file)
python hello.py >> output.txt  # stdout to (file), append
python hello.py 2> error.log   # stderr to (file)
python hello.py 2>&1           # stderr to stdout
python hello.py 2>/dev/null    # stderr to (null)
python hello.py &>/dev/null    # stdout and stderr to (null)
```

```text
python hello.py < foo.txt      # feed foo.txt to stdin for python
```

#### Source relative <a id="source-relative"></a>

```text
source "${0%/*}/../share/foo.sh"
```

#### Directory of script <a id="directory-of-script"></a>

```text
DIR="${0%/*}"
```

#### Case/switch <a id="case-switch"></a>

```text
case "$1" in
    start | up)
    vagrant up
    ;;

    *)
    echo "Usage: $0 {start|stop|ssh}"
    ;;
esac
```

#### Trap errors <a id="trap-errors"></a>

```text
trap 'echo Error at about $LINENO' ERR
```

or

```text
traperr() {
    echo "ERROR: ${BASH_SOURCE[1]} at about ${BASH_LINENO[0]}"
}

set -o errtrace
trap traperr ERR
```

#### printf <a id="printf"></a>

```text
printf "Hello %s, I'm %s" Sven Olga
#=> "Hello Sven, I'm Olga

printf "1 + 1 = %d" 2
#=> "1 + 1 = 2"

printf "Print a float: %f" 2
#=> "Print a float: 2.000000"
```

#### Getting options <a id="getting-options"></a>

```text
while [[ "$1" =~ ^- && ! "$1" == "--" ]]; do case $1 in
    -V | --version )
    echo $version
    exit
    ;;
    -s | --string )
    shift; string=$1
    ;;
    -f | --flag )
    flag=1
    ;;
esac; shift; done
if [[ "$1" == '--' ]]; then shift; fi
```

#### Check for command's result <a id="check-for-command-s-result"></a>

```text
if ping -c 1 google.com; then
    echo "It appears you have a working internet connection"
fi
```

#### Special variables <a id="special-variables"></a>

| `$?` | Exit status of last task |
| :--- | :--- |
| `$!` | PID of last background task |
| `$$` | PID of shell |
| `$0` | Filename of the shell script |

See [Special parameters](http://wiki.bash-hackers.org/syntax/shellvars#special_parameters_and_shell_variables).

#### Grep check <a id="grep-check"></a>

```text
if grep -q 'foo' ~/.bash_history; then
    echo "You appear to have typed 'foo' in the past"
fi
```

#### Backslash escapes <a id="backslash-escapes"></a>

* * !
* "
* \#
* &
* '
* \(
* \)
* ,
* ;
* &lt;
* &gt;
* \[
* \|
* \
* \]
* ^
* {
* }
* \`
* $
* \*
* ?

Escape these special characters with `\`

#### Heredoc <a id="heredoc"></a>

```text
cat <<END
hello world
END
```

#### Go to previous directory <a id="go-to-previous-directory"></a>

```text
pwd # /home/user/foo
cd bar/
pwd # /home/user/foo/bar
cd -
pwd # /home/user/foo
```

#### Reading input <a id="reading-input"></a>

```text
echo -n "Proceed? [y/n]: "
read ans
echo $ans
```

```text
read -n 1 ans    # Just one character
```

#### Conditional execution <a id="conditional-execution"></a>

```text
git commit && git push
git commit || echo "Commit failed"
```

#### Strict mode <a id="strict-mode"></a>

```text
set -euo pipefail
IFS=$'\n\t'
```

See: [Unofficial bash strict mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/)

