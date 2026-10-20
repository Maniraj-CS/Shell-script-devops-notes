# Bash Project: Backup Script with Rotation

## 1. What Does This Script Do?

This script takes two inputs:

1. The path of the folder or file we want to back up.
2. The directory where we want to save backups.

It creates a ZIP backup with the current date and time in its name.

It also keeps only the **5 newest backup files** and deletes older backups.

For example:

```text
backups/
├── backup_2026-10-14-10-00-00.zip
├── backup_2026-10-15-10-00-00.zip
├── backup_2026-10-16-10-00-00.zip
├── backup_2026-10-17-10-00-00.zip
└── backup_2026-10-18-10-00-00.zip
```

If a new backup is created and there are now 6 backups, the oldest one will be deleted.

**Note:** This script keeps the 5 newest backup files, not necessarily backups from 5 different calendar days.

---

## 2. Complete Script (`backup.sh`)

```bash
#!/bin/bash

# This is a backup script with 5-backup rotation.

# This is a multi-line comment.
# Bash does not have a special multi-line comment syntax.
# Here, a here-document is used to display a block of text.
: <<'COMMENT'
This script creates a ZIP backup.
It keeps only the 5 newest backup files.
COMMENT


# This function displays how to run the script.
display_usage() {
    echo "Usage: ./backup.sh <source_path> <backup_directory>"
    exit 1
}


# Check the number of arguments.
# $# means the number of arguments passed to the script.
# We need exactly 2 arguments.
if [[ $# -ne 2 ]]; then
    display_usage
fi


# Store the first argument in source_dir.
# $1 means the first argument.
source_dir="$1"

# Store the second argument in backup_dir.
# $2 means the second argument.
backup_dir="$2"

# Get the current date and time.
# $(command) runs a command and stores its output.
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')


# Check whether the source file or folder exists.
# -e means the path exists.
# ! means NOT.
if [[ ! -e "$source_dir" ]]; then
    echo "Error: Source path does not exist."
    exit 1
fi


# Check whether the zip command is installed.
# command -v checks whether a command is available.
# >/dev/null hides normal output.
# 2>&1 sends error output to the same place as normal output.
if ! command -v zip >/dev/null 2>&1; then
    echo "Error: Please install the zip command first."
    exit 1
fi


# Create the backup directory if it does not exist.
# -p creates parent directories when needed.
if ! mkdir -p -- "$backup_dir"; then
    echo "Error: Could not create backup directory."
    exit 1
fi


# This function creates the ZIP backup.
create_backup() {

    # Build the backup filename.
    # ${backup_dir%/} removes a trailing slash from the directory path.
    # This avoids adding an extra slash before the filename.
    backup_file="${backup_dir%/}/backup_${timestamp}.zip"

    # -r means include files inside folders recursively.
    # > /dev/null hides the normal ZIP output.
    # The if statement checks whether ZIP completed successfully.
    if zip -r "$backup_file" "$source_dir" >/dev/null; then

        echo "Backup created successfully: $backup_file"

        # return 0 means this function succeeded.
        return 0

    else

        echo "Error: Backup creation failed."

        # Remove an incomplete backup file, if one was created.
        rm -f -- "$backup_file"

        # return 1 means this function failed.
        return 1
    fi
}


# This function removes old backups.
perform_backup_rotation() {

    # Enable nullglob temporarily.
    # If no file matches the pattern, Bash returns an empty array.
    shopt -s nullglob

    # Find backup files and store their paths in an array.
    # The timestamp in the filenames makes alphabetical order
    # match chronological order.
    backups=("$backup_dir"/backup_*.zip)

    # Disable nullglob after collecting the files.
    shopt -u nullglob

    # Count the number of backup files.
    # ${#backups[@]} means the number of array elements.
    total=${#backups[@]}

    # If there are more than 5 backups, remove the oldest ones.
    if [[ "$total" -gt 5 ]]; then

        echo "Performing backup rotation..."

        # Calculate how many backups must be deleted.
        remove_count=$((total - 5))

        # Start from the oldest backup at array index 0.
        # This loop runs remove_count times.
        for ((i = 0; i < remove_count; i++)); do

            # Remove the backup at the current array index.
            # -f means do not ask for confirmation.
            # -- marks the end of command options.
            # Quotes protect paths that contain spaces.
            if rm -f -- "${backups[i]}"; then
                echo "Removed old backup: ${backups[i]}"
            else
                echo "Error: Could not remove ${backups[i]}"
                return 1
            fi

        done

    else

        # If there are 5 or fewer backups, keep them all.
        echo "No rotation required. Total backups: $total"

    fi
}


# First, try to create a backup.
# Only rotate old backups if the new backup succeeds.
if create_backup; then
    perform_backup_rotation
else
    # Stop the script if backup creation fails.
    exit 1
fi
```

---

## 3. How to Run This Script

### Step 1: Save the file

Save the code in a file named `backup.sh`.

### Step 2: Give execute permission

```bash
chmod +x backup.sh
```

`chmod` changes file permissions, and `+x` adds execute permission.

### Step 3: Run the script

```bash
./backup.sh ./my-project ./backups
```

Here:

- `./backup.sh` is the script.
- `./my-project` is the source folder.
- `./backups` is the destination directory.

