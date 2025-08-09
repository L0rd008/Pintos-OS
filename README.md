# Pintos Operating System Project

**Author:** Psindu Senevirathne  
**Course:** Operating Systems  
**Institution:** University of Moratuwa  
**Semester:** Third Semester  
**Completion Date:** November 2024

## Overview

This repository contains my implementation of the Pintos operating system framework for the 80x86 architecture. Pintos is a simple educational operating system that supports kernel threads, loading and running user programs, and a basic file system. This project was completed individually as part of the Operating Systems course at the University of Moratuwa.

**Completed Projects:**
- ✅ Project 1: Threads
- ✅ Project 2: User Programs
- ❌ Project 3: Virtual Memory (Not implemented)
- ❌ Project 4: File Systems (Not implemented)

## Project Structure

```
src/
├── threads/          # Core kernel and thread management (Project 1)
├── userprog/         # User program loader and system calls (Project 2)
├── devices/          # I/O device interfacing
├── lib/              # Standard C library implementation
├── tests/            # Test suites for each project
├── examples/         # Example user programs
├── filesys/          # Basic file system (used but not modified)
├── vm/               # Virtual memory (placeholder)
└── utils/            # Utilities and helper scripts
```

## Project 1: Threads

### Overview
Extended the functionality of Pintos' minimally functional thread system to gain a better understanding of synchronization problems and thread scheduling.

### Key Implementations

#### 1. Alarm Clock (`timer_sleep()`)
- **Problem:** Original implementation used busy waiting (spinning in a loop)
- **Solution:** Reimplemented to avoid busy waiting by putting threads to sleep and waking them up when their time expires
- **Files Modified:** `devices/timer.c`

#### 2. Priority Scheduling
- **Features Implemented:**
  - Priority-based thread scheduling (priorities 0-63, higher number = higher priority)
  - Immediate preemption when higher priority thread becomes ready
  - Priority donation to solve priority inversion problems
  - Nested priority donation support
  - Priority scheduling for synchronization primitives (locks, semaphores, condition variables)

- **Key Functions:**
  - `thread_set_priority(int new_priority)` - Sets current thread's priority
  - `thread_get_priority(void)` - Returns current thread's priority (including donated priority)

- **Files Modified:** `threads/thread.c`, `threads/thread.h`, `threads/synch.c`

#### 3. Advanced Scheduler (4.4BSD MLFQS)
- **Implementation:** Multi-Level Feedback Queue Scheduler similar to 4.4BSD
- **Features:**
  - Dynamic priority calculation based on recent CPU usage and niceness
  - Load average tracking
  - Fixed-point arithmetic for calculations
  - Enabled with `-mlfqs` kernel option

- **Files Added:** `threads/fixed-point.h`
- **Files Modified:** `threads/thread.c`, `threads/thread.h`

### Testing
All Project 1 tests pass:
- Alarm clock tests
- Priority scheduling tests  
- Priority donation tests
- MLFQS tests

## Project 2: User Programs

### Overview
Enabled Pintos to run user programs by implementing argument passing, system calls, and proper process management. This project builds the foundation for user-kernel interaction.

### Key Implementations

#### 1. Argument Passing
- **Problem:** `process_execute()` didn't support command-line arguments
- **Solution:** Extended to parse command lines and set up user stack properly
- **Features:**
  - Parses command line at spaces
  - Handles multiple spaces between arguments
  - Sets up stack according to 80x86 calling convention

#### 2. System Call Handler
Implemented comprehensive system call interface in `userprog/syscall.c`:

**Process Control:**
- `halt()` - Terminates Pintos
- `exit(int status)` - Terminates current user program
- `exec(const char *cmd_line)` - Executes a new program
- `wait(pid_t pid)` - Waits for child process to terminate

**File System Calls:**
- `create(const char *file, unsigned initial_size)` - Creates a new file
- `remove(const char *file)` - Deletes a file
- `open(const char *file)` - Opens a file and returns file descriptor
- `filesize(int fd)` - Returns file size
- `read(int fd, void *buffer, unsigned size)` - Reads from file/stdin
- `write(int fd, const void *buffer, unsigned size)` - Writes to file/stdout
- `seek(int fd, unsigned position)` - Changes file position
- `tell(int fd)` - Returns current file position
- `close(int fd)` - Closes file descriptor

