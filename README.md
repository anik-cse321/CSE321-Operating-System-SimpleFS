# CSE321 Operating Systems — SimpleFS

A lab-term project for **CSE321: Operating Systems** implementing a small custom filesystem called **SimpleFS** in the C programming language.

---

## Project Information

- **Course:** CSE321 — Operating Systems
- **Project:** SimpleFS Lab Term Project
- **Lab Section:** 7
- **Group:** 5
- **Language:** C
- **Development Environment:** GitHub Codespaces / Linux
- **Compiler:** GCC

### Group Members

| Name | Student ID |
|---|---:|
| Anik Rahman | 22299368 |
| MD. Shahriar Saif | 22299360 |
| Ayasha Islam | 19241002 |

---

## Project Overview

The objective of this project is to implement the core functionality of a small educational filesystem called **SimpleFS**.

The project contains two main programs:

### `simplefs_builder.c`

Creates and initializes a new SimpleFS filesystem image.

### `simplefs_adder.c`

Adds regular files to an existing SimpleFS filesystem image.

The filesystem maintains:

- A superblock
- An inode bitmap
- A data-block bitmap
- An inode table
- A root directory
- Data blocks for regular files

---

# SimpleFS Specifications

| Property | Value |
|---|---:|
| Block size | 4096 bytes |
| Total blocks | 64 |
| Total filesystem size | 262144 bytes |
| Total inodes | 32 |
| Data blocks | 60 |
| Root inode | 1 |
| Root data block | 4 |
| Maximum direct blocks per file | 3 |
| Maximum regular-file size | 12288 bytes |
| Directory-entry size | 64 bytes |
| Maximum filename length | 58 characters |

The total filesystem size is:

```text
64 blocks × 4096 bytes = 262144 bytes
```

The maximum regular-file size is:

```text
3 direct blocks × 4096 bytes = 12288 bytes
```

---

# Disk Layout

SimpleFS uses the following fixed layout:

| Block | Purpose |
|---:|---|
| 0 | Superblock |
| 1 | Inode bitmap |
| 2 | Data bitmap |
| 3 | Inode table |
| 4 | Root-directory data |
| 5–63 | Available data blocks |

Graphically:

```text
+---------+-------------------------+
| Block 0 | Superblock              |
+---------+-------------------------+
| Block 1 | Inode Bitmap            |
+---------+-------------------------+
| Block 2 | Data Bitmap             |
+---------+-------------------------+
| Block 3 | Inode Table             |
+---------+-------------------------+
| Block 4 | Root Directory          |
+---------+-------------------------+
| Block 5 | File Data / Free        |
+---------+-------------------------+
|   ...   | File Data / Free        |
+---------+-------------------------+
| Block63 | File Data / Free        |
+---------+-------------------------+
```

---

# Filesystem Structures

## Superblock

The superblock stores the main filesystem configuration:

- Magic number
- Block size
- Total block count
- Inode count
- Inode bitmap block
- Data bitmap block
- Inode table block
- Data region block
- Root inode number

The SimpleFS magic number is:

```text
0x53465331
```

---

## Inode Bitmap

The inode bitmap tracks which inode entries are currently allocated.

For example:

```text
00000001
```

means only inode 1 is allocated.

After adding one regular file:

```text
00000011
```

which is decimal:

```text
3
```

This means:

```text
inode 1 → root directory
inode 2 → first regular file
```

---

## Data Bitmap

The data bitmap tracks which data blocks in the data region are currently allocated.

Initially, the root directory uses Block 4.

After adding a regular file that uses Block 5:

```text
bit 0 → Block 4
bit 1 → Block 5
```

Therefore the first bitmap byte becomes:

```text
00000011 = 3
```

---

## Inode Table

SimpleFS contains:

```text
32 inodes
```

Each inode occupies:

```text
128 bytes
```

Therefore:

```text
32 × 128 = 4096 bytes
```

which fits exactly inside Block 3.

Each inode contains:

- File type
- Link count
- File size
- Three direct data-block pointers
- Reserved space

---

## Directory Entry

Each directory entry occupies:

