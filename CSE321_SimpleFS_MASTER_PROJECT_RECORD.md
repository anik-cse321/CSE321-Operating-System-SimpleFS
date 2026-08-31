# CSE321 SimpleFS — Master Project Record

**Purpose of this file:**  
This is the single long-term context/archive file for the CSE321 Operating Systems SimpleFS project. It is meant to preserve the complete story of the project so that Anik, the group members, a future instructor/reviewer, or a future ChatGPT session can understand what the project was, how it was implemented, what was tested, what was submitted, what was later archived in GitHub, and what information is confirmed versus reconstructed.

**Course:** CSE321 — Operating Systems  
**Semester:** Summer 2026  
**Lab Section:** 7 (original notation: 07)  
**Group:** 5  
**Project:** SimpleFS — Implementation of a Simple File System in C  
**Primary development environment:** GitHub Codespaces / Linux  
**Compiler used:** GCC 13.3.0 on Ubuntu  
**Primary local OS:** Windows  
**Original official submission ZIP:** `Project_Sec_7_Group_5.zip`  
**Archive repository:** `anik-cse321/CSE321-Operating-System-SimpleFS`  
**Repository visibility:** Private  
**Archive record prepared:** 31 August 2026

---

# 1. Why This Master Record Exists

The original project question/specification was provided through course materials/Google Form. The exact Google Form wording is no longer available. However, the project itself, the completed source files, submission README, terminal command history, detailed documentation, testing evidence, and final package information were preserved.

This record therefore serves two purposes:

1. Preserve the complete project context from beginning to end.
2. Make future continuation possible even if previous ChatGPT conversations, Codespaces, local temporary files, or course links are no longer available.

A future AI assistant should treat this file as a project map and then inspect the actual source files (`simplefs.h`, `simplefs_builder.c`, and `simplefs_adder.c`) for exact implementation details.

**Important accuracy rule:**  
Do not claim the exact original Google Form question is known. The exact wording is lost. The project requirements described here are reconstructed from the preserved official project materials, starter code, completed implementation, README, testing process, and final submission package.

---

# 2. Group Information

| Name | Student ID | Recorded responsibility |
|---|---:|---|
| Anik Rahman | 22299368 | Primary implementation, builder/adder development, compilation, debugging, testing, final verification, documentation, submission preparation |
| MD. Shahriar Saif | 22299360 | Specification review and test-case review |
| Ayasha Islam | 19241002 | Documentation review and submission checking |

The final README recorded Anik as having primary responsibility for the implementation and final technical verification.

---

# 3. What the Project Actually Does

SimpleFS is a **small educational file system stored inside a binary image file**. It is not a Linux filesystem driver and is not mounted by the operating system.

The project demonstrates core file-system ideas:

- superblock metadata
- inode allocation
- data-block allocation
- inode bitmap
- data-block bitmap
- inode table
- root directory
- directory entries
- direct block pointers
- first-fit allocation
- binary disk-image layout
- file insertion
- metadata consistency
- file-content verification
- boundary/error handling

The project contains two main executable programs:

## `simplefs_builder.c`

Creates and initializes a new empty SimpleFS image.

## `simplefs_adder.c`

Adds one regular host file into an existing SimpleFS image.

The fixed header file:

## `simplefs.h`

Contains the constants and filesystem structures used by both programs. The project instructions treated this file as fixed starter material that should not be modified unless explicitly instructed.

---

# 4. Confirmed Official Scope

The preserved project documentation records the following required/simple behavior.

## Implemented/required core functionality

- Create a fixed-size binary filesystem image.
- Initialize a superblock.
- Maintain an inode bitmap.
- Maintain a data-block bitmap.
- Maintain an inode table.
- Maintain one root directory.
- Create `.` and `..` entries.
- Add regular files to the root directory.
- Allocate inodes using first fit.
- Allocate file data blocks using first fit.
- Store file contents in data blocks.
- Create/update inode metadata.
- Add directory entries.
- Update the root inode's size.
- Reject invalid/unsupported inputs according to the supplied project logic.

## Explicit simplifications / non-features

The preserved specification says SimpleFS does **not** implement:

- subdirectories
- indirect block pointers
- file deletion
- a separate file-read command
- file renaming
- hard links
- symbolic links
- Unix permissions
- journaling
- checksums
- mounting
- caching
- multi-level directory traversal

Only the root directory exists. Source files are expected in the current working directory.

---

# 5. Filesystem Constants and Important Numbers

| Property | Value |
|---|---:|
| Block size | 4096 bytes |
| Total blocks | 64 |
| Total filesystem image size | 262144 bytes |
| Total inodes | 32 |
| Data blocks | 60 |
| Inode size | 128 bytes |
| Maximum direct blocks per inode | 3 |
| Maximum regular-file size | 12288 bytes |
| Root inode | 1 |
| Root data block | 4 |
| Directory-entry size | 64 bytes |
| Maximum filename length | 58 characters |
| Magic number | `0x53465331` |
| Regular file type | 1 |
| Directory type | 2 |

Calculations:

```text
64 blocks × 4096 bytes = 262144 bytes = 256 KiB

32 inodes × 128 bytes = 4096 bytes
Therefore the complete inode table fits exactly in one block.

3 direct blocks × 4096 bytes = 12288 bytes
Therefore the maximum regular-file size is 12288 bytes.
```

The filename field is `name[59]`, so one byte must be reserved for `\0`, giving a maximum filename length of 58 characters.

---

# 6. Fixed Disk Layout

SimpleFS uses a fixed block layout:

| Block(s) | Region | Starting byte |
|---:|---|---:|
| 0 | Superblock | 0 |
| 1 | Inode bitmap | 4096 |
| 2 | Data bitmap | 8192 |
| 3 | Inode table | 12288 |
| 4 | Root directory data | 16384 |
| 5–63 | Remaining data blocks | 20480 onward |

Graphical view:

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

Block offset formula:

```text
starting_byte = block_number × BLOCK_SIZE
```

Examples:

```text
Block 4 = 4 × 4096 = 16384
Block 5 = 5 × 4096 = 20480
Block 6 = 6 × 4096 = 24576
Block 7 = 7 × 4096 = 28672
Block 8 = 8 × 4096 = 32768
```

---

# 7. Inode Offsets

The inode table begins at byte 12288.

Formula:

```text
absolute_inode_byte = 12288 + (inode_number - 1) × 128
```

Useful offsets:

| Inode | Byte offset |
|---:|---:|
| 1 | 12288 |
| 2 | 12416 |
| 3 | 12544 |
| 4 | 12672 |
| 5 | 12800 |
| 6 | 12928 |
| 32 | 16256 |

---

# 8. Core Data Structures

The preserved documentation records these core structures:

```c
typedef struct {
    uint32_t magic, block_size, total_blocks, inode_count;
    uint32_t inode_bitmap_block, data_bitmap_block, inode_table_block, data_region_block;
    uint32_t root_inode;
} superblock_t;

typedef struct {
    uint16_t type, links;
    uint32_t size;
    uint32_t direct[3];
    uint8_t reserved[108];
} inode_t;

typedef struct {
    uint32_t inode_no;
    uint8_t type;
    char name[59];
} dirent_t;
```

Conceptually:

- `superblock_t` tells the program how the filesystem is laid out.
- `inode_t` describes a file/directory and points to data blocks.
- `dirent_t` maps a filename to an inode number and type.

---

# 9. Bitmap Mapping

## Inode bitmap

Bitmap indexing starts at zero, while inode numbering starts at one:

```text
inode bitmap bit 0 -> inode 1
inode bitmap bit 1 -> inode 2
...
inode bitmap bit 31 -> inode 32
```

Therefore:

```text
inode_number = bitmap_index + 1
bitmap_index = inode_number - 1
```

## Data bitmap

The data bitmap is relative to the data region:

```text
data bitmap bit 0 -> absolute Block 4
data bitmap bit 1 -> absolute Block 5
data bitmap bit 2 -> absolute Block 6
...
```

The inode's `direct[]` pointers store **absolute block numbers**, not bitmap indexes.

Example:

```text
data bitmap index 1 = Block 5
direct[0] = 5
```

---

# 10. Allocation Strategy

SimpleFS uses **first-fit allocation**.

For an inode:

1. Scan the inode bitmap from the beginning of the allocatable range.
2. Choose the first clear bit.
3. Convert that bitmap index to an inode number.

For a data block:

1. Scan the data bitmap from index 0.
2. Choose the first clear bit.
3. Convert to an absolute block number by adding `DATA_REGION_BLOCK`.

The data scan starts at index 0. In a valid filesystem, index 0 is already occupied by the root directory (Block 4), so the first user-file data block becomes Block 5.

This detail was specifically corrected during development; see the troubleshooting section later in this file.

---

# 11. Builder Program — Purpose and Flow

`simplefs_builder.c` creates a completely new filesystem image.

High-level flow:

```text
Create/open image
      ↓
Zero-fill all 64 blocks
      ↓
Initialize superblock
      ↓
Initialize inode bitmap
      ↓
Mark root inode allocated
      ↓
Initialize data bitmap
      ↓
Mark root data block allocated
      ↓
Initialize root inode
      ↓
Create "." entry
      ↓
Create ".." entry
      ↓
Write metadata/data to fixed offsets
      ↓
Close image
```

The builder completed TODO 1 through TODO 6.

---

# 12. Builder TODO Implementations

## Builder TODO 1 — Superblock

```c
sb.magic = MAGIC_NUMBER;
sb.block_size = BLOCK_SIZE;
sb.total_blocks = TOTAL_BLOCKS;
sb.inode_count = TOTAL_INODES;
sb.inode_bitmap_block = INODE_BITMAP_BLOCK;
sb.data_bitmap_block = DATA_BITMAP_BLOCK;
sb.inode_table_block = INODE_TABLE_BLOCK;
sb.data_region_block = DATA_REGION_BLOCK;
sb.root_inode = ROOT_INODE;
```

## Builder TODO 2 — Mark root inode used

```c
set_bit(inode_bitmap, ROOT_INODE - 1);
```

Since `ROOT_INODE = 1`, bitmap index 0 is set.

## Builder TODO 3 — Mark root data block used

```c
set_bit(data_bitmap, ROOT_DATA_BLOCK - DATA_REGION_BLOCK);
```

Since both are Block 4, this marks data bitmap index 0.

## Builder TODO 4 — Root inode

```c
root_inode.type = TYPE_DIRECTORY;
root_inode.links = 2;
root_inode.size = 2 * DIRENT_SIZE;
root_inode.direct[0] = ROOT_DATA_BLOCK;
root_inode.direct[1] = 0;
root_inode.direct[2] = 0;
```

Initial root size:

```text
2 entries × 64 bytes = 128 bytes
```

## Builder TODO 5 — `.` entry

```c
dot.inode_no = ROOT_INODE;
dot.type = TYPE_DIRECTORY;
strcpy(dot.name, ".");
```

## Builder TODO 6 — `..` entry

```c
dotdot.inode_no = ROOT_INODE;
dotdot.type = TYPE_DIRECTORY;
strcpy(dotdot.name, "..");
```

Both point to inode 1 because root is its own parent in this simplified filesystem.

