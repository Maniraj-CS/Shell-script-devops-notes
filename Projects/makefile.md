# Makefile Notes for DevOps

## 1. What is Makefile?

A `Makefile` is a file that contains commands and rules. We use it to run commands easily without typing them again and again.

For example, in a project, we may need to:
- Build an application.
- Run tests.
- Start or stop Docker containers.
- Clean old files.
- Run scripts.

Instead of running every command manually, we can create a Makefile and run these commands using `make`.

### Example Without Makefile

```bash
docker compose build
docker compose up -d
docker compose ps
```

We need to type each command separately.

### Example With Makefile

Create a file named `Makefile`:

```makefile
build:
	docker compose build

up:
	docker compose up -d

status:
	docker compose ps
```

Now run:

```bash
make build
make up
make status
```

**Remember:** A Makefile usually uses a tab before each command, not spaces.

---

## 2. Install Make

On Ubuntu or Debian, install Make using:

```bash
sudo apt update
sudo apt install make
```

Check the installed version:

```bash
make --version
```

---

## 3. Basic Makefile Syntax

A Makefile contains targets, dependencies, and commands.

```makefile
target: dependencies
	command
```

### Explanation

- **Target:** The task we want to run.
- **Dependencies:** Other targets or files that the target needs.
- **Command:** The command that runs when we call the target.

Example:

```makefile
hello:
	echo "Hello, DevOps!"
```

Run:

```bash
make hello
```

Output:

```text
echo "Hello, DevOps!"
Hello, DevOps!
```

Make prints the command before running it.

---

## 4. Understanding Targets

A target is a name used to run a task.

```makefile
hello:
	echo "Hello"

backup:
	echo "Running backup"

deploy:
	echo "Deploying application"
```

Run a target by using `make` followed by the target name:

```bash
make hello
make backup
make deploy
```

If you run only:

```bash
make
```

Make normally runs the first target in the Makefile.

---

## 5. Why Must We Use a Tab?

Make expects each command under a target to start with a tab by default.

Correct:

```makefile
hello:
	echo "Hello"
```

Incorrect:

```makefile
hello:
    echo "Hello"
```

The second example uses spaces instead of a tab and may cause a `missing separator` error.

---

## 6. Multiple Commands in One Target

We can write multiple commands under one target.

```makefile
setup:
	echo "Updating packages"
	echo "Installing tools"
	echo "Setup completed"
```

Run:

```bash
make setup
```

Each command normally runs in a separate shell.

**Important:** If you change directories in one command, the next command normally starts in the original working directory.

For example:

```makefile
wrong:
	cd /tmp
	pwd
```

The `pwd` command may show the original directory, not `/tmp`.

To run both commands in the same shell:

```makefile
correct:
	cd /tmp && pwd
```

---

## 7. Using Variables in Makefile

Variables help us store values and reuse them.

```makefile
APP_NAME = my-app
VERSION = 1.0

info:
	echo "Application: $(APP_NAME)"
	echo "Version: $(VERSION)"
```

Run:

```bash
make info
```

Output:

```text
Application: my-app
Version: 1.0
```

### Variable Syntax

```makefile
NAME = Maniraj
```

Use the variable like this:

```makefile
hello:
	echo "Hello, $(NAME)"
```

In a Makefile, `$(NAME)` reads the Make variable.

**Note:** Make supports different assignment operators, such as `=`, `:=`, `?=`, and `+=`. Learn the basic `=` form first.

---

## 8. Passing Variables from the Command Line

We can pass values when running Make.

Makefile:

```makefile
hello:
	echo "Hello, $(NAME)"
```

Run:

```bash
make hello NAME=Maniraj
```

Output:

```text
Hello, Maniraj
```

This is useful when we want to change values without editing the Makefile.

**Security note:** Do not pass passwords, AWS secret keys, or other sensitive information as Make variables on the command line.

---

## 9. Dependencies in Makefile

A dependency is something that a target needs before it can run.

```makefile
install:
	echo "Installing application"

test: install
	echo "Running tests"
```

Run:

```bash
make test
```

Output:

```text
Installing application
Running tests
```

Here, `test` depends on `install`, so Make runs `install` first.

### Multiple Dependencies