```text
64 bytes
```

A directory entry stores:

- Inode number
- Entry type
- Filename

The root directory initially contains:

```text
.
..
```

Each requires 64 bytes, so the initial root-directory size is:

```text
2 × 64 = 128 bytes
```

After adding one regular file:

```text
3 × 64 = 192 bytes
```

---

# Builder Implementation

The file:

```text
simplefs_builder.c
```

creates a new filesystem image and initializes all required filesystem metadata.

The Builder implementation performs the following operations:

1. Creates a filesystem image.
2. Sets the image size to 262144 bytes.
3. Initializes the superblock.
4. Initializes the inode bitmap.
5. Marks the root inode as allocated.
6. Initializes the data bitmap.
7. Marks the root-directory data block as allocated.
8. Creates the root inode.
9. Sets the root inode type to directory.
10. Sets the root inode link count.
11. Sets the root-directory size.
12. Assigns Block 4 to the root inode.
13. Creates the `.` directory entry.
14. Creates the `..` directory entry.
15. Writes the initialized structures to their correct offsets.

---

# Adder Implementation

The file:

```text
simplefs_adder.c
```

adds regular files to an existing SimpleFS filesystem image.

The implementation includes:

- Finding a free inode
- Finding free data blocks
- First-fit allocation
- Duplicate filename detection
- Free root-directory entry search
- Filename-length validation
- Maximum-file-size validation
- Required-block calculation
- File-data copying
- New inode creation
- Direct-block pointer assignment
- Inode bitmap update
- Data bitmap update
- Root-directory entry creation
- Root inode size update

---

# Allocation Strategy

SimpleFS uses **first-fit allocation**.

For inode allocation, the program scans the inode bitmap from the beginning and selects the first available inode.

For data-block allocation, the program scans the data bitmap from the beginning and selects the first free data block.

Example:

```text
Block 4 → Root directory
Block 5 → test1.txt
Block 6 → test2.txt
Block 7 → test3.txt
```

---

# Zero-Byte File Handling

A file with size:

```text
0 bytes
```

still receives an inode and a root-directory entry.

However, it requires:

```text
0 data blocks
```

Its inode therefore contains:

```text
size = 0

direct[0] = 0
direct[1] = 0
direct[2] = 0
```

---

# Compilation

Compile the Builder using:

```bash
gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
```

Compile the Adder using:

```bash
gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder
```

The final implementation compiled successfully using these options without warnings or errors.

---

# Running the Project

## Create a filesystem image

```bash
./simplefs_builder --image disk.img
```

Expected output:

```text
SimpleFS image created successfully: disk.img
```

---

## Add a regular file

```bash
./simplefs_adder --input disk.img --file test1.txt
```

Expected output:

```text
test1.txt added successfully to disk.img
```

---

# Testing and Verification

The project was tested progressively by examining both program behaviour and the raw filesystem image.

---

## 1. Filesystem Image Size

Command:

```bash
stat -c %s disk.img
```

Verified output:

```text
262144
```

---

## 2. Superblock Verification

Command used:

```bash
od -An -t u4 -N 36 disk.img
```

Verified values included:

```text
1397117745 4096 64 32 1 2 3 4 1
```

The first value corresponds to:

```text
0x53465331
```

---

## 3. Initial Inode Bitmap

Command:

```bash
od -An -t u1 -j 4096 -N 1 disk.img
```

Initial verified value:

```text
1
```

This represents the allocated root inode.

---

## 4. Initial Data Bitmap

Command:

```bash
od -An -t u1 -j 8192 -N 1 disk.img
```

Initial verified value:

```text
1
```

This represents the root-directory data block.

---

## 5. Root Inode Type and Link Count

Command:

```bash
od -An -t u2 -j 12288 -N 4 disk.img
```

Verified output:

```text
2 2
```

Meaning:

```text
type  = directory
links = 2
```

---

## 6. Root Directory Initial Size

Command:

```bash
od -An -t u4 -j 12292 -N 4 disk.img
```

Verified output:

```text
128
```

---

## 7. Root Direct Block Pointer