---

# 13. Builder Compilation and Initial Verification

Compile:

```bash
gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
```

Create image:

```bash
./simplefs_builder --image disk.img
```

Expected/observed success:

```text
SimpleFS image created successfully: disk.img
```

Verify image size:

```bash
stat -c %s disk.img
```

Observed:

```text
262144
```

Verify superblock:

```bash
od -An -t u4 -N 36 disk.img
```

Observed:

```text
1397117745 4096 64 32 1 2 3 4 1
```

`1397117745` decimal corresponds to `0x53465331`.

Initial inode bitmap:

```bash
od -An -t u1 -j 4096 -N 1 disk.img
```

Observed:

```text
1
```

Initial data bitmap:

```bash
od -An -t u1 -j 8192 -N 1 disk.img
```

Observed:

```text
1
```

Root inode type/link count:

```bash
od -An -t u2 -j 12288 -N 4 disk.img
```

Observed:

```text
2 2
```

Root inode size:

```bash
od -An -t u4 -j 12292 -N 4 disk.img
```

Observed:

```text
128
```

Root direct pointers:

```bash
od -An -t u4 -j 12296 -N 12 disk.img
```

Observed:

```text
4 0 0
```

Raw `.` entry:

```bash
od -An -t u1 -j 16384 -N 7 disk.img
```

Observed:

```text
1 0 0 0 2 46 0
```

Raw `..` entry:

```bash
od -An -t u1 -j 16448 -N 8 disk.img
```

Observed:

```text
1 0 0 0 2 46 46 0
```

ASCII 46 is `.`.

---

# 14. Adder Program — Purpose and Flow

`simplefs_adder.c` inserts one regular host file into an existing SimpleFS image.

High-level flow:

```text
Open existing image
      ↓
Validate image / magic
      ↓
Open source host file
      ↓
Validate name and size
      ↓
Reject duplicate filename if present
      ↓
Find free inode
      ↓
Find free root-directory slot
      ↓
Calculate required blocks
      ↓
Allocate data blocks first-fit
      ↓
Copy source bytes into blocks
      ↓
Build new inode
      ↓
Mark inode bitmap
      ↓
Create new directory entry
      ↓
Update root inode size
      ↓
Write updated metadata
      ↓
Close files
```

The adder completed TODO 1 through TODO 11.

---

# 15. Adder TODO Implementations

## Adder TODO 1 — Find free inode

```c
for (int i = 1; i < TOTAL_INODES; i++) {
    if (!is_bit_set(bitmap, i)) {
        return i + 1;
    }
}
```

The scan starts at bitmap index 1 because index 0 is reserved for root inode 1.

## Adder TODO 2 — Find free data block

```c
for (int i = 0; i < DATA_BLOCKS; i++) {
    if (!is_bit_set(bitmap, i)) {
        return DATA_REGION_BLOCK + i;
    }
}
```

Strict first-fit begins at bitmap index 0.

## Adder TODO 3 — Duplicate filename check

```c
fseek(image, ROOT_DATA_BLOCK * BLOCK_SIZE, SEEK_SET);

for (int i = 0; i < BLOCK_SIZE / (int)sizeof(dirent_t); i++) {
    if (fread(&entry, sizeof(entry), 1, image) != 1) {
        break;
    }

    if (entry.inode_no != 0 && strcmp(entry.name, filename) == 0) {
        return 1;
    }
}
```

## Adder TODO 4 — Find free directory entry

```c
fseek(image,
      (ROOT_DATA_BLOCK * BLOCK_SIZE) + (2 * sizeof(dirent_t)),
      SEEK_SET);

for (int i = 2; i < BLOCK_SIZE / (int)sizeof(dirent_t); i++) {
    if (fread(&entry, sizeof(entry), 1, image) != 1) {
        break;
    }

    if (entry.inode_no == 0) {
        return i;
    }
}
```

Search starts at entry 2 because entries 0 and 1 are `.` and `..`.

## Adder TODO 5 — Filename limit and block count

```c
if (strlen(source_name) > 58) {
    printf("Error: file name is too long.\n");
    fclose(source);
    fclose(image);
    return 1;
}

required_blocks = (file_size == 0) ? 0
                  : (int)((file_size + BLOCK_SIZE - 1) / BLOCK_SIZE);
```

A zero-byte file uses zero data blocks.

For positive sizes, ceiling division determines how many 4096-byte blocks are required.

## Adder TODO 6 — Allocate data blocks in memory

```c
for (int i = 0; i < required_blocks; i++) {
    int block = find_free_data_block(data_bitmap);

    if (block == -1) {
        printf("Error: not enough free data blocks.\n");
        fclose(source);
        fclose(image);
        return 1;
    }

    allocated_blocks[i] = block;
    set_bit(data_bitmap, data_bitmap_index(block));
}
```

Each chosen block is immediately marked in the in-memory bitmap so the next allocation cannot return the same block.

## Adder TODO 7 — Copy file content

```c
for (int i = 0; i < required_blocks; i++) {
    unsigned char buffer[BLOCK_SIZE] = {0};

    size_t bytes_read = fread(buffer, 1, BLOCK_SIZE, source);

    if (ferror(source)) {
        printf("Error: could not read source file.\n");
        fclose(source);
        fclose(image);
        return 1;
    }

    fseek(image, (long)allocated_blocks[i] * BLOCK_SIZE, SEEK_SET);

    if (fwrite(buffer, BLOCK_SIZE, 1, image) != 1) {
        printf("Error: could not write file data.\n");
        fclose(source);
        fclose(image);
        return 1;
    }

    (void)bytes_read;
}
```