```makefile
build:
	echo "Building application"

test:
	echo "Testing application"

deploy: build test
	echo "Deploying application"
```

Run:

```bash
make deploy
```

Make runs the prerequisites before the `deploy` recipe. In this example, `build` and `test` do not depend on each other, so do not use this pattern when one must always run before the other.

---

## 10. The `.PHONY` Target

Sometimes a target has the same name as a file or directory in the project.

Make may think that the target is already up to date and skip its commands.

Use `.PHONY` for task names that are not actual files.

```makefile
.PHONY: hello clean

hello:
	echo "Hello, DevOps!"

clean:
	echo "Cleaning project"
```

Now Make treats `hello` and `clean` as tasks.

**Common `.PHONY` targets:**
- `install`
- `build`
- `test`
- `run`
- `clean`
- `deploy`

For most command-based Makefile tasks, using `.PHONY` is a good habit.

---

## 11. The `@` Symbol

By default, Make prints each command before running it.

Use `@` to hide a command from the output.

```makefile
.PHONY: hello

hello:
	@echo "Hello, DevOps!"
```

Run:

```bash
make hello
```

Output:

```text
Hello, DevOps!
```

Without `@`, the output would also show:

```text
echo "Hello, DevOps!"
```

Use `@` when you want cleaner output.

---

## 12. The `-` Symbol

A hyphen before a recipe command tells Make to ignore that command's error status.

```makefile
.PHONY: clean

clean:
	-rm old-file.txt
	echo "Cleanup finished"
```

If `old-file.txt` does not exist, `rm` normally fails. The `-` tells Make to continue anyway.

**Warning:** Use this only when the error is safe to ignore. Do not hide important build or deployment failures.

A safer cleanup command is often:

```makefile
.PHONY: clean

clean:
	rm -f old-file.txt
```

The `-f` option makes `rm` ignore a missing file.

---

## 13. Automatic Variables

Make provides special variables that can be used inside recipes.

| Variable | Meaning |
|---|---|
| `$@` | Name of the current target |
| `$<` | First prerequisite |
| `$^` | All prerequisites |
| `$?` | Prerequisites that are newer than the target |

Example:

```makefile
.PHONY: show

show: file1.txt file2.txt
	@echo "Target: $@"
	@echo "First prerequisite: $<"
	@echo "All prerequisites: $^"
```

Run:

```bash
make show
```

Output:

```text
Target: show
First prerequisite: file1.txt
All prerequisites: file1.txt file2.txt
```

This example displays the prerequisite names; the files do not need to exist because `show` is marked as phony.

**Important:** Make uses `$@` for the current target. Inside a shell command, `$$` is used when you need to pass a literal dollar sign to the shell.

---

## 14. Using Makefile with Docker

Makefile is useful for common Docker commands.

Suppose your project contains a `Dockerfile` and a `compose.yaml` file.

Create this Makefile:

```makefile
.PHONY: build up down logs status

build:
	docker compose build

up:
	docker compose up -d

down:
	docker compose down

logs:
	docker compose logs -f

status:
	docker compose ps
```

Run these commands:

```bash
make build
make up
make status
make logs
make down
```

### What Do These Targets Do?

| Target | Purpose |
|---|---|
| `build` | Builds the images |
| `up` | Starts the services in the background |
| `down` | Stops and removes the Compose containers and network |
| `logs` | Shows and follows service logs |
| `status` | Shows the service status |

Makefile does not replace Docker. It provides a shorter way to run Docker commands.

---

## 15. Using Makefile with a Shell Script

Suppose your project contains a script named `backup.sh`.

```makefile
.PHONY: backup

backup:
	bash backup.sh
```

Run:

```bash
make backup
```

This runs the shell script through Bash.

You can also use Make to run a script that checks Linux system information.

```makefile
.PHONY: system-info

system-info:
	bash system-info.sh
```

Run:

```bash
make system-info
```

This is useful when a project contains multiple scripts and you want simple commands to run them.

---

## 16. Using Makefile with AWS CLI

We can use Makefile to make common AWS CLI commands easier to run.

