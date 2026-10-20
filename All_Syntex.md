# Bash Variables and Scripting

## 1. Variables

Variables are used to store data. We can use this data anywhere in a Bash script.

### Declaring Variables

To create a variable, write its name and assign a value using `=`.

**Syntax:**

```bash
variable_name=value
```

**Rules:**
- Do not add spaces around `=`.
- Use `$` to get the value of a variable.
- Use quotes when storing text with spaces.

**Example:**

```bash
#!/bin/bash

name="Maniraj Pandit"
echo "Hello, $name!"

number=42
echo "The number is $number"
```

**Output:**

```text
Hello, Maniraj Pandit!
The number is 42
```

### Variable Operations

Variables can store text and numbers. We can also join text and perform calculations.

```bash
#!/bin/bash

# Join two strings
greeting="Hello, "
name="World"
echo "${greeting}${name}"

# Add two numbers
num1=5
num2=10

sum=$((num1 + num2))
echo "The sum is $sum"
```

**Output:**

```text
Hello, World
The sum is 15
```

**Why do we use `$(( ))`?**

- `$(( ))` is used to perform arithmetic calculations in Bash.
- `$` gets the result of the calculation.
- The double parentheses tell Bash to calculate the expression.

Example:

```bash
result=$((5 + 3))
echo "$result"
```

Output:

```text
8
```

---

## 2. Data Types in Bash

Bash does not use data types in the same way as languages like Python or C++. Most variables are treated as strings, but Bash can perform integer calculations.

### Strings

Strings are used to store text.

```bash
#!/bin/bash

greeting="Hello"
name="Alice"

full_greeting="$greeting, $name!"
echo "$full_greeting"
```

**Output:**

```text
Hello, Alice!
```

### Numbers

Bash supports integer calculations using arithmetic expansion.

```bash
#!/bin/bash

num1=5
num2=10

sum=$((num1 + num2))
difference=$((num2 - num1))
product=$((num1 * num2))
quotient=$((num2 / num1))
remainder=$((num2 % num1))

echo "Sum: $sum"
echo "Difference: $difference"
echo "Product: $product"
echo "Quotient: $quotient"
echo "Remainder: $remainder"
```

**Output:**

```text
Sum: 15
Difference: 5
Product: 50
Quotient: 2
Remainder: 0
```

**Note:** Bash arithmetic normally uses integers. Division between integers does not give a decimal result.

### Indexed Arrays

Arrays store multiple values in one variable.

```bash
#!/bin/bash

fruits=("apple" "banana" "cherry")

# Access the first element
echo "${fruits[0]}"

# Access all elements
echo "${fruits[@]}"

# Loop through the array
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

**Output:**

```text
apple
apple banana cherry
apple
banana
cherry
```

**Important:**
- Array indexes start from `0`.
- `${fruits[0]}` accesses the first element.
- `${fruits[@]}` accesses all elements.

### Associative Arrays

Associative arrays store values using named keys instead of number indexes. They are similar to dictionaries in Python.

```bash
#!/bin/bash

declare -A colors

colors[apple]="red"
colors[banana]="yellow"
colors[grape]="purple"

# Remove one key and its value
unset 'colors[banana]'

echo "${colors[apple]}"
echo "${colors[grape]}"
```

**Output:**

```text
red
purple
```

**Note:** `declare -A` creates an associative array. Associative arrays require Bash 4 or newer.

---

## 3. User Input

The `read` command takes input from the user.

### Basic Input

```bash
#!/bin/bash

echo "Enter your name:"
read username

echo "Hello, $username!"
```

### Input with a Message

Use `-p` to show a message before taking input.

```bash
#!/bin/bash

read -p "Enter your name: " username

echo "Hello, $username!"
```

**Example output:**

```text
Enter your name: Maniraj
Hello, Maniraj!
```

---

## 4. Bash Operators

Operators are symbols or options used to compare values, perform calculations, and check conditions.

### Comparison Operators for Numbers

These operators are used inside Bash test expressions such as `[ ]`.

| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |

**Example:**

```bash
#!/bin/bash

num1=10
num2=20

if [ "$num1" -lt "$num2" ]; then
    echo "num1 is smaller"
fi
```

**Output:**

```text
num1 is smaller
```

### String Comparison Operators

These operators compare text.

| Operator | Meaning |
|---|---|
| `=` | Strings are equal |
| `!=` | Strings are not equal |
| `<` | Comes before in alphabetical order |
| `>` | Comes after in alphabetical order |
| `-z` | String is empty |
| `-n` | String is not empty |

**Example:**

```bash
#!/bin/bash