The buffer is zero-initialized every time. If the final source read is shorter than 4096 bytes, unused bytes in that allocated block remain zero.

## Adder TODO 8 — New inode

```c
new_inode.type = TYPE_FILE;
new_inode.links = 1;
new_inode.size = (uint32_t)file_size;

for (int i = 0; i < required_blocks; i++) {
    new_inode.direct[i] = allocated_blocks[i];
}
```

Unused direct pointers remain zero because `new_inode` was zeroed before this code.

## Adder TODO 9 — Mark inode allocated

```c
set_bit(inode_bitmap, free_inode - 1);
```

## Adder TODO 10 — New directory entry

```c
new_entry.inode_no = free_inode;
new_entry.type = TYPE_FILE;
strncpy(new_entry.name, source_name, sizeof(new_entry.name) - 1);
new_entry.name[sizeof(new_entry.name) - 1] = '\0';
```

## Adder TODO 11 — Update root size

```c
root_inode.size += sizeof(dirent_t);
```

Every successful regular-file insertion adds one 64-byte root directory entry.

---

# 16. Sample Files Used During Testing

Three supplied sample files were used during manual testing:

```text
test1.txt = 22 bytes
test2.txt = 77 bytes
test3.txt = 23 bytes
```

All three fit in a single 4096-byte data block.

The test files were part of starter/testing materials. They were not required inside the final official four-file submission ZIP.

---

# 17. Manual Test Sequence and Results

## 17.1 `test1.txt`

Add:

```bash
./simplefs_adder --input disk.img --file test1.txt
```

Observed:

```text
test1.txt added successfully to disk.img
```

Inode bitmap:

```text
3
```

Binary `00000011`: inode 1 root + inode 2 file.

Data bitmap:

```text
3
```

Block 4 root + Block 5 `test1.txt`.

Inode 2:

```text
type/links = 1 1
size = 22
direct = 5 0 0
```

Root size:

```text
192
```

Calculation:

```text
128 initial + 64 file entry = 192
```

Raw first user directory entry began at byte 16512.

Actual content verification:

```bash
dd if=disk.img bs=1 skip=20480 count=22 status=none | cmp - test1.txt && echo MATCH
```

Observed:

```text
MATCH
```

## 17.2 Duplicate rejection

Attempted to add `test1.txt` again:

```bash
./simplefs_adder --input disk.img --file test1.txt
```

Observed:

```text
Error: file already exists in SimpleFS.
```

Bitmaps remained unchanged at 3/3.

## 17.3 `test2.txt`

Added successfully.

Observed:

```text
inode bitmap = 7
data bitmap = 7
inode 3 type/links = 1 1
size = 77
direct = 6 0 0
root size = 256
```

Content verification:

```text
MATCH
```

## 17.4 `test3.txt`

Added successfully.

Observed:

```text
inode bitmap = 15
data bitmap = 15
inode 4 type/links = 1 1
size = 23
direct = 7 0 0
root size = 320
```

Content verification:

```text
MATCH
```

At this point the root directory logically contained:

```text
.
..
test1.txt
test2.txt
test3.txt
```

Five entries × 64 bytes = 320 bytes.

---

# 18. Zero-Byte File Test

Created:

```bash
: > empty.txt
```

Verified size:

```text
0
```

Added:

```bash
./simplefs_adder --input disk.img --file empty.txt
```

Observed success.

After insertion:

```text
inode bitmap = 31
data bitmap = 15
file size = 0
direct pointers = 0 0 0
```

This proves a zero-byte file:

- gets an inode
- gets a directory entry
- does not consume a data block
- has zero direct pointers

---

# 19. Exact Maximum-Size File Test

Created a 12288-byte file:

```bash
dd if=/dev/zero of=maxfile.bin bs=12288 count=1 status=none
```

Added successfully:

```bash
./simplefs_adder --input disk.img --file maxfile.bin
```

Observed direct pointers:

```text
8 9 10
```

This proves use of all three direct block pointers.

Content verification:

```bash
dd if=disk.img bs=1 skip=32768 count=12288 status=none | cmp - maxfile.bin && echo MATCH
```

Observed:

```text
MATCH
```

---

# 20. Oversized File Rejection

Created a file exactly one byte too large:

```bash
dd if=/dev/zero of=toolarge.bin bs=12289 count=1 status=none
```

Attempted insertion:

```bash
./simplefs_adder --input disk.img --file toolarge.bin
```

Observed:

```text
Error: file is too large for SimpleFS.
```

The inode and data bitmaps remained unchanged.

This verifies the 12288-byte maximum.

---

# 21. Filename Length Boundary Test

A temporary filename of 59 characters was generated:

```bash
name=$(printf 'a%.0s' {1..59}); printf "x" > "$name"; echo ${#name}
```

Observed length:

```text
59
```

Insertion attempt:

```bash
./simplefs_adder --input disk.img --file "$name"
```

Observed:

```text
Error: file name is too long.
```

The temporary host file was then removed:

```bash
rm -- "$name"
```

No allocation state changed.

---

# 22. Final Clean Compilation

Both programs were compiled together:

```bash
gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder && \
gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder && \
echo COMPILE_OK
```

Observed:

```text
COMPILE_OK
```

No compiler warnings or errors were reported above the success message.

---

# 23. Fresh End-to-End Integration Verification

A brand-new image was created to prove the final builder and adder work together independently of the earlier test image:

```bash
./simplefs_builder --image finalcheck.img
./simplefs_adder --input finalcheck.img --file test1.txt
```

Combined diagnostics produced:

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

Conclusion:

```text
Fresh Builder + Adder integration test: PASSED
```

Temporary files were cleaned:

```bash
rm -f finalcheck.img empty.txt maxfile.bin toolarge.bin && echo CLEANUP_OK
```

Observed:

```text
CLEANUP_OK
```

---

# 24. Tests Executed vs. Not Separately Executed

## Executed and passed

- Exact 262144-byte image
- Superblock values
- Initial inode bitmap
- Initial data bitmap
- Root inode
- `.` and `..`
- One-block regular files
- Multiple regular files
- First-fit inode allocation
- First-fit data allocation
- Byte-for-byte content verification
- Duplicate filename rejection
- Zero-byte file
- 12288-byte maximum file
- 12289-byte rejection
- 59-character filename rejection
- Fresh builder + adder integration

## Not separately exercised during the documented final manual sequence

These behaviors existed in starter/final logic but were not individually forced during the documented session:

- separate 5000-byte two-block test
- missing source file
- missing image file
- invalid magic-number corruption test
- complete inode exhaustion
- complete data-block exhaustion
- complete root directory entry exhaustion
- forced low-level read/write failure

A future reviewer should not describe those as manually tested unless new evidence/tests are run.

---

# 25. Troubleshooting and Corrections During Development

## 25.1 First-fit data-block scan correction

The scan in `find_free_data_block()` was corrected to begin at bitmap index 0, not index 1.

Why:

```text
data bitmap index 0 = Block 4
```

Block 4 is already marked as occupied by root in a valid image. Therefore strict first-fit naturally sees index 0 as used and then chooses index 1 / Block 5.

## 25.2 Human transcription error: root size

A value was initially typed in conversation as `912`, but the actual terminal output was:

```text
192
```

There was no filesystem error.

## 25.3 Human transcription error: `test2.txt` direct pointer

A value was initially typed as:

```text
7 0 0
```

but the actual terminal output was:

```text
6 0 0
```

The implementation was correct; this was only a chat transcription mistake.

## 25.4 Tiny sample files

The provided test files were intentionally small:

```text
test1.txt = 22 bytes
test2.txt = 77 bytes
test3.txt = 23 bytes
```

Seeing only one/few lines in the editor was normal.

## 25.5 Temporary long filename

The long `aaaa...` file was intentionally created to test the 58-character filename limit and then deleted.

## 25.6 Codespaces was temporary

The GitHub Codespace was used as a temporary Linux/GCC development environment. Windows itself was not reinstalled, dual-booted, or modified.

The active working directory was:

```text
/workspaces/codespaces-blank/SimpleFS_Work
```

The Codespace could auto-delete after inactivity, which is why the project was downloaded/backed up elsewhere and later archived in GitHub.

---

# 26. Original Development File Handling

The original local project folder was described as:

```text
project.321/
├── Starter Files/     # untouched original copy
└── SimpleFS_Work/     # working implementation copy
```

This was done to protect the starter files while edits were made in the working copy.

Starter/testing materials included:

```text
README.txt
simplefs.h
simplefs_builder.c
simplefs_adder.c
test1.txt
test2.txt
test3.txt
```

---

# 27. Chronological Terminal History

The following history preserves the original end-to-end command sequence as recorded.

