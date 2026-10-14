# Bash Project: Connect AWS CLI and Create an EC2 Instance

## 1. Project Overview

In this project, we use a Bash script to connect to AWS and create an EC2 instance automatically.

Normally, we install AWS CLI and configure our AWS credentials before using AWS commands. In this project, the script can install AWS CLI if it is missing, ask for AWS credentials, check the connection, and create an EC2 instance.

### What will we learn?

- How to check whether AWS CLI is installed.
- How to install AWS CLI using Bash.
- How to handle AWS credentials safely.
- How to use AWS CLI commands inside a Bash script.
- How to create an EC2 instance.
- How to use functions and command-line arguments.
- How to wait until an EC2 instance reaches the running state.
- How to handle errors in a script.

### Project Flow

```text
Start Script
    |
    v
Check AWS CLI
    |
    +---- Not Installed ----> Install AWS CLI
    |                              |
    +------------------------------+
                   |
                   v
          Get AWS Credentials
                   |
                   v
          Check AWS Connection
                   |
                   v
          Create EC2 Instance
                   |
                   v
          Wait for Instance
                   |
                   v
                 Done
```

---

## 2. Complete Bash Script

Create a file named `create-ec2.sh`.

**Before running it:** Update the AMI ID, key pair name, subnet ID, security group ID, and AWS region to match your AWS account. These values depend on your region and VPC setup.

```bash
#!/bin/bash

# Stop the script when an error occurs, an undefined variable is used,
# or a command in a pipeline fails.
set -euo pipefail


# -----------------------------------------
# 1. Check AWS CLI
# -----------------------------------------

check_awscli() {
    if ! command -v aws >/dev/null 2>&1; then
        echo "AWS CLI is not installed."
        return 1
    fi

    echo "AWS CLI is installed."
}


# -----------------------------------------
# 2. Install AWS CLI
# -----------------------------------------

install_awscli() {
    echo "Installing AWS CLI..."

    if ! command -v curl >/dev/null 2>&1; then
        echo "Please install curl first." >&2
        return 1
    fi

    if ! command -v unzip >/dev/null 2>&1; then
        sudo apt-get update
        sudo apt-get install -y unzip
    fi

    local temp_dir
    temp_dir=$(mktemp -d)

    # Remove temporary files when this function exits.
    trap 'rm -rf "$temp_dir"' RETURN

    curl -fsSL \
        "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
        -o "$temp_dir/awscliv2.zip"

    unzip -q "$temp_dir/awscliv2.zip" -d "$temp_dir"

    sudo "$temp_dir/aws/install"

    aws --version

    echo "AWS CLI installation completed."
}


# -----------------------------------------
# 3. Get AWS Credentials
# -----------------------------------------

get_aws_credentials() {
    echo "Enter your AWS credentials."

    # Ask for the Access Key ID.
    read -rp "AWS Access Key ID: " AWS_ACCESS_KEY_ID

    # Hide the Secret Access Key while typing.
    read -rsp "AWS Secret Access Key: " AWS_SECRET_ACCESS_KEY
    echo

    # Make credentials available to AWS CLI commands
    # started by this script.
    export AWS_ACCESS_KEY_ID
    export AWS_SECRET_ACCESS_KEY

    # Set the AWS region.
    export AWS_DEFAULT_REGION="ap-southeast-2"

    echo "Checking AWS connection..."

    if ! aws sts get-caller-identity >/dev/null; then
        echo "AWS authentication failed. Check your credentials." >&2
        return 1
    fi

    echo "AWS connection successful."
}


# -----------------------------------------
# 4. Wait for EC2 Instance
# -----------------------------------------

wait_for_instance() {
    local instance_id="$1"
    local state
    local attempts=0

    echo "Waiting for instance $instance_id..."

    while (( attempts < 60 )); do

        if ! state=$(aws ec2 describe-instances \
            --instance-ids "$instance_id" \
            --query 'Reservations[0].Instances[0].State.Name' \
            --output text); then

            echo "Could not check the instance state." >&2
            return 1
        fi

        if [[ "$state" == "running" ]]; then
            echo "Instance is now running."
            return 0
        fi

        if [[ "$state" == "terminated" || "$state" == "shutting-down" ]]; then
            echo "Instance stopped before reaching the running state." >&2
            return 1
        fi

        sleep 10
        ((attempts += 1))
    done

    echo "Timed out while waiting for the instance." >&2
    return 1
}


# -----------------------------------------
# 5. Create EC2 Instance
# -----------------------------------------

create_ec2_instance() {

    local ami_id="$1"
    local instance_type="$2"
    local key_name="$3"
    local subnet_id="$4"
    local security_group_id="$5"
    local instance_name="$6"

    local instance_id

    echo "Creating EC2 instance..."

    if ! instance_id=$(aws ec2 run-instances \
        --image-id "$ami_id" \
        --instance-type "$instance_type" \
        --count 1 \
        --key-name "$key_name" \
        --subnet-id "$subnet_id" \
        --security-group-ids "$security_group_id" \
        --tag-specifications \
        "ResourceType=instance,Tags=[{Key=Name,Value=$instance_name}]" \
        --query 'Instances[0].InstanceId' \
        --output text); then

        echo "Failed to create EC2 instance." >&2
        return 1
    fi

    if [[ -z "$instance_id" || "$instance_id" == "None" ]]; then
        echo "No instance ID was returned." >&2
        return 1
    fi

    echo "Instance created successfully."
    echo "Instance ID: $instance_id"

    wait_for_instance "$instance_id"
}


# -----------------------------------------
# 6. Main Function
# -----------------------------------------

main() {

    # Check AWS CLI and install it if needed.
    if ! check_awscli; then
        install_awscli
    fi

    # Get credentials and verify AWS access.
    get_aws_credentials

    echo "Starting EC2 creation..."

    # Replace these values with your own AWS resources.
    local AMI_ID="ami-xxxxxxxxxxxxxxxxx"
    local INSTANCE_TYPE="t2.micro"
    local KEY_NAME="your-key-pair"
    local SUBNET_ID="subnet-xxxxxxxxxxxxxxxxx"
    local SECURITY_GROUP_ID="sg-xxxxxxxxxxxxxxxxx"
    local INSTANCE_NAME="Shell-Script-EC2-Demo"

    create_ec2_instance \
        "$AMI_ID" \
        "$INSTANCE_TYPE" \
        "$KEY_NAME" \
        "$SUBNET_ID" \
        "$SECURITY_GROUP_ID" \
        "$INSTANCE_NAME"

    echo "EC2 creation process completed."
}


# Run the main function.
main
```

