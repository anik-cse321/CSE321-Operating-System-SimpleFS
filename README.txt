CSE 321: Operating Systems
SimpleFS Lab Term Project - Group 5

GROUP MEMBERS
1. Anik Rahman - 22299368
2. MD. Shahriar Saif - 22299360
3. Ayasha Islam - 19241002

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

1. Anik Rahman
   - Primary responsibility for SimpleFS implementation.
   - Builder and Adder development.
   - Compilation, debugging, testing, and final verification.
   - Documentation and final submission preparation.

2. MD. Shahriar Saif
   - Assigned responsibility for specification review and test-case review.

3. Ayasha Islam
   - Assigned responsibility for documentation review and submission checking.

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