```makefile
.PHONY: aws-identity ec2-list

aws-identity:
	aws sts get-caller-identity

ec2-list:
	aws ec2 describe-instances \
		--query 'Reservations[].Instances[].[InstanceId,State.Name]' \
		--output table
```

Run:

```bash
make aws-identity
make ec2-list
```

The commands use your existing AWS CLI credentials and configuration.

**Important:** Configure AWS credentials safely before running these targets. Makefile is not a secure place to store access keys or secret keys.

---

## 17. Using Makefile with a C/C++ Project

Make was originally created to help build programs and manage dependencies.

For example, a C++ project may contain:

```text
project/
├── main.cpp
└── Makefile
```

Makefile:

```makefile
CXX = g++
CXXFLAGS = -Wall -Wextra -std=c++17

.PHONY: clean

main: main.cpp
	$(CXX) $(CXXFLAGS) main.cpp -o main

clean:
	rm -f main
```

Build the program:

```bash
make
```

Run it:

```bash
./main
```

Remove the generated executable:

```bash
make clean
```

Here, `main` depends on `main.cpp`. If the source file changes, Make can rebuild the executable.

Unlike a phony task, `main` is a real file target. Make can check whether it needs rebuilding based on timestamps.

---

## 18. Useful Make Commands

| Command | Meaning |
|---|---|
| `make` | Run the default target |
| `make build` | Run the `build` target |
| `make clean` | Run the `clean` target |
| `make -n build` | Show commands without running them |
| `make -B build` | Force targets to be considered out of date |
| `make -f custom.mk` | Use a different Makefile |
| `make -j4` | Allow up to four jobs to run in parallel |

**Warning:** `make -n` is useful for reviewing commands, but it is not a complete security check. Some Makefile constructs can still execute commands while Make is processing the file.

Parallel jobs (`-j`) are useful when tasks are independent. Avoid using them when tasks must run in a specific order.

---

## 19. Common Makefile Errors

### Error 1: `missing separator`

**Reason:** A command usually starts with spaces instead of a tab.

**Fix:** Use a tab before the command.

### Error 2: `No rule to make target`

**Reason:** Make cannot find a required target or prerequisite.

**Fix:** Check the target name, file name, and path.

### Error 3: `Nothing to be done`

**Reason:** Make thinks the target is already up to date.

**Fix:** If the target is a command rather than a generated file, mark it as `.PHONY`.

### Error 4: Command not found

**Reason:** The required program is not installed or cannot be found in the command's environment.

**Fix:** Install the program and check whether it is available in your `PATH`.

### Error 5: Permission denied

**Reason:** The current user cannot execute a file or access a resource.

**Fix:** Check file permissions and resource access. Avoid using `sudo` automatically for every command.

---

## 20. Practice Project

Create a project that uses Makefile to manage a simple DevOps workflow.

### Project Structure

```text
devops-project/
├── Makefile
├── scripts/
│   ├── backup.sh
│   └── system-info.sh
└── compose.yaml
```

### Tasks

- Create a `build` target to build Docker images.
- Create an `up` target to start Docker services.
- Create a `down` target to stop the services.
- Create a `backup` target to run `backup.sh`.
- Create a `system-info` target to display Linux system information.
- Create a `status` target to show running containers.
- Add `.PHONY` to command-based targets.

### Goal

Run your common project tasks using simple commands:

```bash
make build
make up
make status
make system-info
make backup
make down
```

## Summary

Makefile helps us run repeated commands using short, easy-to-remember targets. In DevOps, we can use it to manage Docker commands, shell scripts, builds, tests, and AWS CLI tasks.

**Key things to remember:**

- `target:` defines a task.
- A tab is required before recipe commands by default.
- `$(VARIABLE)` reads a Make variable.
- Dependencies run before their target's recipe.
- `.PHONY` marks targets that are not actual files.
- `$@` refers to the current target.
- Makefile simplifies commands, but it does not replace the tools those commands use.

---

<p align="center">
  <a href="./ec2_creating.md">
    <img src="https://img.shields.io/badge/⬅_Back-blue?style=for-the-badge" alt="Back">
  </a>
  <a href="./log_and_rotation.md">
    <img src="https://img.shields.io/badge/Forward_➡-green?style=for-the-badge" alt="Forward">
  </a>
</p>