**Important:** This installation command is for 64-bit x86 Linux. Also, `t2.micro` is not available in every region or account configuration. Choose an instance type that is available to you.

---

## 3. Understanding the Important Parts

### A. What does `set -euo pipefail` mean?

This command enables three Bash safety options.

```bash
set -euo pipefail
```

| Option | Simple meaning |
|---|---|
| `-e` | Stop the script when an unhandled command fails. |
| `-u` | Treat the use of an undefined variable as an error. |
| `-o pipefail` | Make a pipeline fail if one of its commands fails. |

**Example:**

```bash
#!/bin/bash

set -e

echo "Starting script"

false

echo "This line will not run"
```

The `false` command returns an error status, so Bash stops before the last `echo`.

**Why use it?**

It helps prevent a script from continuing after an important command fails. However, it is not a replacement for proper error handling. Commands inside `if` statements can be handled differently, so important operations should still be checked.

### B. Why use `command -v aws`?

```bash
command -v aws
```

This checks whether the `aws` command is available.

```bash
if ! command -v aws >/dev/null 2>&1; then
    echo "AWS CLI is not installed."
fi
```

The `!` reverses the result. If the command is not found, the condition becomes true.

### C. Why use `>/dev/null 2>&1`?

```bash
command -v aws >/dev/null 2>&1
```

This redirects command output so it does not appear on the terminal.

- `>` redirects standard output.
- `/dev/null` discards the output.
- `2>&1` sends standard error to the same place as standard output.

In simple words, both normal output and error output are hidden.

**Difference from `&> /dev/null`:**

```bash
command -v aws &> /dev/null
```

This also hides both output streams in Bash. The form `>/dev/null 2>&1` is commonly used because it also works in many other shell environments.

### D. Why use `>&2`?

```bash
echo "Something went wrong." >&2
```

Normally, `echo` writes to standard output, also called `stdout`.

`>&2` sends the message to standard error, also called `stderr`.

This helps separate normal messages from error messages. It is useful when saving script output to a file or connecting commands with pipes.

### E. Why use `curl -fsSL`?

```bash
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
```

`curl` downloads the AWS CLI installation file.

| Option | Meaning |
|---|---|
| `-f` | Treat HTTP error responses as failures. |
| `-s` | Hide the progress display. |
| `-S` | Show an error if a silent request fails. |
| `-L` | Follow redirects. |
| `-o` | Save the downloaded file using the given name. |

### F. Why use `unzip`?

The AWS CLI download is a ZIP file.

```bash
unzip -q awscliv2.zip -d "$temp_dir"
```

- `-q` reduces the output.
- `-d` selects the destination directory.

The script uses `mktemp -d` to create a temporary directory, then removes it when the installation function finishes.

### G. Why use `read -rp` and `read -rsp`?

```bash
read -rp "AWS Access Key ID: " AWS_ACCESS_KEY_ID
read -rsp "AWS Secret Access Key: " AWS_SECRET_ACCESS_KEY
echo
```

- `read` accepts user input.
- `-p` displays a prompt.
- `-r` reads the input without treating backslashes specially.
- `-s` hides the input while the user types.

The secret key is hidden on the screen. It is not placed in the script itself or passed as a command-line argument.

**Security note:** Environment variables are safer than putting secret keys in command arguments or shell history, but they are not a complete secret-management solution. Prefer IAM roles or AWS profiles where possible. Never upload credentials to GitHub.

### H. Why use `export`?

