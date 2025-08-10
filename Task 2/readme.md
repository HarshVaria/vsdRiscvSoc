# 🛠️ RISC-V Task 2 - Local Setup Verification

## Overview
This repository contains the implementation and verification of 4 RISC-V C programs compiled with the local RISC-V toolchain and executed using Spike simulator with proxy kernel (pk). Each program includes a uniqueness mechanism that embeds username, hostname, machine ID, and timestamps to ensure outputs are unique to this PC.

## 🔧 System Information

### Spike Version
To find its version we use the following command :
```bash
$ spike -h
```
And we got its output as :-

<img width="940" height="453" alt="image" src="https://github.com/user-attachments/assets/4ab872c0-f55b-45f0-bb3c-162b3581d61c" />

### GCC Toolchain Information
To find its version we use the following command :
```bash
$ riscv64-unknown-elf-gcc -v
```
And we got its output as :-

<img width="940" height="168" alt="image" src="https://github.com/user-attachments/assets/a6581f5e-6989-4f13-95d6-d5f09477d23a" />

## 🔍 Step-by-Step Implementation

### A. Uniqueness Mechanism Setup
First, set identity variables in the Linux host shell:

```bash
export U=$(id -un)
export H=$(hostname -s)
export M=$(cat /etc/machine-id | head -c 16)
export T=$(date -u +%Y-%m-%dT%H:%M:%SZ)
export E=$(date +%s)
```

#### Verification of Environment Variables
```bash
echo "Username: $U"
echo "Hostname: $H" 
echo "Machine ID: $M"
echo "Build UTC: $T"
echo "Build Epoch: $E"
```

**Output (for user 'ars' on 'LAPTOPL8E-5720C'):**

<img width="940" height="113" alt="image" src="https://github.com/user-attachments/assets/bac0ae49-6d59-4c3d-bdcd-116e500a7d17" />

### B. Common Header (unique.h)
Created using EOF to avoid escape character issues:

```bash
cat > unique.h << 'EOF'
#ifndef UNIQUE_H
#define UNIQUE_H
#include <stdio.h>
#include <stdint.h>
#include <time.h>
#ifndef USERNAME
#define USERNAME "unknown_user"
#endif
#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif
#ifndef MACHINE_ID
#define MACHINE_ID "unknown_machine"
#endif
#ifndef BUILD_UTC
#define BUILD_UTC "unknown_time"
#endif
#ifndef BUILD_EPOCH
#define BUILD_EPOCH 0
#endif
static uint64_t fnv1a64(const char *s) {
const uint64_t OFF = 1469598103934665603ULL, PRIME = 1099511628211ULL;
uint64_t h = OFF;
for (const unsigned char *p=(const unsigned char*)s; *p; ++p) {
h ^= *p; h *= PRIME;
}
return h;
}
static void uniq_print_header(const char *program_name) {
time_t now = time(NULL);
char buf[512];
int n = snprintf(buf, sizeof(buf), "%s|%s|%s|%s|%ld|%s|%s",
USERNAME, HOSTNAME, MACHINE_ID, BUILD_UTC,
(long)BUILD_EPOCH, __VERSION__, program_name);
(void)n;
uint64_t proof = fnv1a64(buf);
char rbuf[600];
snprintf(rbuf, sizeof(rbuf), "%s|run_epoch=%ld", buf, (long)now);
uint64_t runid = fnv1a64(rbuf);
printf("=== RISC-V Proof Header ===\n");
printf("User : %s\n", USERNAME);
printf("Host : %s\n", HOSTNAME);
printf("MachineID : %s\n", MACHINE_ID);
printf("BuildUTC : %s\n", BUILD_UTC);
printf("BuildEpoch : %ld\n", (long)BUILD_EPOCH);
printf("GCC : %s\n", __VERSION__);
printf("PointerBits: %d\n", (int)(8*(int)sizeof(void*)));
printf("Program : %s\n", program_name);
printf("ProofID : 0x%016llx\n", (unsigned long long)proof);
printf("RunID : 0x%016llx\n", (unsigned long long)runid);
printf("===========================\n");
}
#endif
EOF
```

### C. Program Implementation

#### 1. factorial.c
```bash
cat > factorial.c << 'EOF'
#include "unique.h"
static unsigned long long fact(unsigned n){ return (n<2)?1ULL:n*fact(n-1); }
int main(void){
uniq_print_header("factorial");
unsigned n = 12;
printf("n=%u, n!=%llu\n", n, fact(n));
return 0;
}
EOF
```

#### 2. max_array.c
```bash
cat > max_array.c << 'EOF'
#include "unique.h"
int main(void){
uniq_print_header("max_array");
int a[] = {42,-7,19,88,3,88,5,-100,37};
int n = sizeof(a)/sizeof(a[0]), max=a[0];
for(int i=1;i<n;i++) if(a[i]>max) max=a[i];
printf("Array length=%d, Max=%d\n", n, max);
return 0;
}
EOF
```

#### 3. bitops.c
```bash
cat > bitops.c << 'EOF'
#include "unique.h"
int main(void){
uniq_print_header("bitops");
unsigned x=0xA5A5A5A5u, y=0x0F0F1234u;
printf("x&y=0x%08X\n", x&y);
printf("x|y=0x%08X\n", x|y);
printf("x^y=0x%08X\n", x^y);
printf("x<<3=0x%08X\n", x<<3);
printf("y>>2=0x%08X\n", y>>2);
return 0;
}
EOF
```