name="Maniraj"

if [ "$name" = "Maniraj" ]; then
    echo "Name matches"
fi

if [ -n "$name" ]; then
    echo "Name is not empty"
fi
```

**Output:**

```text
Name matches
Name is not empty
```

**Note:** When using `<` or `>` inside `[ ]`, quote and escape them to avoid shell redirection. For example, use `[[ "$a" < "$b" ]]` for a string comparison.

### Difference Between Number and String Comparisons

Number comparisons check the numeric value. String comparisons check the text.

```bash
#!/bin/bash

# Number comparison
if [ 10 -gt 5 ]; then
    echo "10 is greater than 5"
fi

# String comparison
if [ "apple" != "banana" ]; then
    echo "The strings are different"
fi
```

**Output:**

```text
10 is greater than 5
The strings are different
```

**Remember:**
- Use `-eq`, `-gt`, and `-lt` to compare numbers.
- Use `=` and `!=` to compare strings.

### Arithmetic Operators

These operators perform calculations inside `$(( ))`.

| Operator | Meaning |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Integer division |
| `%` | Remainder |

For exponentiation, use an external tool such as `bc` or `awk`.

### Logical Operators

Logical operators combine or change conditions.

| Operator | Meaning |
|---|---|
| `&&` | AND: both conditions must be true |
| `||` | OR: at least one condition must be true |
| `!` | NOT: reverses a condition |

**Example:**

```bash
#!/bin/bash

age=20

if [ "$age" -ge 18 ] && [ "$age" -le 60 ]; then
    echo "Age is between 18 and 60"
fi
```

**Output:**

```text
Age is between 18 and 60
```

### File Test Operators

These operators check files and directories.

| Operator | Meaning |
|---|---|
| `-e` | Path exists |
| `-d` | Path is a directory |
| `-f` | Path is a regular file |
| `-s` | File exists and is not empty |
| `-r` | File can be read |
| `-w` | File can be written to |
| `-x` | File can be executed |

**Example:**

```bash
#!/bin/bash

file="app.log"