#### 3. Process Management
- **Process Loading:** Enhanced ELF binary loading and process creation
- **Memory Protection:** Proper validation of user memory accesses
- **Process Termination:** Clean resource cleanup and parent notification
- **Synchronization:** Thread-safe system call implementation

#### 4. File Descriptor Management
- **Per-process file descriptor table**
- **Standard I/O support (stdin/stdout)**
- **Proper file descriptor inheritance and cleanup**

### Files Modified
- `userprog/syscall.c` - System call handler implementation
- `userprog/process.c` - Process loading and management
- `userprog/exception.c` - Exception handling for invalid memory access
- `threads/thread.h` - Added process-related thread fields

### Testing
All Project 2 tests pass:
- Argument passing tests
- System call functionality tests
- Process management tests
- Memory protection tests

## Building and Running

### Prerequisites
- GCC cross-compiler for i386
- Bochs or QEMU simulator
- Make build system

### Build Instructions

1. **Build Project 1 (Threads):**
   ```bash
   cd src/threads
   make
   ```

2. **Build Project 2 (User Programs):**
   ```bash
   cd src/userprog
   make
   ```

### Running Tests

1. **Run specific test:**
   ```bash
   cd src/threads/build
   pintos -- run alarm-multiple
   
   cd src/userprog/build
   pintos --filesys-size=2 -p ../../examples/echo -a echo -- run 'echo x'
   ```

2. **Run all tests:**
   ```bash
   cd src/threads
   make check
   
   cd src/userprog
   make check
   ```

### Debugging
- Use GDB for kernel debugging: `pintos-gdb kernel.o`
- Enable debug output: `pintos -v -- run test-name`
- Check backtraces for kernel panics

## Technical Details

### Thread System Architecture
- **Thread Structure:** Extended `struct thread` with priority donation fields, sleep timing, and MLFQS statistics
- **Scheduler:** Implements both priority scheduling and MLFQS with proper preemption
- **Synchronization:** Priority-aware locks, semaphores, and condition variables

### System Call Architecture  
- **User Memory Access:** Safe validation and access to user virtual memory
- **File System Integration:** Thread-safe interface to existing file system
- **Process Control Block:** Maintains parent-child relationships and exit status

### Memory Management
- **Stack Setup:** Proper argument placement following 80x86 conventions
- **Memory Protection:** Validates all user memory accesses
- **Resource Cleanup:** Ensures proper cleanup of process resources

## Design Decisions

1. **Priority Donation:** Implemented with depth limit of 8 levels to prevent infinite chains
2. **File System Synchronization:** Used global file system lock for simplicity and safety
3. **Process Wait:** Implemented using semaphores for efficient parent-child synchronization
4. **Fixed-Point Arithmetic:** Custom implementation for MLFQS calculations

## Challenges Overcome

1. **Priority Inversion:** Solved through comprehensive priority donation system
2. **Race Conditions:** Careful synchronization in system calls and process management
3. **Memory Safety:** Robust user memory validation to prevent kernel corruption
4. **Stack Setup:** Proper implementation of 80x86 calling convention for argument passing

## Testing Results

- **Project 1:** All 27 tests pass
- **Project 2:** All 76 tests pass
- **Total Test Coverage:** 100% for implemented projects

## Future Work

The foundation laid in these two projects provides a solid base for implementing:
- **Virtual Memory (Project 3):** Paging, memory mapping, and swap
- **File Systems (Project 4):** Subdirectories, extensible files, and buffer cache

## References

- [Pintos Documentation](https://web.stanford.edu/class/cs140/projects/pintos/pintos.html)
- [80x86 Calling Convention](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)
- [4.4BSD Scheduler](https://docs.freebsd.org/en/books/design-44bsd/scheduler.html)

## License

This project is distributed under the same license as Pintos. Students own the code they write and may use it for any purpose. See the LICENSE file for details.

---

*This implementation demonstrates understanding of operating system fundamentals including thread scheduling, synchronization, system calls, and process management. The code successfully passes all test suites and provides a solid foundation for advanced OS concepts.*