Command:

```bash
od -An -t u4 -j 12296 -N 12 disk.img
```

Verified output:

```text
4 0 0
```

---

## 8. Root `.` Entry

The `.` entry was verified directly from the disk image.

The stored values confirmed:

```text
inode = 1
type  = directory
name  = "."
```

---

## 9. Root `..` Entry

The `..` entry was also verified.

The stored values confirmed:

```text
inode = 1
type  = directory
name  = ".."
```

---

# Regular File Testing

Three supplied sample files were tested:

```text
test1.txt
test2.txt
test3.txt
```

Their sizes were:

```text
test1.txt = 22 bytes
test2.txt = 77 bytes
test3.txt = 23 bytes
```

Each required one data block.

---

## test1.txt

Command:

```bash
./simplefs_adder --input disk.img --file test1.txt
```

Output:

```text
test1.txt added successfully to disk.img
```

### Inode Bitmap

Verified:

```text
3
```

### Data Bitmap

Verified:

```text
3
```

### Inode 2 Type and Links

Command:

```bash
od -An -t u2 -j 12416 -N 4 disk.img
```

Output:

```text
1 1
```

### File Size

Command:

```bash
od -An -t u4 -j 12420 -N 4 disk.img
```

Output:

```text
22
```

### Direct Block Pointers

Command:

```bash
od -An -t u4 -j 12424 -N 12 disk.img
```

Output:

```text
5 0 0
```

### Root Directory Size

Verified:

```text
192
```

### Content Verification

Command:

```bash
dd if=disk.img bs=1 skip=20480 count=22 status=none | cmp - test1.txt && echo MATCH
```

Output:

```text
MATCH
```

---

# Duplicate Filename Test

The same file was added again:

```bash
./simplefs_adder --input disk.img --file test1.txt
```

Correct error:

```text
Error: file already exists in SimpleFS.
```

After rejection, both the inode bitmap and data bitmap were checked and confirmed unchanged.

---

# test2.txt

Command:

```bash
./simplefs_adder --input disk.img --file test2.txt
```

Output:

```text
test2.txt added successfully to disk.img
```

Verified:

```text
inode bitmap = 7
data bitmap  = 7
file size    = 77
direct block = 6
```

Content was compared directly against the original file and produced:

```text
MATCH
```

---

# test3.txt

Command:

```bash
./simplefs_adder --input disk.img --file test3.txt
```

Output:

```text
test3.txt added successfully to disk.img
```

Verified:

```text
inode bitmap = 15
data bitmap  = 15
file size    = 23
direct block = 7
```

Content verification produced:

```text
MATCH
```

After three files were added, the root-directory size became:

```text
320 bytes
```

corresponding to:

```text
.
..
test1.txt
test2.txt
test3.txt
```

---

# Zero-Byte File Test

A zero-byte file was created:

```bash
: > empty.txt
```

Its size was verified using:

```bash
stat -c %s empty.txt
```

Output:

```text
0
```

It was added using:

```bash
./simplefs_adder --input disk.img --file empty.txt
```

Output:

```text
empty.txt added successfully to disk.img
```

The inode bitmap changed as expected, while the data bitmap remained unchanged.

Its direct pointers were verified as:

```text
0 0 0
```

This confirmed that a zero-byte file receives an inode but consumes no data block.

---

# Maximum File Size Test

A file of exactly 12288 bytes was created:

```bash
dd if=/dev/zero of=maxfile.bin bs=12288 count=1 status=none
```

The size was confirmed using:

```bash
stat -c %s maxfile.bin
```

Output:

```text
12288
```

The file was added successfully:

```bash
./simplefs_adder --input disk.img --file maxfile.bin
```

Output:

```text
maxfile.bin added successfully to disk.img
```

The three direct block pointers were verified as:

```text
8 9 10
```

The complete 12288-byte stored content was compared against the original file:

```bash
dd if=disk.img bs=1 skip=32768 count=12288 status=none | cmp - maxfile.bin && echo MATCH
```

Output:

```text
MATCH
```

---

# Oversized File Test

A file exactly one byte larger than the maximum was created:

```bash
dd if=/dev/zero of=toolarge.bin bs=12289 count=1 status=none
```

Verified size:

```text
12289
```

Attempting to add it produced:

```text
Error: file is too large for SimpleFS.
```

The inode bitmap and data bitmap were then checked and confirmed unchanged.

---

# Filename Length Test

SimpleFS supports filenames with a maximum length of:

```text
58 characters
```

A 59-character filename was created using:

```bash
name=$(printf 'a%.0s' {1..59}); printf "x" > "$name"; echo ${#name}
```

Output:

```text
59
```

Attempting to add this file produced:

```text
Error: file name is too long.
```

The inode and data bitmaps remained unchanged.

The temporary file was removed after testing.

---

# Final Compilation Test

Both source files were compiled together using:

```bash
gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder && \
gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder && \
echo COMPILE_OK
```

Output:

```text
COMPILE_OK
```

No compiler warnings or errors were produced.

---

# Fresh End-to-End Verification

A completely fresh filesystem image was created:

```bash
./simplefs_builder --image finalcheck.img
```

Then:

```bash
./simplefs_adder --input finalcheck.img --file test1.txt
```

The fresh filesystem was inspected and produced:

```text
image_size=262144
inode_bitmap=3
data_bitmap=3
root_size=192
inode2_type_links=1 1
inode2_size=22
inode2_ptrs=5 0 0
content=MATCH
```

This confirmed that the Builder and Adder work correctly together starting from a completely clean filesystem image.

---

# Development Environment

The project was developed and tested using:

- C programming language
- GCC compiler
- GitHub Codespaces
- Linux command-line utilities
- Visual Studio Code / browser-based Codespaces editor
- Windows as the primary local operating system

GitHub Codespaces was used as a temporary Linux environment for compilation and testing.

---

# Project Files

```text
README.txt
simplefs.h
simplefs_builder.c
simplefs_adder.c
```

### `simplefs.h`

Contains the fixed:

- Constants
- Structures
- Function declarations
- Filesystem configuration

### `simplefs_builder.c`

Implements filesystem-image creation and root filesystem initialization.

### `simplefs_adder.c`

Implements regular-file insertion and filesystem metadata updates.

### `README.txt`

Contains the official project submission documentation and compilation instructions.

---

# Work Distribution

## Anik Rahman

Primary responsibility for:

- SimpleFS implementation
- Builder development
- Adder development
- Compilation
- Debugging
- Functional testing
- Boundary testing
- Filesystem inspection
- Final verification
- Documentation
- Submission preparation

## MD. Shahriar Saif

Assigned responsibility for:

- Specification review
- Test-case review

## Ayasha Islam

Assigned responsibility for:

- Documentation review
- Submission checking

---

# Known Limitations

The implementation follows the scope of the supplied SimpleFS project specification.

Current filesystem limitations include:

- Maximum regular-file size: **12288 bytes**
- Maximum of **3 direct data blocks per regular file**
- Files are inserted into the **root directory**
- Maximum supported filename length: **58 characters**
- SimpleFS is an educational fixed-layout filesystem rather than a general-purpose filesystem

No known implementation problems were found during the completed testing.

---

# Final Result

The final SimpleFS implementation successfully:

- Creates a valid filesystem image
- Initializes the superblock correctly
- Initializes inode and data bitmaps
- Creates a valid root directory
- Creates `.` and `..` entries
- Allocates inodes using first-fit allocation
- Allocates data blocks using first-fit allocation
- Adds regular files to the root directory
- Maintains correct inode metadata
- Maintains correct filesystem bitmaps
- Stores file data correctly
- Supports zero-byte files
- Supports files up to the full 12288-byte limit
- Rejects duplicate filenames
- Rejects oversized files
- Rejects filenames longer than 58 characters
- Compiles cleanly using GCC
- Passes fresh end-to-end filesystem verification

---

## Repository Purpose

This repository is maintained as a personal academic archive of the **CSE321 Operating Systems SimpleFS Lab Term Project**.

The official course submission was prepared separately using the required submission structure.
