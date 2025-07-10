# C++ Version Control System

## Overview

This project is a lightweight, filesystem-based version control system (VCS) written in C++. Inspired by the core concepts of Git, it's designed to manage and track changes to source code over time. It demonstrates fundamental VCS operations like initializing a repository, staging changes, committing snapshots, and viewing project history, all through a simple command-line interface.

The system uses the C++17 `<filesystem>` library to interact with the file system, creating a `.vcs` directory to store all versioning data, including commits and a staging area.

## Features

- **Initialize Repository (`vcs init`)**: Sets up the necessary `.vcs` directory structure in your project folder to start tracking changes.
- **Stage Changes (`vcs add`)**: Copies files and directories to a temporary "staging area" before they are committed. It supports adding all changes (`.`) or specifying individual files/directories.
- **Create Commits (`vcs commit`)**: Creates a permanent snapshot of the staged files. Each commit is assigned a unique 8-character ID (hash), a user-provided message, and a timestamp.
- **View History (`vcs log`)**: Displays a chronological list of all commits, showing their ID, message, and date.
- **Revert to a Past State (`vcs revert`)**: Creates a *new* commit that mirrors the exact file state of a specified previous commit. This action doesn't alter the existing history but appends a new "revert" state to it.

## How It Works

The entire version control system is built upon a specific directory structure created within your project.

### Core Directory Structure

When you run `vcs init`, the following structure is created:

```
project-directory/
├── .vcs/
│   ├── commits/
│   │   ├── 0x1111/               # The initial commit "node"
│   │   │   ├── Data/             # Snapshot of files for this commit
│   │   │   ├── commitInfo.txt    # Metadata: ID, message, timestamp
│   │   │   └── nextCommitInfo.txt # "Pointer" to the next commit's ID
│   │   └── [commit_hash]/        # Subsequent commit "nodes"
│   └── staging_area/            # Temporary area for files added with vcs add
└── vcs                          # The compiled executable
```

### The On-Disk "Linked List"

The commit history is managed as a **singly linked list on the filesystem**.

- **Node**: Each directory inside `.vcs/commits/` represents a `commitNode`.
- **Data**: The `commitInfo.txt` file within each node directory stores the commit's metadata (ID, message, timestamp). The `Data/` subdirectory holds the actual snapshot of the project's files at the time of the commit.
- **Pointer**: The `nextCommitInfo.txt` file acts as the `next` pointer. It contains the ID of the *next* commit in the sequence.

The chain starts with the initial commit, hardcoded as `0x1111`. To find the latest commit or traverse the history, the program starts at `0x1111` and follows the chain of IDs stored in `nextCommitInfo.txt` files.

### Command Workflow

1. **`./vcs init`**: Creates the `.vcs`, `.vcs/staging_area`, and `.vcs/commits` directories.
2. **`./vcs add <files>`**: Copies the specified files and directories into the `.vcs/staging_area`.
3. **`./vcs commit -m "..."`**:
    - Traverses the on-disk linked list to find the last commit.
    - Generates a new random 8-character ID for the new commit.
    - Creates a new directory `.vcs/commits/[new_id]`.
    - Copies the entire contents of `.vcs/staging_area/` into the new commit's `Data/` folder.
    - Writes the new ID, commit message, and current timestamp into `.vcs/commits/[new_id]/commitInfo.txt`.
    - Updates the `nextCommitInfo.txt` of the *previous* commit to contain the new commit's ID, linking it into the chain.
4. **`./vcs log`**: Starts at the head (`0x1111`) and iterates through the commit chain, reading and printing the `commitInfo.txt` from each commit directory along the way.
5. **`./vcs revert <commit_id>`**:
    - Performs the same actions as a normal commit (finds the end of the chain, generates a new ID, etc.).
    - However, instead of copying from `.vcs/staging_area`, it copies the contents of `.vcs/commits/[commit_id]/Data/` into the new commit's `Data/` folder.
    - This creates a new commit at the end of the history that restores the project's state to that of an older commit.

## Run Locally

### Clone the project

```bash
git clone https://github.com/hardik4tiwari/version-control-system
```

### Go to the project directory

```bash
cd version-control-system
```

### Compile main.cpp

You need a C++ compiler that supports the C++17 standard (for the `<filesystem>` library).

```bash
g++ src/main.cpp -o vcs -std=c++17 -lfilesystem
```

> **Note**: The `-lfilesystem` flag might be required on some Linux distributions or with certain compiler versions to link the filesystem library correctly.

This will create an executable file named `vcs` in the current directory. You can move this executable to any other project directory where you want to use it.

## Supported Commands

### `vcs`
Displays help information and a list of available commands.

```bash
./vcs
```

### `vcs init`
Initializes an empty VCS repository in the current directory.

```bash
./vcs init
```

### `vcs add`
Adds files to the staging area. It can be used in several ways:

To add all files and directories in the current folder (excluding `.vcs` itself):

```bash
./vcs add .
```

To add a specific file:

```bash
./vcs add <filename>
```

To add multiple files or directories:

```bash
./vcs add <file1> <directory1> <file2>
```

### `vcs commit`
Commits the staged changes with a required message.

```bash
./vcs commit -m "Your commit message"
```

### `vcs revert`
Creates a new commit that restores the project files to the state of a specific commit.

```bash
./vcs revert <commit_id>
```

### `vcs log`
Displays the commit history, showing the ID, message, and timestamp for each commit.

```bash
./vcs log
```
