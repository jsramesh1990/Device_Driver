# **SHORT I/O Port Driver**

## **Project Overview**
This Linux kernel module implements a basic character device driver for I/O port manipulation. It provides direct hardware port access through device file operations, allowing userspace programs to read from and write to I/O ports. The driver is based on parallel port operations and demonstrates fundamental hardware interaction in the Linux kernel.

## **Build Instructions**

```bash
# Option 1: With KERNELDIR environment variable
export KERNELDIR=/path/to/kernel/source
make

# Option 2: With make parameter
make KERNELDIR=/path/to/kernel/source/dir

# Option 3: Using current running kernel
make
```

## **Installation**

```bash
# Load module with default base address 0x200
insmod short_ioport.ko

# Or specify custom base address
insmod short_ioport.ko base=0x378

# Verify module loaded
lsmod | grep short
```

## **Device File Operations**

The module creates a character device `/dev/short` with the following operations:

| Operation | Function | Description |
|-----------|----------|-------------|
| `open()` | `short_open()` | Opens device (no initialization needed) |
| `read()` | `short_read()` | Reads 1 byte from I/O port |
| `write()` | `short_write()` | Writes bytes to I/O port |
| `release()` | `short_release()` | Closes device (no cleanup needed) |

## **Working Flow**

```mermaid
graph TD
    A[Userspace Program] --> B[System Calls]
    
    subgraph "Kernel Space"
        B --> C[/dev/short Device File]
        C --> D{File Operations}
        
        D --> E[short_open]
        E --> F[Returns 0]
        
        D --> G[short_read]
        G --> H[inb(port) Read byte]
        H --> I[Memory Barrier: rmb]
        I --> J[copy_to_user]
        J --> K[Return 1 byte]
        
        D --> L[short_write]
        L --> M[copy_from_user]
        M --> N[Allocate kernel buffer]
        N --> O[outb(port, val) Write bytes]
        O --> P[Memory Barrier: wmb]
        P --> Q[Free buffer]
        Q --> R[Return count]
        
        D --> S[short_release]
        S --> T[Returns 0]
    end
    
    subgraph "Hardware"
        H --> U[I/O Port 0x200]
        O --> U
        U --> V[Parallel Port Hardware]
    end
    
    J --> W[Return data to userspace]
    R --> X[Return bytes written]
```

## **Detailed Implementation Flow**

### **1. Module Initialization**
```
insmod short_ioport.ko → m_init()
    ↓
Set short_base = base (default 0x200)
    ↓
request_region(short_base, SHORT_NR_PORTS, "short")
    ↓
Reserves I/O port range [0x200, 0x200+SHORT_NR_PORTS]
    ↓
register_chrdev(major, "short", &fops)
    ↓
Create /dev/short device with major number
```

### **2. Device Open Flow**
```
open("/dev/short", O_RDWR) → short_open()
    ↓
Returns 0 (always successful)
    ↓
No hardware initialization needed
```

### **3. Read Operation Flow**
```
read(fd, buf, 1) → short_read()
    ↓
Check position (*f_pos > 0) → return 0 if beyond first byte
    ↓
inb(short_base) → Read byte from I/O port
    ↓
rmb() → Read memory barrier for consistency
    ↓
(*f_pos)++ → Advance file position
    ↓
copy_to_user(buf, &v, 1) → Copy byte to userspace
    ↓
Return 1 (bytes read)
```

### **4. Write Operation Flow**
```
write(fd, buf, count) → short_write()
    ↓
kmalloc(cnt, GFP_KERNEL) → Allocate kernel buffer
    ↓
copy_from_user(kbuf, buf, cnt) → Copy data from userspace
    ↓
For each byte in buffer:
    outb(*ptr, short_base) → Write byte to I/O port
    wmb() → Write memory barrier
    ptr++ → Next byte
    ↓
kfree(kbuf) → Free kernel buffer
    ↓
Return count (bytes written)
```

### **5. Device Close Flow**
```
close(fd) → short_release()
    ↓
Returns 0 (always successful)
    ↓
No hardware cleanup needed
```

### **6. Module Cleanup**
```
rmmod short_ioport → m_exit()
    ↓
unregister_chrdev(major, "short")
    ↓
Remove /dev/short device
    ↓
release_region(short_base, SHORT_NR_PORTS)
    ↓
Free I/O port range for other drivers
```

## **Key Kernel APIs Used**

### **I/O Port Operations**
```c
// Read byte from port
unsigned char inb(unsigned long port);

// Write byte to port
void outb(unsigned char value, unsigned long port);

// Read memory barrier (ensures read completion)
rmb();

// Write memory barrier (ensures write completion)
wmb();
```

### **Port Reservation**
```c
// Reserve I/O port region
int request_region(unsigned long start, unsigned long n, const char *name);

// Release I/O port region
void release_region(unsigned long start, unsigned long n);
```

### **Character Device Registration**
```c
// Register character device
int register_chrdev(unsigned int major, const char *name,
                    struct file_operations *fops);

// Unregister character device
void unregister_chrdev(unsigned int major, const char *name);
```

### **Memory Management**
```c
// Allocate kernel memory
void *kmalloc(size_t size, gfp_t flags);

// Free kernel memory
void kfree(const void *objp);
```

### **User-Kernel Data Transfer**
```c
// Copy from userspace to kernel
unsigned long copy_from_user(void *to, const void __user *from,
                             unsigned long n);

// Copy from kernel to userspace
unsigned long copy_to_user(void __user *to, const void *from,
                           unsigned long n);
```