Make sure the `zip` command is installed before running the script.

---

## 4. Important Bash Syntax

### A. `$#` — Number of arguments

```bash
echo "$#"
```

`$#` tells us how many arguments were passed to the script.

For example:

```bash
./backup.sh ./project ./backups
```

There are two arguments, so `$#` is `2`.

### B. `$1` and `$2` — Arguments

```bash
source_dir="$1"
backup_dir="$2"
```

- `$1` contains the first argument.
- `$2` contains the second argument.

For the command above, `source_dir` is `./project`, and `backup_dir` is `./backups`.

### C. `$?` — Previous command's exit status

```bash
mkdir -p ./backups

echo "$?"
```

`$?` gives the exit status of the last command.

- `0` usually means success.
- A nonzero value means the command failed.

Important: save or check `$?` immediately after the command you want to check. Running another command changes its value.

You can also check a command directly using `if`, as this script does:

```bash
if mkdir -p ./backups; then
    echo "Directory created successfully"
else
    echo "Directory creation failed"
fi
```

### D. `$(command)` — Run a command and use its output

```bash
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')
```

`$(...)` runs the command inside the parentheses and replaces it with the command's output.

Here, the current date and time are stored in the `timestamp` variable.

### E. `>/dev/null` — Hide normal output

```bash
zip -r backup.zip ./project >/dev/null
```

- `>` redirects output.
- `/dev/null` is a special location that discards output.

This hides the normal output from the ZIP command. Error messages can still appear.

### F. `2>/dev/null` — Hide error output

```bash
some_command 2>/dev/null
```

In Bash:

- `1` means standard output (stdout).
- `2` means standard error (stderr).

So `2>/dev/null` hides error messages.

### G. `>/dev/null 2>&1` — Hide both outputs

```bash
some_command >/dev/null 2>&1
```

This redirects normal output to `/dev/null` and then sends error output to the same destination.

It hides both normal output and error output.

Use it when you want a command to run silently. During debugging, showing errors is usually more helpful.

### H. `${array[@]}` — All array elements

```bash
backups=("backup1.zip" "backup2.zip" "backup3.zip")

echo "${backups[@]}"
```

`${backups[@]}` accesses all elements of the array.

Use quotes around `"${backups[@]}"` when passing array elements as separate arguments, especially when filenames may contain spaces.

### I. `${#array[@]}` — Number of array elements

```bash
echo "${#backups[@]}"
```

This returns the number of elements in the array.

For example, if the array contains three backup filenames, the output is `3`.

### J. `${array[index]}` — One array element

```bash
echo "${backups[0]}"
```

This prints the first array element.

**Important:** Bash array indexes start at `0`, not `1`.

### K. `$((calculation))` — Arithmetic

```bash
total=8
remove_count=$((total - 5))

echo "$remove_count"
```

The output is `3`.

`$((...))` tells Bash to calculate the arithmetic expression inside the parentheses.

### L. `if`, `then`, `else`, and `fi`

```bash
if [[ "$total" -gt 5 ]]; then
    echo "More than 5 backups"
else
    echo "5 or fewer backups"
fi
```

- `if` starts the condition.
- `then` starts the commands to run when the condition is true.
- `else` handles the other case.
- `fi` ends the `if` statement.
- `-gt` means greater than.

Keep spaces around the condition inside `[[ ... ]]`.

### M. `for` loop

```bash
for ((i = 0; i < 3; i++)); do
    echo "$i"
done
```

This prints `0`, `1`, and `2`.

- `i = 0` sets the starting value.
- `i < 3` is the condition for continuing.
- `i++` increases `i` by one.
- `do` starts the loop body.
- `done` ends the loop.

### N. `rm -f -- "$file"`

```bash
rm -f -- "$file"
```

- `rm` removes files.
- `-f` forces removal without prompting.
- `--` marks the end of command options.
- `"$file"` is the path of the file to remove.

Be careful: `rm` can permanently delete files. Always verify the path before using it.

### O. Functions

```bash
create_backup() {
    echo "Creating backup..."
}

create_backup
```

A function groups commands under one name so you can reuse them.

- `create_backup() { ... }` defines the function.
- `create_backup` runs the function.
- `return 0` signals success.
- `return 1` signals failure.

---

## 5. How Backup Rotation Works

Suppose the backup directory contains seven files:

```text
backup_01.zip
backup_02.zip
backup_03.zip
backup_04.zip
backup_05.zip
backup_06.zip
backup_07.zip
```

The script calculates:

```bash
remove_count=$((7 - 5))
```

The result is `2`, so it removes the first two files in the sorted array and keeps the five newest ones.

The timestamp format is important because filenames must sort in chronological order for this method to work correctly.

---

## 6. Important Notes

- The script must have permission to read the source and write to the backup directory.
- The destination should ideally be outside the source directory to avoid backing up its own backups.
- The backup directory should be used for files following the `backup_*.zip` naming pattern.
- This script uses ZIP archives; it does not encrypt them.
- Keeping five backup files does not guarantee that the backups are stored on a separate device. For important data, keep an additional copy in another location.

---

<p align="center">
  <a href="../Readme.md">
    <img src="https://img.shields.io/badge/⬅_Back-blue?style=for-the-badge" alt="Back">
  </a>
</p>