```text
1  cd /workspaces/codespaces-blank/SimpleFS_Work
2  ls
3  mv ../README.txt ../simplefs.h ../simplefs_builder.c ../simplefs_adder.c ../test1.txt ../test2.txt ../test3.txt .
4  ls
5  gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
6  ./simplefs_builder --image disk.img
7  stat -c %s disk.img
8  od -An -t u4 -N 36 disk.img
9  od -An -t u1 -j 4096 -N 1 disk.img
10 od -An -t u1 -j 8192 -N 1 disk.img
11 od -An -t u2 -j 12288 -N 4 disk.img
12 od -An -t u4 -j 12292 -N 4 disk.img
13 od -An -t u4 -j 12296 -N 12 disk.img
14 od -An -t u1 -j 16384 -N 7 disk.img
15 od -An -t u1 -j 16448 -N 8 disk.img
16 gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder
17 ./simplefs_adder --input disk.img --file test1.txt
18 od -An -t u1 -j 4096 -N 1 disk.img
19 od -An -t u1 -j 8192 -N 1 disk.img
20 od -An -t u2 -j 12416 -N 4 disk.img
21 od -An -t u4 -j 12420 -N 4 disk.img
22 od -An -t u4 -j 12424 -N 12 disk.img
23 od -An -t u4 -j 12292 -N 4 disk.img
24 od -An -t u1 -j 16512 -N 15 disk.img
25 ./simplefs_adder --input disk.img --file test1.txt
26 od -An -t u1 -j 4096 -N 1 disk.img
27 od -An -t u1 -j 8192 -N 1 disk.img
28 ./simplefs_adder --input disk.img --file test2.txt
29 od -An -t u1 -j 4096 -N 1 disk.img
30 stat -c %s test2.txt
31 od -An -t u1 -j 8192 -N 1 disk.img
32 od -An -t u2 -j 12544 -N 4 disk.img
33 od -An -t u4 -j 12548 -N 4 disk.img
34 od -An -t u4 -j 12552 -N 12 disk.img
35 od -An -t u4 -j 12292 -N 4 disk.img
36 dd if=disk.img bs=1 skip=20480 count=22 status=none | cmp - test1.txt && echo MATCH
37 dd if=disk.img bs=1 skip=24576 count=77 status=none | cmp - test2.txt && echo MATCH
38 stat -c %s test3.txt
39 ./simplefs_adder --input disk.img --file test3.txt
40 od -An -t u1 -j 4096 -N 1 disk.img
41 od -An -t u1 -j 8192 -N 1 disk.img
42 od -An -t u2 -j 12672 -N 4 disk.img
43 od -An -t u4 -j 12676 -N 4 disk.img
44 od -An -t u4 -j 12680 -N 12 disk.img
45 dd if=disk.img bs=1 skip=28672 count=23 status=none | cmp - test3.txt && echo MATCH
46 od -An -t u4 -j 12292 -N 4 disk.img
47 : > empty.txt && stat -c %s empty.txt
48 ./simplefs_adder --input disk.img --file empty.txt
49 od -An -t u1 -j 4096 -N 1 disk.img
50 od -An -t u1 -j 8192 -N 1 disk.img
51 od -An -t u4 -j 12804 -N 4 disk.img
52 od -An -t u4 -j 12808 -N 12 disk.img
53 dd if=/dev/zero of=maxfile.bin bs=12288 count=1 status=none && stat -c %s maxfile.bin
54 ./simplefs_adder --input disk.img --file maxfile.bin
55 od -An -t u1 -j 8192 -N 1 disk.img
56 od -An -t u4 -j 12936 -N 12 disk.img
57 dd if=disk.img bs=1 skip=32768 count=12288 status=none | cmp - maxfile.bin && echo MATCH
58 dd if=/dev/zero of=toolarge.bin bs=12289 count=1 status=none && stat -c %s toolarge.bin
59 ./simplefs_adder --input disk.img --file toolarge.bin
60 od -An -t u1 -j 4096 -N 1 disk.img
61 od -An -t u1 -j 8192 -N 1 disk.img
62 name=$(printf 'a%.0s' {1..59}); printf "x" > "$name"; echo ${#name}
63 ./simplefs_adder --input disk.img --file "$name"
64 rm -- "$name"
65 od -An -t u1 -j 4096 -N 1 disk.img
66 od -An -t u1 -j 8192 -N 1 disk.img
67 gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder && gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder && echo COMPILE_OK
68 ./simplefs_builder --image finalcheck.img
69 ./simplefs_adder --input finalcheck.img --file test1.txt
70 echo "image_size=$(stat -c %s finalcheck.img)"; echo "inode_bitmap=$(od -An -t u1 -j 4096 -N 1 finalcheck.img | xargs)"; echo "data_bitmap=$(od -An -t u1 -j 8192 -N 1 finalcheck.img | xargs)"; echo "root_size=$(od -An -t u4 -j 12292 -N 4 finalcheck.img | xargs)"; echo "inode2_type_links=$(od -An -t u2 -j 12416 -N 4 finalcheck.img | xargs)"; echo "inode2_size=$(od -An -t u4 -j 12420 -N 4 finalcheck.img | xargs)"; echo "inode2_ptrs=$(od -An -t u4 -j 12424 -N 12 finalcheck.img | xargs)"; dd if=finalcheck.img bs=1 skip=20480 count=22 status=none | cmp - test1.txt && echo "content=MATCH"
71 rm -f finalcheck.img empty.txt maxfile.bin toolarge.bin && echo CLEANUP_OK
72 cat README.txt
73 SimpleFS Lab Term Project - Student Starter Files
74 FILES
75 1. simplefs.h          - fixed constants and structures; do not modify unless instructed.
76 2. simplefs_builder.c  - complete TODO 1 through TODO 6.
77 3. simplefs_adder.c    - complete TODO 1 through TODO 11.
78 4. test1.txt, test2.txt, test3.txt - simple sample input files.
79 COMPILE
80 gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
81 gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder
82 RUN
83 ./simplefs_builder --image disk.img
84 ./simplefs_adder --input disk.img --file test1.txt
85 IMPORTANT
86 - Follow the official project specification for the exact layout and expected behavior.
87 for f in simplefs.h simplefs_builder.c simplefs_adder.c README.txt; do [ -f "$f" ] && echo "$f OK" || echo "$f MISSING"; done
88 rm -rf Project_Sec_7_Group_5 Project_Sec_7_Group_5.zip && mkdir Project_Sec_7_Group_5 && cp simplefs.h simplefs_builder.c simplefs_adder.c README.txt Project_Sec_7_Group_5/ && zip -r Project_Sec_7_Group_5.zip Project_Sec_7_Group_5
89 unzip -Z1 Project_Sec_7_Group_5.zip
90 history > CSE321_terminal_commands.txt
```

Note: lines 73–86 are text that appeared in shell history as part of the README/history workflow and are preserved here exactly as contextual history.

---

# 28. Final README Used for Submission

The final `README.txt` records:

```text
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
```

---

# 29. Official Submission Packaging

The official submission was intentionally kept separate from the broader GitHub archive.

Required deliverables:

```text
simplefs.h
simplefs_builder.c
simplefs_adder.c
README.txt
```

Verification command:

```bash
for f in simplefs.h simplefs_builder.c simplefs_adder.c README.txt; do
    [ -f "$f" ] && echo "$f OK" || echo "$f MISSING"
done
```

All four were reported `OK`.

Final folder:

```text
Project_Sec_7_Group_5
```

Packaging:

```bash
rm -rf Project_Sec_7_Group_5 Project_Sec_7_Group_5.zip && \
mkdir Project_Sec_7_Group_5 && \
cp simplefs.h simplefs_builder.c simplefs_adder.c README.txt Project_Sec_7_Group_5/ && \
zip -r Project_Sec_7_Group_5.zip Project_Sec_7_Group_5
```

ZIP inspection:

```bash
unzip -Z1 Project_Sec_7_Group_5.zip
```

Verified archive contents:

```text
Project_Sec_7_Group_5/
Project_Sec_7_Group_5/simplefs.h
Project_Sec_7_Group_5/simplefs_builder.c
Project_Sec_7_Group_5/simplefs_adder.c
Project_Sec_7_Group_5/README.txt
```

