CSE 321: Operating Systems
SimpleFS Lab Term Project - Group 5

GROUP MEMBERS
1. Anik Rahman - 22299368
2. MD. Shahriar Saif - 22299360
3. Ayasha Islam - 19241002

Project Goal :
The goal of this project was to implement a small educational filesystem called SimpleFS for the CSE321 Operating Systems course. We developed two main components: a filesystem builder that creates and initializes a valid SimpleFS disk image by setting up the superblock, inode bitmap, data bitmap, inode table, and root directory, and a file adder that inserts regular files into the filesystem using inode and data-block allocation, directory updates, and validation checks. We also tested the system through different scenarios such as normal file insertion, duplicate files, zero-byte files, maximum-size files, oversized files, and filename limits to verify that the filesystem works correctly according to the given specifications.


IMPLEMENTATION DESCRIPTION
- simplefs_builder.c creates a 262144-byte SimpleFS image and initializes the
  superblock, inode bitmap, data bitmap, root inode, and "." / ".." directory entries.
- simplefs_adder.c adds regular files to the root directory using first-fit inode
  and data-block allocation, supports up to three direct blocks per file, updates
  the required bitmaps and directory metadata, and handles the required error cases.

KNOWN LIMITATIONS / PROBLEMS
- No known problems were found during testing.
- The implementation follows the project specification and supports regular files
  up to 12288 bytes as required.

WORK DISTRIBUTION

Anik Rahman
Responsible for:

Development of simplefs_adder.c
Implementing file insertion functionality
Implementing inode and data block allocation logic
Handling duplicate filename detection
Implementing file size and filename validation
Testing regular file insertion scenarios
Debugging and verification of Adder module
Integration testing with the complete SimpleFS system
MD. Shahriar Saif
Responsible for:

Development of simplefs_builder.c
Implementing filesystem image creation logic
Initializing superblock, inode bitmap, and data bitmap
Creating and configuring the root directory
Implementing root directory entries (. and ..)
Compilation and debugging of Builder module
Verification of initial filesystem structure
Ayasha Islam
Responsible for:

Maintaining and reviewing simplefs.h
Reviewing filesystem constants and data structures
Verifying compatibility between Builder and Adder modules
Preparing project documentation and README files
Reviewing test cases and expected outputs
Performing final documentation and submission checks

FILES
1. simplefs.h          - fixed constants and structures; do not modify unless instructed.
2. simplefs_builder.c  - complete TODO 1 through TODO 6.
3. simplefs_adder.c    - complete TODO 1 through TODO 11.
4. test1.txt, test2.txt, test3.txt - simple sample input files.

COMPILE
gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder

RUN
./simplefs_builder --image disk.img
./simplefs_adder --input disk.img --file test1.txt

IMPORTANT
- Follow the official project specification for the exact layout and expected behavior.
- Source files must be in the current working directory.
- Maximum regular-file size is 12288 bytes.
- Implement only the clearly marked TODO sections.
