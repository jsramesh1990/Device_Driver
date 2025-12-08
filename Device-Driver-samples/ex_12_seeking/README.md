# seeking

Seeking method is used to reposition read/write offset of the opened file.
Obviously, not all the char devices need to implement this method. For example,
the keyboard device is one of this sort which can not be repositioned. The
seeking method is easy to implement, driver designer is free to choose the
appropriate meaning of the position to their devices. For example, use the
device block as the basic unit to offset, instead of byte.

<img width="1536" height="1024" alt="seeking" src="https://github.com/user-attachments/assets/bd962a6b-2985-4ec0-afa7-f0d519dd0dae" />


Here, in this example, we create a simple char device file named "seeking" in
`/dev` directory. It has an internal buffer containing a DEC-HEX table, which
can be used to convert decimal number to its hex counterpart. Read the
different position of this device file can get the hex format number
accordingly. For example, read one byte from the position of 10 from the device
file, a char of `A` is returned. Read after position of 15, for example 16,
will cause the position cursor wrapping around.

## build the module

To build this module, execute:

```bash
make KERNELDIR=/path/to/kernel/source/dir
```

If you have already set and exported `KERNELDIR` environment variable, simply
execute `make` is enough.

If neither `KERNELDIR` environment variable nor `KERNELDIR` option of make
are set, the current running kernel will be built against.

## Usage

Copy **load_module.sh** and **seeking.ko** files to the target machine,
then run:

```bash
sh load_module.sh
```
##  Working Flow

COMPILE - make builds seeking.ko module with llseek() implementation

LOAD - sh load_module.sh inserts module, creates /dev/seeking device

SETUP - Driver initializes internal DEC-HEX conversion buffer table

POSITION - User program calls lseek() to navigate through buffer positions

CONVERT - Reading at position N returns hex character for decimal value N

##   Detailed Flow

┌──────────────────────────────────────────────────────────────────────────┐
│                              COMPILATION PHASE                            │
├──────────────────────────────────────────────────────────────────────────┤
│ make [KERNELDIR=/path/to/kernel]                                         │
│   ↓                                                                       │
│ Compiles main.c + fops.c → seeking.ko (kernel object)                     │
│   ↓                                                                       │
│ Implements file_operations.llseek = seeking_llseek()                      │
│                                                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                              LOADING PHASE                                │
├──────────────────────────────────────────────────────────────────────────┤
│ sh load_module.sh                                                         │
│   ↓                                                                       │
│ insmod seeking.ko → /dev/seeking device created                           │
│   ↓                                                                       │
│ Module registers:                                                         │
│   - open(), read(), write(), release()                                   │
│   - llseek() for file position manipulation                               │
│                                                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                              TEST EXECUTION                               │
├──────────────────────────────────────────────────────────────────────────┤
│ ./test/seeking_test                                                       │
│   ↓                                                                       │
│ 1. open("/dev/seeking", O_RDONLY)                                         │
│   ↓                                                                       │
│ 2. for(pos=15; pos>=0; pos--) {                                          │
│       lseek(fd, pos, SEEK_SET)  // Position at offset                    │
│       read(fd, &buf, 1)         // Read hex char at position             │
│       printf("%d(dec) = %c(hex)", pos, buf)                               │
│    }                                                                      │
│   ↓                                                                       │
│ Output: 15=F, 14=E, 13=D, ..., 0=0 (DEC to HEX conversion)                │
│                                                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                          BUFFER MANAGEMENT FLOW                           │
├──────────────────────────────────────────────────────────────────────────┤
│ Driver Buffer: "0123456789ABCDEF" (16-byte hex table)                     │
│   ↓                                                                       │
│ llseek() logic:                                                           │
│   - position > 15 → wraps around: pos = pos % 16                          │
│   - position < 0  → sets to 0                                            │
│   - SEEK_SET: absolute position                                           │
│   - SEEK_CUR: current + offset                                            │
│   - SEEK_END: buffer_end + offset                                         │
│   ↓                                                                       │
│ read() returns buffer[position] → hex character                           │
│                                                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                              CLEANUP PHASE                                │
├──────────────────────────────────────────────────────────────────────────┤
│ rmmod seeking                                                             │
│   ↓                                                                       │
│ Device file removed, buffer deallocated                                   │
└──────────────────────────────────────────────────────────────────────────┘


## test the module

In the **test** directory, we write a simple test program to test the
functionality of the kernel driver.

Build and copy the binary file to the target machine, run the test program
in terminal to check the output.

In this test program, we read the device file from offset 15 to offset 0, one
byte per read, and print the char to the terminal:

```
15(dec) = F(hex)
14(dec) = E(hex)
13(dec) = D(hex)
12(dec) = C(hex)
11(dec) = B(hex)
10(dec) = A(hex)
9(dec) = 9(hex)
8(dec) = 8(hex)
7(dec) = 7(hex)
6(dec) = 6(hex)
5(dec) = 5(hex)
4(dec) = 4(hex)
3(dec) = 3(hex)
2(dec) = 2(hex)
1(dec) = 1(hex)
0(dec) = 0(hex)
```

---

### ¶ The end