No executables, disk images, test files, or temporary generated files were included in the official submission ZIP.

The ZIP was downloaded to Windows and later opened in Google Drive, where the same four items were visible.

---

# 30. Difference Between Official Submission and GitHub Archive

This distinction matters.

## Official course submission

Only the required four project files were packaged:

```text
README.txt
simplefs.h
simplefs_builder.c
simplefs_adder.c
```

## Later GitHub archive

The repository is broader and exists to preserve implementation + process + evidence.

Before adding the two new archive files described in this master record, the GitHub repository contained:

```text
1. CSE321_SimpleFS_Complete_Project_Documentation_Group5.docx
2. CSE321_terminal_commands.txt
3. README.md
4. README.txt
5. simplefs.h
6. simplefs_adder.c
7. simplefs_builder.c
```

The intended additional archive files are:

```text
8. 22299368.patch
9. CSE321_SimpleFS_MASTER_PROJECT_RECORD.md
```

After those are added, the repository will contain nine principal archive files.

The `.patch` file is **not an official submission requirement**. It is intentionally being added as an archival/development artifact.

This master record is also **not an official submission requirement**. It exists so future work can be resumed without relying on a previous ChatGPT conversation.

---

# 31. Purpose of the Patch File

Intended archive file:

```text
22299368.patch
```

The patch file is a record of source-code changes/differences associated with the project/student implementation.

Why preserve it:

- gives historical evidence of code modifications
- can help compare starter code vs. completed code if the patch contains those differences
- can be inspected by a future developer/reviewer
- provides another recovery artifact if the source history is needed

A future AI assistant should not treat a patch as a replacement for the actual final `.c` and `.h` files. The final source files remain the main source of truth for the completed implementation.

---

# 32. GitHub README vs. Submission README

The repository intentionally contains both:

## `README.txt`

The submission-oriented README.

It is short, plain-text, and matches the official course deliverable style.

## `README.md`

The GitHub-facing README.

It is more detailed and explains:

- project information
- filesystem layout
- structures
- builder behavior
- adder behavior
- allocation strategy
- testing
- boundary cases
- final result
- repository purpose

This is not duplication by mistake. The two files serve different purposes.

---

# 33. Full Documentation File

The repository also contains:

```text
CSE321_SimpleFS_Complete_Project_Documentation_Group5.docx
```

That document is a detailed formal record containing:

- final package confirmation
- reconstructed/official project requirements
- local setup
- Codespaces environment
- architecture/calculations
- Builder TODOs
- Builder verification
- Adder TODOs
- Adder testing
- fresh integration test
- cleanup
- final README
- ZIP packaging
- Google Drive verification
- chronological command reference
- test coverage matrix
- troubleshooting corrections
- viva guide
- final checklist
- appendices

This master Markdown file intentionally overlaps some of that content because its purpose is portability and future AI/context recovery.

---

# 34. Terminal Command History File

The repository contains:

```text
CSE321_terminal_commands.txt
```

This is the raw command-history/evidence file.

Use it when exact terminal commands or sequence are important.

The master record summarizes and reproduces the important chronology, but `CSE321_terminal_commands.txt` remains the preferred raw-history artifact.

---

# 35. GitHub Repository Ownership and Privacy History

The repository was originally under the personal GitHub account:

```text
anik-rahman/CSE321-Operating-System-SimpleFS
```

The user wanted the repository to remain private while allowing project groupmates to see the repository without being able to modify it.

A personal private repository did not provide the desired read-only collaborator model in the chosen setup, so an organization was created:

```text
anik-cse321
```

The repository was transferred to:

```text
anik-cse321/CSE321-Operating-System-SimpleFS
```

The repository remained:

```text
Private
```

The organization owner remained Anik.

Organization invitations were sent to:

- MD. Shahriar Saif
- Ayasha Islam

Their organization role was set to:

```text
Member
```

The organization base repository permission was left at:

```text
Read
```

With that base permission, accepted organization members can read private repositories to which the base permission applies but cannot push code with read-only repository permission.

At the time of the archive setup discussion, both invitations were still shown as pending.

**Future note:** If organization permissions are later changed, inspect GitHub's current `Settings -> Member privileges` and repository access pages rather than assuming this August 2026 state remains unchanged.

---

# 36. What Groupmates Can See After Read Access

Once a groupmate accepts organization membership and has read access, they can generally:

- open the private repository
- browse source files
- read README files
- read this master record
- view documentation committed to the repository
- inspect commit history
- clone the repository
- pull newer commits
- download repository content

They cannot push changes to the organization repository with only `Read` permission.

If Anik modifies a file and commits/pushes it to the repository, members with read access can see the updated version after refreshing GitHub or pulling the latest commits.

A local edit on Anik's computer is not visible to them until it is uploaded/committed/pushed.

---

# 37. Recommended Nine-File Archive Layout

After adding the patch and this master file, the clean archive should look approximately like:

```text
CSE321-Operating-System-SimpleFS/
├── README.md
├── README.txt
├── simplefs.h
├── simplefs_builder.c
├── simplefs_adder.c
├── CSE321_terminal_commands.txt
├── CSE321_SimpleFS_Complete_Project_Documentation_Group5.docx
├── 22299368.patch
└── CSE321_SimpleFS_MASTER_PROJECT_RECORD.md
```

This is an archive layout, not the original submission layout.

Generated files such as the following do not need to be committed just to preserve the project:

```text
simplefs_builder
simplefs_adder
disk.img
finalcheck.img
empty.txt
maxfile.bin
toolarge.bin
```

They can be recreated using the source and commands when needed.

---

