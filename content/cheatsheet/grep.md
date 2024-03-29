# Grep



### [\#](https://quickref.me/grep#getting-started)Getting started <a id="getting-started"></a>

#### Usage <a id="usage"></a>

Search standard output \(i.e. a stream of text\)

```text
$ grep [options] search_string
```

Search for an exact string in file:

```text
$ grep [options] search_string path/to/file
```

Print lines in myfile.txt containing the string "mellon"

```text
$ grep 'mellon' myfile.txt
```

Wildcards are accepted in filename.

#### Option examples <a id="option-examples"></a>

| `-i` | grep -i ^DA demo.txt | Forgets about case sensitivity |
| :--- | :--- | :--- |
| `-w` | grep -w "of" demo.txt | Search only for the full word |
| `-A` | grep -A 3 'Exception' error.log | Display 3 lines after matching string |
| `-B` | grep -B 4 'Exception' error.log | Display 4 lines before matching string |
| `-C` | grep -C 5 'Exception' error.log | Display 5 lines around matching string |
| `-r` | grep -r 'quickref.me' /var/log/nginx/ | Recursive search _\(within subdirs\)_ |
| `-v` | grep -v 'warning' /var/log/syslog | Return all lines which don't match the pattern |
| `-e` | grep -e '^al' filename | Use regex _\(lines starting with 'al'\)_ |
| `-E` | grep -E 'ja\(s\|cks\)on' filename | Extended regex _\(lines containing jason or jackson\)_ |
| `-c` | grep -c 'error' /var/log/syslog | Count the number of matches |
| `-l` | grep -l 'robot' /var/log/\* | Print the name of the file\(s\) of matches |
| `-o` | grep -o search\_string filename | Only show the matching part of the string |
| `-n` | grep -n "go" demo.txt | Show the line numbers of the matches |

### [\#](https://quickref.me/grep#basic-regular-expressions)Basic regular expressions <a id="basic-regular-expressions"></a>

#### Refer <a id="refer"></a>

* [Regex syntax](https://quickref.me/regex) _\(quickref.me\)_
* [Regex examples](https://quickref.me/regex#regex-examples) _\(quickref.me\)_

Please refer to the full version of the regex cheat sheet for more complex requirements.

#### Wildcards <a id="wildcards"></a>

| . | Any character. |
| :--- | :--- |
| `?` | Optional and can only occur once. |
| `*` | Optional and can occur more than once. |
| `+` | Required and can occur more than once. |

#### Quantifiers <a id="quantifiers"></a>

| `{n}` | Previous item appears exactly n times. |
| :--- | :--- |
| `{n,}` | Previous item appears n times or more. |
| `{,m}` | Previous item appears n times maximum. |
| `{n,m}` | Previous item appears between n and m times. |

#### POSIX <a id="posix"></a>

| `[:alpha:]` | Any lower and upper case letter. |
| :--- | :--- |
| `[:digit:]` | Any number. |
| `[:alnum:]` | Any lower and upper case letter or digit. |
| `[:space:]` | Any whites­pace. |

#### Character <a id="character"></a>

| `[A-Z­a-z]` | Any lower and upper case letter. |
| :--- | :--- |
| `[0-9]` | Any number. |
| `[0-9­A-Z­a-z]` | Any lower and upper case letter or digit. |

#### Position <a id="position"></a>

| `^` | Beginning of line. |
| :--- | :--- |
| `$` | End of line. |
| `^$` | Empty line. |
| `\<` | Start of word. |
| `\>` | End of word. |