## **Testing**

### **1. Basic Testing with dd**
```bash
# Write byte 0x55 to port
echo -n $'\x55' | sudo dd of=/dev/short bs=1 count=1

# Read byte from port
sudo dd if=/dev/short bs=1 count=1 | hexdump -C
```

### **2. Python Test Script**
```python
#!/usr/bin/env python3
import struct

# Write bytes to device
with open('/dev/short', 'wb') as f:
    # Write test pattern
    for i in range(256):
        f.write(struct.pack('B', i))

# Read from device
with open('/dev/short', 'rb') as f:
    data = f.read(1)
    print(f"Read byte: 0x{data.hex()}")
```

### **3. C Test Program**
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("/dev/short", O_RDWR);
    if (fd < 0) {
        perror("open");
        return 1;
    }
    
    // Write test byte
    char write_byte = 0xAA;
    if (write(fd, &write_byte, 1) != 1) {
        perror("write");
    }
    
    // Read back
    char read_byte;
    lseek(fd, 0, SEEK_SET);
    if (read(fd, &read_byte, 1) != 1) {
        perror("read");
    }
    
    printf("Written: 0x%02X, Read: 0x%02X\n", 
           write_byte & 0xFF, read_byte & 0xFF);
    
    close(fd);
    return 0;
}
```

## **Expected Behavior**

### **Write Operation Output:**
```
port=512(0x200), val=85(0x55)
port=512(0x200), val=170(0xAA)
```
*Visible via `dmesg` or `journalctl -k`*

### **Read Operation:**
- Returns single byte from port 0x200
- Position advances after each read
- Subsequent reads return 0 bytes (EOF)

## **Hardware Requirements**

### **Default Configuration:**
- **Base Address**: 0x200 (Parallel Port LPT1)
- **Port Range**: 8 ports (SHORT_NR_PORTS)

### **Common Parallel Port Addresses:**
- LPT1: 0x378 (default for most systems)
- LPT2: 0x278
- LPT3: 0x3BC

### **Testing Hardware Options:**
1. **Real Parallel Port**: Connect to actual hardware
2. **Virtual Port**: Use with QEMU virtual machine
3. **Loopback Connector**: Connect output pins to input pins

## **Security Considerations**

⚠️ **WARNING**: This driver provides direct hardware access!

1. **Privilege Required**: Root access needed for device operations
2. **Hardware Damage Risk**: Writing to wrong ports can damage hardware
3. **System Stability**: Can crash system if used incorrectly
4. **Port Conflicts**: May conflict with other drivers using same ports

### **Safety Recommendations:**
```bash
# Set restrictive permissions
sudo chmod 600 /dev/short
sudo chown root:root /dev/short

# Use specific group for access
sudo groupadd portusers
sudo chgrp portusers /dev/short
sudo chmod 660 /dev/short
```

## **Troubleshooting**

### **1. Module Load Fails:**
```bash
# Check if ports are available
cat /proc/ioports | grep 0200
# If busy, try different base address
sudo insmod short_ioport.ko base=0x378
```

### **2. Permission Denied:**
```bash
# Check device permissions
ls -l /dev/short
# Run as root or adjust permissions
sudo chmod 666 /dev/short  # Temporary solution
```

### **3. No Parallel Port Hardware:**
```bash
# Check for parallel port
lspci | grep -i parallel
dmesg | grep -i parport
# Use virtual machine with parallel port emulation
```

### **4. Read Returns Wrong Data:**
```bash
# Check hardware connection
# Verify port address
cat /sys/module/short_ioport/parameters/base
```

## **Advanced Usage**

### **Custom Port Configuration:**
```bash
# Load with different base and size
sudo insmod short_ioport.ko base=0x378 size=16

# Check module parameters
modinfo short_ioport.ko
```

### **Kernel Debug Messages:**
```bash
# Enable debug output
echo 8 > /proc/sys/kernel/printk

# Monitor kernel messages
dmesg -w
tail -f /var/log/kern.log
```

## **Performance Considerations**

1. **Memory Barriers**: `rmb()` and `wmb()` ensure proper ordering but add overhead
2. **User-Kernel Copies**: `copy_to/from_user()` have validation overhead
3. **Port Access**: `inb()`/`outb()` are relatively slow (~100ns each)
4. **Buffer Allocation**: `kmalloc()`/`kfree()` per write operation

## **Alternative Implementations**

### **Using ioport.h Helpers:**
```c
// Alternative read/write functions
unsigned char ioread8(void __iomem *addr);
void iowrite8(u8 value, void __iomem *addr);
```

### **MMIO (Memory-Mapped I/O):**
```c
// For memory-mapped devices
void __iomem *ioremap(resource_size_t offset, unsigned long size);
iounmap(void __iomem *addr);
```

## **Cleanup**

```bash
# Remove module
sudo rmmod short_ioport

# Verify cleanup
ls -l /dev/short  # Should not exist
cat /proc/ioports | grep 0200  # Ports should be free
```

## **References**
- Linux Kernel Documentation: `Documentation/driver-api/io-port.rst`
- Linux Device Drivers, 3rd Edition - Chapter 9
- `include/asm-generic/io.h` - I/O port operations
- `arch/x86/include/asm/io.h` - x86-specific I/O functions
- Parallel Port Programming Guide

## **License**
GPL v2