if [ -f "$file" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

---

## 5. If-Else Statements

Conditional statements run different commands based on a condition.

### Basic If Statement

```bash
#!/bin/bash

num=15

if [ "$num" -gt 10 ]; then
    echo "Number is greater than 10"
fi
```

### If-Else Statement

```bash
#!/bin/bash

num=8

if [ "$num" -gt 10 ]; then
    echo "Number is greater than 10"
else
    echo "Number is 10 or less"
fi
```

### If-Elif-Else Statement

Use `elif` to check another condition.

```bash
#!/bin/bash

num=10

if [ "$num" -gt 10 ]; then
    echo "Number is greater than 10"
elif [ "$num" -eq 10 ]; then
    echo "Number is exactly 10"
else
    echo "Number is less than 10"
fi
```

### Nested If Statement

A nested `if` means putting one `if` statement inside another.

```bash
#!/bin/bash

num=5

if [ "$num" -gt 0 ]; then
    if [ "$num" -lt 10 ]; then
        echo "Number is between 1 and 9"
    fi
fi
```

---

## 6. Loops

Loops run a group of commands multiple times.

### For Loop

A `for` loop is useful when working with a list of values.

```bash
#!/bin/bash

for i in {1..5}; do
    echo "Iteration $i"
done
```

**Output:**

```text
Iteration 1
Iteration 2
Iteration 3
Iteration 4
Iteration 5
```

### While Loop

A `while` loop runs as long as its condition is true.

```bash
#!/bin/bash

count=1

while [ "$count" -le 5 ]; do
    echo "Count is $count"
    ((count++))
done
```

### Until Loop

An `until` loop runs until its condition becomes true.

```bash
#!/bin/bash

count=1

until [ "$count" -gt 5 ]; do
    echo "Count is $count"
    ((count++))
done
```

### Break and Continue

- `break` stops the loop.
- `continue` skips the current iteration and moves to the next one.

```bash
#!/bin/bash

for i in {1..5}; do
    if [ "$i" -eq 3 ]; then
        continue
    fi

    echo "Number $i"

    if [ "$i" -eq 4 ]; then
        break
    fi
done
```

**Output:**

```text
Number 1
Number 2
Number 4
```

### Nested Loops

A nested loop is a loop inside another loop.

```bash
#!/bin/bash

for i in {1..3}; do
    for j in {1..2}; do
        echo "Outer loop $i, Inner loop $j"
    done
done
```

---

## 7. Command-Line Arguments

Command-line arguments pass values to a Bash script when it runs.

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` | Third argument |
| `$#` | Number of arguments |
| `$@` | All arguments as separate values |
| `$*` | All arguments, usually treated as one string when quoted |

**Example:**

Create a file named `main.sh`.

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Number of arguments: $#"
```

Run the script:

```bash
bash main.sh Maniraj 20
```

**Output:**

```text
Script name: main.sh
First argument: Maniraj
Second argument: 20
Number of arguments: 2
```

**Important:** Command-line arguments are generally treated as text. Use arithmetic syntax or another suitable method when you need to perform calculations with numeric arguments.

---

## 8. Functions

Functions group commands so they can be used again without writing the same code repeatedly.

### Creating a Function

```bash
#!/bin/bash

my_function() {
    echo "Hello, World!"
}
```

### Calling a Function

```bash
my_function
```

**Output:**

```text
Hello, World!
```

### Passing Arguments to a Function

Arguments passed to a function are accessed using `$1`, `$2`, and so on.

Use `local` to create a variable that belongs to the function.

```bash
#!/bin/bash

greet() {
    local name="$1"
    echo "Hello, $name!"
}

greet "Alice"
```

**Output:**

```text
Hello, Alice!
```

### Returning a Result

Bash functions can print a result using `echo`. Command substitution `$()` can capture that output.

```bash
#!/bin/bash

add() {
    local sum=$(($1 + $2))
    echo "$sum"
}

result=$(add 5 3)

echo "The sum is $result"
```

**Output:**

```text
The sum is 8
```

**Note:** Bash's `return` keyword is used to return an exit status, usually a number from `0` to `255`. Use printed output when you want to pass a calculation result back to a variable.

---

## 9. Working with Arrays

Arrays store multiple values in one variable.

```bash
#!/bin/bash

# Create an array
my_array=("value1" "value2" "value3")

# Access an element
echo "${my_array[0]}"

# Change an element
my_array[1]="new_value"

# Access all elements
echo "${my_array[@]}"

# Get the number of elements
echo "${#my_array[@]}"
```

**Output:**

```text
value1
value1 new_value value3
3
```

**Remember:** Array indexes start at `0`.

---

## 10. Scheduling Tasks with Cron

Cron is a Linux tool that runs commands or scripts automatically at scheduled times.

For example, cron can be used to:
- Run a backup script every day.
- Clean old log files.
- Check a service at regular times.
- Run a scheduled maintenance script.

### Cron and Crontab

- **cron:** The background service that runs scheduled tasks.
- **crontab:** The file or command used to manage scheduled tasks for a user.

### Open the Crontab

```bash
crontab -e
```

This opens the current user's cron schedule for editing.

### Cron Syntax

```text
minute hour day_of_month month day_of_week command
```

| Field | Allowed values |
|---|---|
| Minute | `0-59` |
| Hour | `0-23` |
| Day of month | `1-31` |
| Month | `1-12` |
| Day of week | `0-7` (Sunday can be `0` or `7`) |

### Cron Examples

Run a script every day at 2:30 AM:

```cron
30 2 * * * /home/user/backup.sh
```

Run a script every five minutes:

```cron
*/5 * * * * /home/user/check.sh
```

Run a script every Monday at 9:00 AM:

```cron
0 9 * * 1 /home/user/weekly.sh
```

**Important:**
- Use the full path to your script.
- Make sure the script has the required permissions.
- Cron has a limited environment, so use full paths for required commands when needed.

### Useful Crontab Commands

```bash
# Edit scheduled tasks
crontab -e

# List scheduled tasks
crontab -l

# Remove all scheduled tasks for the current user
crontab -r
```

**Warning:** `crontab -r` removes all scheduled tasks for the current user. Use it carefully.

---

## 11. Useful Bash Tips

### Add a Shebang

Start a Bash script with:

```bash
#!/bin/bash
```

This tells the system to use Bash to run the script.

### Make a Script Executable

```bash
chmod +x main.sh
```

Run it using:

```bash
./main.sh
```

### Quote Variables

Prefer quoting variables when using their values.

```bash
name="Maniraj Pandit"
echo "$name"
```

Quotes help prevent problems with spaces and special characters.

### Use Comments

Comments explain what the code does. Bash ignores comments that start with `#`.

```bash
# Store the username
username="Maniraj"
```

---

<p align="center">
  <a href="../Readme.md">
    <img src="https://img.shields.io/badge/⬅_Back-blue?style=for-the-badge" alt="Back">
  </a>
</p>