#### 4. bubble_sort.c
```bash
cat > bubble_sort.c << 'EOF'
#include "unique.h"
void bubble(int *a,int n){ for(int i=0;i<n-1;i++) for(int j=0;j<n-1-i;j++) if(a[j]>a[j+1]){int t=a[j];a[j]=a[j+1];a[j+1]=t;} }
int main(void){
uniq_print_header("bubble_sort");
int a[]={9,4,1,7,3,8,2,6,5}, n=sizeof(a)/sizeof(a[0]);
bubble(a,n);
printf("Sorted:"); for(int i=0;i<n;i++) printf(" %d",a[i]); puts("");
return 0;
}
EOF
```

### D. Build, Run, and Capture Evidence

#### Program 1: factorial
**Compile:**
```bash
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    factorial.c -o factorial
```

**Run:**
```bash
spike pk ./factorial
```

**Output:**

<img width="940" height="215" alt="image" src="https://github.com/user-attachments/assets/8c909fbd-eb12-4e33-82d4-c11abfb6c2c6" />

#### Program 2: max_array
**Compile:**
```bash
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    max_array.c -o max_array
```

**Run:**
```bash
spike pk ./max_array
```

**Output:**

<img width="940" height="216" alt="image" src="https://github.com/user-attachments/assets/78827d6c-acb3-40a0-ab02-3b1bb30fdbe2" />

#### Program 3: bitops
**Compile:**
```bash
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    bitops.c -o bitops
```

**Run:**
```bash
spike pk ./bitops
```

**Output:**

<img width="940" height="259" alt="image" src="https://github.com/user-attachments/assets/fdebc00d-9082-46b4-9fe3-7ad5e68e9769" />

#### Program 4: bubble_sort
**Compile:**
```bash
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    bubble_sort.c -o bubble_sort
```

**Run:**
```bash
spike pk ./bubble_sort
```

**Output:**

<img width="940" height="225" alt="image" src="https://github.com/user-attachments/assets/439da00e-529a-47ab-92e4-b1715114c44c" />

### E. Produce Assembly and Disassembly

#### factorial
**Generate Assembly:**
```bash
riscv64-unknown-elf-gcc -O0 -S -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    factorial.c -o factorial.s
```

**Generate Disassembly:**
```bash
riscv64-unknown-elf-objdump -d ./factorial | sed -n '/<main>:/,/^$/p' > factorial_main_objdump.txt
```

#### max_array
**Generate Assembly:**
```bash
riscv64-unknown-elf-gcc -O0 -S -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    max_array.c -o max_array.s
```

**Generate Disassembly:**
```bash
riscv64-unknown-elf-objdump -d ./max_array | sed -n '/<main>:/,/^$/p' > max_array_main_objdump.txt
```

#### bitops
**Generate Assembly:**
```bash
riscv64-unknown-elf-gcc -O0 -S -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    bitops.c -o bitops.s
```

**Generate Disassembly:**
```bash
riscv64-unknown-elf-objdump -d ./bitops | sed -n '/<main>:/,/^$/p' > bitops_main_objdump.txt
```

#### bubble_sort
**Generate Assembly:**
```bash
riscv64-unknown-elf-gcc -O0 -S -march=rv64imac -mabi=lp64 \
    -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
    -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
    bubble_sort.c -o bubble_sort.s
```

**Generate Disassembly:**
```bash
riscv64-unknown-elf-objdump -d ./bubble_sort | sed -n '/<main>:/,/^$/p' > bubble_sort_main_objdump.txt
```

## 📁 Repository Structure
```
riscv-task2-ars/
├── unique.h
├── factorial.c
├── factorial.s
├── factorial_main_objdump.txt
├── factorial_output.png
├── factorial_main_asm.png
├── max_array.c
├── max_array.s
├── max_array_main_objdump.txt
├── max_array_output.png
├── max_array_main_asm.png
├── bitops.c
├── bitops.s
├── bitops_main_objdump.txt
├── bitops_output.png
├── bitops_main_asm.png
├── bubble_sort.c
├── bubble_sort.s
├── bubble_sort_main_objdump.txt
├── bubble_sort_output.png
├── bubble_sort_main_asm.png
├── instruction_decoding.md
└── README.md
```

## 🔧 Key Features

### Uniqueness Mechanism
- **ProofID**: Generated from user, host, machine ID, build time, GCC version, and program name
- **RunID**: Adds runtime timestamp for execution verification
- **Machine Independence**: Each PC produces different hash values
- **Temporal Independence**: Each build/run produces different IDs

### Compilation Flags
- **-O0**: No optimization for clear assembly output
- **-g**: Debug information included
- **-march=rv64imac**: RISC-V 64-bit with integer, multiplication, atomic, and compressed extensions
- **-mabi=lp64**: 64-bit ABI with 64-bit pointers and long integers

### Assembly Generation
- **Source Assembly**: Generated with `-S` flag
- **Object Disassembly**: Extracted main function disassembly using `objdump`
- **Clean Output**: Filtered to show only main function code

## 🔍 Verification Notes
- ProofID is unique per user+host+machine+build combination
- RunID adds per-run randomness for execution verification
- All .s and .objdump files match the compiled code
- Screenshots contain visible ProofID and RunID for verification
- All programs demonstrate different RISC-V instruction patterns

## 🎯 Learning Objectives Achieved
- ✅ RISC-V cross-compilation setup verification
- ✅ Spike simulator functionality testing
- ✅ Assembly code generation and analysis
- ✅ Object file disassembly examination
- ✅ Unique output generation for authenticity
- ✅ Multiple program types: recursion, arrays, bit operations, sorting
