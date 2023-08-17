# Bash is WEIRD

## Error Propegation

### Unset variables

#### Unset variable at the top level

```bash
$ cat test.sh

#!/usr/bin/env zsh

set -xeuo pipefail

echo "foo: $foo"
echo "foo RC: $?"

$ bash ./test.sh; echo "test rc: $?"
./test.sh: line 5: foo: unbound variable
test rc: 1
```


#### Unset variable in a function

```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

function bar() {
  echo "foo: $foo"
  echo "oh well"
}

bar
echo "foo RC: $?"

$ bash ./test.sh; echo "test rc: $?"
+ bar
./test.sh: line 6: foo: unbound variable
test rc: 1
```


#### Checking an unset variable in an if-statement

```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

if [[ -z "${foo}" ]]; then
  echo "foo is empty"
else
  echo "foo is not empty"
fi

$ bash ./test.sh; echo "test rc: $?"

./test.sh: line 5: foo: unbound variable
test rc: 1
```


#### WORKING: Checking a potentially unset variable in an if-statement

```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

if [[ "${foo:-UNSET}" == "UNSET" ]]; then
  echo "foo is unset"
else
  echo "foo is not unset"
fi

$ bash ./test.sh; echo "test rc: $?"
+ [[ UNSET == \U\N\S\E\T ]]
+ echo 'foo is unset'
foo is unset
test rc: 0
```


### Non existent functions/commands

#### Calling a non existent function/command at top-level

This will not execute past the point where the function/command does not exist

```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

non-existent-function
echo "function RC: $?"

$ bash ./test.sh; echo "test rc: $?"

+ non-existent-function
./test.sh: line 5: non-existent-function: command not found
test rt: 127
```

#### Calling a non existent function/command from a function


```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

function bar() {
  non-existent-function
  echo "function RC: $?"
}

bar
echo "bar RC: $?"

$ bash ./test.sh; echo "test rt: $?"
+ bar
+ non-existent-function
./test.sh: line 6: non-existent-function: command not found
test rt: 127
```

#### Calling a non existent function/command from a function, inside an if condition







################


```bash
$ cat test.sh

#!/usr/bin/env bash

set -xeuo pipefail

function foo() {
  echo "foo 1"
  non-existent-command
  echo $? # should be 127
  exit 127
  echo $? # should be 127
  echo "foo 2"
}

if (foo); then
  echo "foo succeeded"
else
  echo "foo failed"
fi

$ bash ./test-error-prop.sh

+ foo
+ echo 'foo 1'
foo 1
+ non-existent-command
./src/scripts/test-thing.sh: line 7: non-existent-command: command not found
+ echo 127
127
+ exit 127
+ echo 'foo failed'
foo failed
```