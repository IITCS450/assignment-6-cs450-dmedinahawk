# Assignment 6 – Symbolic Links 
# Destiny Medina
# Cs450-01

This assignment extends xv6 to support symbolic links by introducing a new inode type, implementing a system call, and modifying path resolution in the file system.

A new inode type `T_SYMLINK` was added in `fs.h` and `stat.h` to represent symbolic links. Unlike regular files, a symlink inode stores a NUL-terminated string containing the target path in its data blocks. This design aligns with how metadata is persisted in xv6’s on-disk inode structure.

The system call `symlink(const char *target, const char *linkpath)` was implemented to create symbolic links. Internally, it uses `create()` to allocate a new inode of type `T_SYMLINK`, then writes the target path into the inode using `writei()`. The syscall is integrated through `syscall.h`, `user.h`, `usys.S`, and `syscall.c`. It returns `0` on success and `-1` on failure.

To support symlink resolution, `sys_open()` was modified. When opening a path, if the resolved inode is of type `T_SYMLINK`, the kernel reads the stored target path using `readi()` and continues resolution via `namei()`. This process supports chained symbolic links. To prevent infinite resolution loops, a maximum depth limit (`MAXSYMLINK = 10`) is enforced. If the limit is exceeded, `open()` fails and returns an error.

Correctness was validated using the provided `testsymlink.c`. The test verifies that, symbolic links can be created successfully, reading through a symbolic link returns the target file’s contents, and cyclic symbolic links are detected and fail due to the depth limit.

The observed output was:

PASS: symlink created  
PASS: read through symlink  
PASS: loop detected (open failed)  
testsymlink done  

This demonstrates correct behavior for symlink creation, resolution, and loop prevention.

## Limitations
- Maximum symlink resolution depth is limited to 10
- Maximum target path length is bounded by a single block (`BSIZE`)