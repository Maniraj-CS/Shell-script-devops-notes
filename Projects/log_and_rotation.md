# Backup Script with 5-Backup Rotation

This Bash script creates a ZIP backup of a source path and automatically keeps only the **5 most recent backups**. 

## `backup.sh`

```bash
#!/bin/bash

: <<'COMMENT'
This script creates a backup and keeps the 5 most recent backups.
Usage: ./backup.sh <source_path> <backup_directory>
COMMENT

# Display script usage
display_usage() {
    echo "Usage: ./backup.sh <source_path> <backup_directory>"
    exit 1
}

# Check arguments
if [[ $# -ne 2 ]]; then
    display_usage
fi

source_dir="$1"
backup_dir="$2"
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')

# Check source path
if [[ ! -e "$source_dir" ]]; then
    echo "Error: Source path does not exist."
    exit 1
fi

# Check whether zip is installed
if ! command -v zip >/dev/null 2>&1; then
    echo "Error: zip is not installed."
    exit 1
fi

# Create backup directory if it does not exist
if ! mkdir -p -- "$backup_dir"; then
    echo "Error: Could not create backup directory."
    exit 1
fi

# Create a backup
create_backup() {
    local backup_file="${backup_dir%/}/backup_${timestamp}.zip"

    if zip -r "$backup_file" "$source_dir" >/dev/null; then
        echo "Backup created successfully: $backup_file"
        return 0
    else
        echo "Error: Backup creation failed."
        rm -f -- "$backup_file"
        return 1
    fi
}

# Keep only the 5 most recent backups
perform_backup_rotation() {
    local backups
    local total
    local remove_count
    local i

    # Collect backup filenames in alphabetical order.
    # The timestamp format makes this chronological order too.
    shopt -s nullglob
    backups=("$backup_dir"/backup_*.zip)
    shopt -u nullglob

    total=${#backups[@]}

    if (( total > 5 )); then
        remove_count=$((total - 5))

        echo "Performing backup rotation..."

        for ((i = 0; i < remove_count; i++)); do
            if rm -f -- "${backups[i]}"; then
                echo "Removed old backup: ${backups[i]}"
            else
                echo "Error: Could not remove ${backups[i]}"
                return 1
            fi
        done
    else
        echo "No rotation required. Total backups: $total"
    fi
}

# Run the backup first.
# Rotate old backups only if backup creation succeeds.
if create_backup; then
    perform_backup_rotation
else
    exit 1
fi
```

## How the Important Parts Work

### 1. Check the number of arguments

```bash
if [[ $# -ne 2 ]]; then
    display_usage
fi
```

- `$#` gives the number of arguments passed to the script.
- `-ne` means “not equal.”
- The script requires exactly two arguments: source path and backup directory.

Example:

```bash
./backup.sh ./my-project ./backups
```

### 2. Create a timestamp

```bash
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')
```

This creates a timestamp such as:

```text
2026-10-18-17-45-30
```

The timestamp makes backup filenames unique in normal use and sorts chronologically when written in this format.

### 3. Suppress command output

```bash
zip -r "$backup_file" "$source_dir" >/dev/null
```

`>/dev/null` redirects **standard output (stdout)** to `/dev/null`, so the ZIP file listing is hidden. Error output is still visible.

To hide both standard output and standard error, you could use:

```bash
command >/dev/null 2>&1
```

For this script, keeping error messages visible is more useful.

### 4. Check whether a command succeeded

```bash
if zip -r "$backup_file" "$source_dir" >/dev/null; then
    echo "Backup created successfully"
else
    echo "Backup creation failed"
fi
```

This checks the command's exit status directly. Exit status `0` normally means success; a nonzero status indicates failure.

This is preferable to checking `$?` separately because the `if` statement evaluates the command itself.

### 5. Collect backup files into an array

```bash
shopt -s nullglob
backups=("$backup_dir"/backup_*.zip)
shopt -u nullglob
```

- `backup_*.zip` matches backup files with the expected naming pattern.
- `nullglob` makes an unmatched pattern expand to an empty array instead of remaining a literal filename.
- Quoting the directory portion helps preserve spaces in directory names.

### 6. Count the backups

```bash
total=${#backups[@]}
```

`${#backups[@]}` gives the number of elements in the array.

For example, if there are 7 backups:

```text
total = 7
```

### 7. Calculate how many backups to delete

```bash
remove_count=$((total - 5))
```

If there are 7 backups:

```text
remove_count = 7 - 5
remove_count = 2
```

The script removes the two oldest backups, leaving the five newest ones.

### 8. Remove old backups safely

```bash
rm -f -- "${backups[i]}"
```

- `rm` removes the file.
- `-f` avoids prompting and ignores missing files.
- `--` marks the end of command options.
- Quoting `"${backups[i]}"` protects paths containing spaces.

The array is sorted by filename, and the timestamp naming pattern makes the oldest backup appear first.

---

## Run the Script

```bash
chmod +x backup.sh

./backup.sh ./my-project ./backups
```

Example output:

```text
Backup created successfully: ./backups/backup_2026-10-18-17-45-30.zip
Performing backup rotation...
Removed old backup: ./backups/backup_2026-10-10-09-00-00.zip
```

If there are fewer than six backups, no files are removed.

**Important:** This script keeps the five most recent backup files matching `backup_*.zip` in the specified directory. Test it with sample files before using it on important data.


---

<p align="center">
  <a href="./makefile.md">
    <img src="https://img.shields.io/badge/⬅_Back-blue?style=for-the-badge" alt="Back">
  </a>
  <a href="../Readme.md">
    <img src="https://img.shields.io/badge/Forward_➡-green?style=for-the-badge" alt="Back To ReadMe">
  </a>
</p>