```bash
export AWS_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY
export AWS_DEFAULT_REGION="ap-southeast-2"
```

`export` makes these variables available to commands started by the script.

The AWS CLI can read these environment variables automatically. Therefore, you do not need to run `aws configure` for this script's credentials.

### I. Why use `aws sts get-caller-identity`?

```bash
aws sts get-caller-identity
```

This command checks whether AWS can identify the current credentials.

It returns information such as the AWS account ID and the identity associated with the credentials.

It does not create an EC2 instance. It only verifies that the AWS CLI can make an authenticated request.

### J. Why use `local` inside a function?

```bash
local instance_id="$1"
```

`local` creates a variable that is available only inside the current function.

This helps prevent function variables from accidentally changing variables used elsewhere in the script.

### K. Why use `$1`, `$2`, and other numbered variables?

Bash functions can receive arguments.

```bash
create_ec2_instance \
    "$AMI_ID" \
    "$INSTANCE_TYPE" \
    "$KEY_NAME"
```

Inside the function:

- `$1` contains the AMI ID.
- `$2` contains the instance type.
- `$3` contains the key pair name.

The same idea applies to command-line arguments passed to a script.

### L. Why use `$(command)`?

```bash
instance_id=$(aws ec2 run-instances ...)
```

`$(...)` is called command substitution. It runs a command and stores its standard output in a variable.

In this project, AWS CLI returns an instance ID, and the script saves it in `instance_id`.

### M. Why use `--query` and `--output text`?

```bash
--query 'Instances[0].InstanceId' \
--output text
```

- `--query` selects the information you need from the AWS response.
- `Instances[0].InstanceId` selects the instance ID from the first returned instance.
- `--output text` returns the selected value as plain text instead of formatted JSON.

This makes it easier to store the result in a Bash variable.

### N. Why use `while` and `sleep`?

```bash
while (( attempts < 60 )); do
    # Check the instance state
    sleep 10
    ((attempts += 1))
done
```

The loop checks the instance state repeatedly.

`sleep 10` waits ten seconds before the next check. The counter limits the number of checks, so the script does not wait forever.

The maximum wait in this example is about ten minutes, depending on command execution time.

**Important:** The EC2 instance reaching `running` does not guarantee that its operating system or application is ready. AWS status checks and application health checks are separate.

---

## 4. How to Run the Project

### Step 1: Create the script

```bash
nano create-ec2.sh
```

Paste the code and save the file.

### Step 2: Make the script executable

```bash
chmod +x create-ec2.sh
```

### Step 3: Update the AWS resource values

Before running, update these variables:

- `AMI_ID`
- `INSTANCE_TYPE`
- `KEY_NAME`
- `SUBNET_ID`
- `SECURITY_GROUP_ID`
- `INSTANCE_NAME`

The AMI, subnet, security group, and key pair must be suitable for the selected AWS region. The security group should allow only the network access you actually need.

### Step 4: Run the script

```bash
./create-ec2.sh
```

The script will check AWS CLI, install it if needed, ask for your credentials, verify access, create the instance, and wait for its state to become `running`.

### Step 5: Verify the instance

You can check your EC2 instances in the AWS Console, or run:

```bash
aws ec2 describe-instances \
    --query 'Reservations[].Instances[].[InstanceId,State.Name]' \
    --output table
```

---

## 5. Important AWS Notes

1. **AWS credentials:** Use credentials with only the permissions required for the project. Creating an instance may require additional permissions for EC2, tagging, and related resources.
2. **AMI ID:** AMI IDs are region-specific. An AMI from one region may not work in another.
3. **Key pair:** The key pair must already exist in the selected region. This script does not create one.
4. **Security group:** The security group must belong to the correct VPC and be suitable for the selected subnet.
5. **Costs:** EC2 instances and related resources can incur charges. Stop or terminate resources when you no longer need them.
6. **Duplicate instances:** Running the script more than once can create more than one instance. Check your AWS account before running it again.
7. **Credentials:** Do not commit access keys, secret keys, or other credentials to GitHub.

---

## 6. Improvements to Try Later

After understanding this script, try adding these features:

- Accept the instance type and instance name as command-line arguments.
- Ask the user to select an AWS region.
- Support multiple security group IDs.
- Add a command to stop or terminate an instance by its ID.
- Save the created instance ID to a file.
- Add a dry-run option where supported by the AWS CLI.
- Add better logging and error messages.
- Use an AWS profile or IAM role instead of entering credentials each time.

---

## 7. What You Learned

By completing this project, you practise Bash functions, variables, user input, loops, command substitution, error handling, environment variables, Linux commands, and AWS CLI automation.

This is a useful beginner project for learning how DevOps scripts can automate cloud tasks.

---

<p align="center">
  <a href="../All_Syntex.md"><img src="https://img.shields.io/badge/⬅_Back-blue?style=for-the-badge" alt="Back"></a>
  <a href="./ch_2.md"><img src="https://img.shields.io/badge/Forward_➡-green?style=for-the-badge" alt="Forward"></a>
</p>