# 38. Important Source-of-Truth Priority for Future Work

If any future document or AI response conflicts with another, use this priority:

1. **Actual current source files**
   - `simplefs.h`
   - `simplefs_builder.c`
   - `simplefs_adder.c`

2. **Official/final submission README**
   - `README.txt`

3. **Raw command/testing evidence**
   - `CSE321_terminal_commands.txt`

4. **Complete project documentation**
   - `CSE321_SimpleFS_Complete_Project_Documentation_Group5.docx`

5. **Patch/history artifact**
   - `22299368.patch`

6. **GitHub presentation README**
   - `README.md`

7. **This master context file**
   - `CSE321_SimpleFS_MASTER_PROJECT_RECORD.md`

This master record is intended to connect all sources, not override the actual code.

---

# 39. What a Future ChatGPT Should Do

If this project is reopened in a new conversation, provide this master record plus the current code files.

A good instruction is:

```text
This is my CSE321 Operating Systems SimpleFS project archive.
Read CSE321_SimpleFS_MASTER_PROJECT_RECORD.md first.
Then inspect simplefs.h, simplefs_builder.c, and simplefs_adder.c as the source of truth.
Use README.txt and CSE321_terminal_commands.txt to verify original submission behavior and testing.
Do not invent the lost Google Form wording.
If something conflicts, trust the current source code and clearly state the conflict.
```

A future AI should be able to continue tasks such as:

- explain every line/function of the C code
- reconstruct the assignment scope
- prepare viva questions
- verify filesystem calculations
- re-run tests
- compare patch vs. final code
- improve documentation
- create diagrams
- prepare presentation slides
- explain shell commands
- identify potential bugs or edge cases
- reproduce the official submission package
- explain first-fit allocation and bitmap logic

---

# 40. Viva/Understanding Quick Reference

## What is SimpleFS?

A small educational fixed-layout filesystem stored in a single binary image.

## Why 262144 bytes?

```text
64 × 4096 = 262144
```

## Superblock location?

Block 0, byte 0.

## Inode bitmap location?

Block 1, byte 4096.

## Data bitmap location?

Block 2, byte 8192.

## Inode table location?

Block 3, byte 12288.

## Why does the inode table exactly fit Block 3?

```text
32 × 128 = 4096
```

## Data region start?

Block 4, byte 16384.

## Root inode?

Inode 1.

## Root data block?

Block 4.

## Initial root size?

```text
2 × 64 = 128 bytes
```

for `.` and `..`.

## Why max filename 58?

`char name[59]` requires one byte for `\0`.

## Why max file size 12288?

Three direct pointers × 4096 bytes.

## What is first-fit?

Choose the first free allocation bit from the start of the bitmap.

## Why can an empty file use no data blocks?

It has metadata/name but no file content.

## Why zero-fill data buffers?

To ensure unused bytes in a partially filled final block remain zero.

## Why does test1 use Block 5?

Block 4 belongs to root; first-fit chooses the next free block.

## Bitmap value 3 after test1?

Binary `00000011`.

For inode bitmap: inode 1 + inode 2.  
For data bitmap: Block 4 + Block 5.

## Bitmap value 127 after maxfile?

Binary `01111111`; data bitmap indexes 0–6 are occupied, corresponding to Blocks 4–10.

---

# 41. Final Project Status

Based on the preserved implementation and testing record:

```text
simplefs.h fixed constants/structures: PASS
Builder TODO 1–6: PASS
Adder TODO 1–11: PASS
Clean compilation: PASS
Image size: PASS
Superblock: PASS
Initial bitmaps: PASS
Root inode: PASS
. and .. entries: PASS
Regular file insertion: PASS
First-fit allocation: PASS
Byte-for-byte content verification: PASS
Duplicate rejection: PASS
Zero-byte file: PASS
12288-byte max file: PASS
12289-byte rejection: PASS
59-character filename rejection: PASS
Fresh end-to-end test: PASS
README: PASS
Official ZIP: PASS
Official ZIP exact four deliverables: PASS
Google Drive verification: PASS
GitHub archive: CREATED
Private organization repository: CREATED
Patch archive addition: PLANNED/TO BE COMMITTED
Master context file addition: THIS FILE
```

Overall implementation status:

```text
COMPLETE
```

Official submission packaging status:

```text
COMPLETE AND VERIFIED
```

Archive/documentation status after committing this file and the patch:

```text
COMPLETE LONG-TERM PROJECT ARCHIVE
```

---

# 42. Final Notes

- The exact Google Form question is no longer available.
- Do not pretend that its verbatim wording has been recovered.
- The preserved materials are sufficient to understand and reconstruct the project technically.
- The actual C source files remain the authoritative implementation.
- The official course submission and the later GitHub archive have different purposes.
- The GitHub archive intentionally contains more files than the four-file course submission.
- Keep the repository private unless there is a deliberate future reason to change visibility.
- When editing this project in the future, commit meaningful changes so groupmates/read-only members can see the newest version.
- If tests are re-run or new bugs are found, update this master record rather than relying only on chat history.

---

# 43. Archive Manifest Summary

**Core implementation**
- `simplefs.h`
- `simplefs_builder.c`
- `simplefs_adder.c`

**Official submission documentation**
- `README.txt`

**GitHub-facing documentation**
- `README.md`

**Raw process evidence**
- `CSE321_terminal_commands.txt`

**Formal full documentation**
- `CSE321_SimpleFS_Complete_Project_Documentation_Group5.docx`

**Code-change history**
- `22299368.patch`

**Long-term recovery/context file**
- `CSE321_SimpleFS_MASTER_PROJECT_RECORD.md`

---

**END OF MASTER PROJECT RECORD**
