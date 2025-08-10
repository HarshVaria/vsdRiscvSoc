# vsdRiscvSoc

# RISC-V Toolchain and Simulation Environment Setup

## 🎯 Overview 
This project documents the complete setup process for a RISC-V development toolchain, including:
- RISC-V GCC Cross-Compiler (riscv64-unknown-elf-gcc)
- Spike RISC-V ISA Simulator
- RISC-V Proxy Kernel (pk)
- Icarus Verilog for HDL simulation
- GTKWave for waveform viewing

The setup includes comprehensive troubleshooting documentation for compatibility issues between different toolchain versions and solutions for cross-compiler interference problems.

## 🔧 Prerequisites

### System Requirements
- OS: Ubuntu 22.04 LTS (64-bit) - native or VirtualBox (But works on wsl too, I have done this on wsl)
- wsl is faster than virtualbox, as I have compared the both. The output here are from wsl
- RAM: 4-8 GB recommended
- Storage: 30 GB free space
- CPU: 2+ cores recommended

## 🚀 Installation

Once you have completed the installation of the ubuntu distro, Open the terminal and run the base dependencies command shown below.

### Base Dependencies
```bash
sudo apt-get update
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave \
device-tree-compiler
```

Once that is done you will get some msg like this
<img width="940" height="817" alt="image" src="https://github.com/user-attachments/assets/1c594afa-bb92-4480-9806-48de56353d89" />

---

## 1. Workspace Setup

We first create a workspace where we will download and install all our files and tools.  
We store our home path so installing, cleaning, and removing files becomes easier.

```bash
cd ~
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```

---

## 2. Install RISC-V GCC Toolchain

Download the prebuilt toolchain, extract it, and verify it is installed correctly.  
[Download RISC-V GCC Toolchain](https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz)

```bash
wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
ls -la
ls -la riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
```

### Add to PATH
```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
echo $PATH | grep riscv
```
<img width="940" height="379" alt="image" src="https://github.com/user-attachments/assets/355a8c30-389f-41e4-ba58-cf73d46f72e3" />
---

## 3. Verifying the RISC-V Toolchain Installation

```bash
riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-objdump --version
riscv64-unknown-elf-gdb --version
riscv64-unknown-elf-gcc -dumpmachine
ls -la ~/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/ | grep riscv64
```

---

## 4. Fixing libncurses.so.5 Missing Library Error

If you get:
```
riscv64-unknown-elf-gdb: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory
```

It's because newer Ubuntu uses **libncurses.so.6** and **libtinfo.so.6**, but the prebuilt toolchain was compiled against version 5.

### Install legacy versions:
```bash
cd /tmp
wget http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libncurses5_6.3-2ubuntu0.1_amd64.deb
sudo dpkg -i libtinfo5_6.3-2ubuntu0.1_amd64.deb
sudo dpkg -i libncurses5_6.3-2ubuntu0.1_amd64.deb
```

### Create symbolic links:
```bash
sudo ln -s /lib/x86_64-linux-gnu/libncurses.so.5.9 /usr/lib/x86_64-linux-gnu/libncurses.so.5
sudo ln -s /lib/x86_64-linux-gnu/libtinfo.so.5.9 /usr/lib/x86_64-linux-gnu/libtinfo.so.5
sudo ldconfig
```

---

## 5. Install Device Tree Compiler (DTC)
```bash
sudo apt-get install -y device-tree-compiler
```
<img width="940" height="133" alt="image" src="https://github.com/user-attachments/assets/0987acd4-2277-4841-9e4c-37dbb8185722" />
---

## 6. Build Spike (RISC-V ISA Simulator)
```bash
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
cd $pwd/riscv_toolchain
```

---

## 7. Build RISC-V Proxy Kernel (Fixed Version)

**Problem:**  
Newer riscv-pk uses CSRs not supported by GCC 8.3.0 toolchain.  
**Solution:** Use version **v1.0.0**.

```bash
cd $pwd/riscv_toolchain
rm -rf riscv-pk
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
git checkout v1.0.0
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```

---

## 8. Ensure Cross Bin in PATH
```bash
# Current shell
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH

# Persistent
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 9. Install Icarus Verilog (Fixed Build)

**Problem:**  
RISC-V assembler was overriding the system assembler.  

**Solution:** Isolate build environment.

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
git pull
chmod +x autoconf.sh
./autoconf.sh

# Clean build if failed before
make distclean 2>/dev/null || true
git clean -fdx

# PATH isolation - use system GCC
echo $PATH > /tmp/saved_path
export PATH=/usr/bin:/bin:/usr/sbin:/sbin
export CC=/usr/bin/gcc
export CXX=/usr/bin/g++

# Verify correct tools
which gcc
which as
gcc --version

# Test compiler
echo 'int main(){return 0;}' > test.c
gcc test.c -o test
./test
echo $?
rm -f test test.c

# Build Icarus Verilog
./configure
make -j$(nproc)
sudo make install

# Restore full PATH
export PATH=$(cat /tmp/saved_path)
rm -f /tmp/saved_path
```

### Add /usr/local/bin to PATH
```bash
export PATH=/usr/local/bin:$PATH
echo 'export PATH=/usr/local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 10. Installation Check

**Note:**  
spike --version does not work. Use:
```bash
spike --help
```

This shows version info, e.g.:
```
Spike RISC-V ISA Simulator 1.1.1-dev
```
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/37ab1ec2-8d3c-49f4-8151-064cf77ee100" />

### Verify tools:
```bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v

which spike
spike --help

which pk
which iverilog
iverilog -V
```

---

## 11. Unique C Test (Username & Machine Dependent)

Create the test program:
```bash
cat > ~/riscv_toolchain/unique_test.c << 'EOF'
#include <stdint.h>
#include <stdio.h>

#ifndef USERNAME
#define USERNAME "unknown_user"
#endif

#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif

static uint64_t fnv1a64(const char *s) {
    const uint64_t FNV_OFFSET = 1469598103934665603ULL;
    const uint64_t FNV_PRIME = 1099511628211ULL;
    uint64_t h = FNV_OFFSET;
    for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
        h ^= (uint64_t)(*p);
        h *= FNV_PRIME;
    }
    return h;
}

int main(void) {
    const char *user = USERNAME;
    const char *host = HOSTNAME;
    char buf[256];
    int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
    if (n <= 0) return 1;
    uint64_t uid = fnv1a64(buf);
    printf("RISC-V Uniqueness Check\n");
    printf("User: %s\n", user);
    printf("Host: %s\n", host);
    printf("UniqueID: 0x%016llx\n", (unsigned long long)uid);
#ifdef __VERSION__
    unsigned long long vlen = (unsigned long long)sizeof(__VERSION__) - 1;
    printf("GCC_VLEN: %llu\n", vlen);
#endif
    return 0;
}
EOF
```

Get your username and hostname:
```bash
echo "Username: '$(id -un)'"
echo "Hostname: '$(hostname -s)'"
```
<img width="940" height="47" alt="image" src="https://github.com/user-attachments/assets/61552e98-95f6-4c4c-8285-6b23e22ad05a" />

**Example compilation for user 'ars' and host 'LAPTOPL8E-5720C':**
```bash
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME='"ars"' -DHOSTNAME='"LAPTOPL8E-5720C"' \
~/riscv_toolchain/unique_test.c -o ~/riscv_toolchain/unique_test

spike pk ~/riscv_toolchain/unique_test
```
<img width="940" height="112" alt="image" src="https://github.com/user-attachments/assets/e3f3cfde-3cdf-40f3-995f-855982414f60" />

**Error with path, as I am working in different directory
**The correct code:
<img width="940" height="33" alt="image" src="https://github.com/user-attachments/assets/58d3e828-2859-4b38-a51b-abf0b9e31d6a" />

**Note:** Replace 'ars' and 'LAPTOPL8E-5720C' with your actual username and hostname from the previous command output.

---

## 12. Final Test Output

After compiling and running the unique C test, you should see output similar to:

```
<img width="940" height="80" alt="image" src="https://github.com/user-attachments/assets/81d8c43a-558e-4aa7-a715-b587ac5180a4" />

```

This confirms the installation worked correctly — all tools and dependencies are functional.

---

## 🚨 Issues Encountered & Solutions

### Summary Table of All Problems and Fixes

| Task     | Issue Description                                  | Root Cause                                                 | Symptoms                                                    | Failed Attempts                                           | Final Solution                                             | Status |
|----------|----------------------------------------------------|------------------------------------------------------------|-------------------------------------------------------------|-----------------------------------------------------------|------------------------------------------------------------|--------|
| Task 1   | Missing legacy libraries                           | Ubuntu 22.04 has newer libncurses.so.6 but toolchain needs v5 | libncurses.so.5: cannot open shared object file           | N/A                                                       | Install legacy packages + create symbolic links            | ✅     |
| Task 7   | RISC-V Proxy Kernel build fails                    | GCC 8.3.0 too old for newer CSRs (menvcfg, senvcfg)         | Error: unknown CSR 'menvcfg'<br>Error: unknown CSR 'senvcfg' | N/A – Direct identification                               | Use compatible riscv-pk v1.0.0                           | ✅     |
| Task 9   | Icarus configure failure                           | RISC-V as used instead of system as                     | C compiler cannot create executables                      | Install dependencies, clean builds, force CC, env vars    | Complete PATH isolation during build                       | ✅     |
| Task 9   | Assembler option error                              | RISC-V assembler doesn't understand x86-64 options          | as: unrecognized option '--64'                            | Multiple configure attempts with different flags          | Use clean PATH /usr/bin:/bin:/usr/sbin:/sbin              | ✅     |
| Task 9   | Tool not found after install                        | /usr/local/bin not in PATH                                | iverilog: command not found                               | N/A                                                       | Add /usr/local/bin to PATH permanently                    | ✅     |
| Final Test | Preprocessor macro error                          | Shell expansion + preprocessor tokenization                 | 'kevinshah' undeclared identifier                         | Alternative quoting, variables, escaping                  | Explicit quoted string literals                            | ✅     |
| Final Test | Hostname parsing error                            | Hyphens parsed as separate tokens                           | 'Dell' and 'G15' undeclared                             | Multiple quoting syntaxes                                 | Direct string constants bypass shell expansion             | ✅     |

---

## 🔧 Detailed Problem Categories

### 1. Library Compatibility Issues
- **Problem:** Modern Ubuntu systems lack legacy library versions required by older toolchains.  
- **Impact:** GDB and other tools fail to start due to missing dependencies.  
- **Solution:** Manual installation of legacy packages with symbolic link creation.  

### 2. Version Incompatibility Between Tools
- **Problem:** Newer software versions use features not supported by older toolchain components.  
- **Impact:** Build failures with cryptic CSR errors.  
- **Solution:** Use version-matched compatible releases aligned with the toolchain era.  

### 3. Cross-Compiler PATH Interference
- **Problem:** RISC-V cross-compiler tools interfere with native x86-64 compilation.  
- **Impact:** Native builds fail because wrong assembler is used.  
- **Solution:** Temporary PATH isolation during native tool compilation.  

### 4. Shell Command Expansion Issues
- **Problem:** Shell substitution creates unquoted tokens for the C preprocessor.  
- **Impact:** Preprocessor sees separate identifiers instead of single string constants.  
- **Solution:** Use explicit string literals.  

### 5. Installation Path Configuration
- **Problem:** Tools install to non-standard locations not in default PATH.  
- **Impact:** Installed tools appear missing.  
- **Solution:** Permanent PATH updates including all install directories.  

---

## 🎯 Key Learning Points

### Debugging Methodology
- Multi-stage debugging: Complex issues often require multiple investigation rounds.
- Root cause isolation: Surface symptoms may not reveal the underlying problem.
- Environment testing: Test individual components to identify interference sources.

### Toolchain Management
- Version compatibility: All components must match in feature support and era.
- PATH management: Cross-compilers can pollute native build environments.
- Clean environments: Isolate native and cross-compilation environments when needed.

### Configuration Best Practices
- Persistent configuration: Always make PATH changes permanent via .bashrc.
- Verification steps: Test each installation step before proceeding.
- Backup strategies: Save working configurations before making changes.

---

## 🔍 Troubleshooting Tips

**When Builds Fail:**
- Check which tools are actually being used (which gcc, which as).
- Verify library dependencies (ldd for missing libraries).
- Examine detailed error logs, not just summary messages.
- Test components individually in isolation.

**For PATH Issues:**
- Use echo $PATH to verify current configuration.
- Test with minimal clean PATH when debugging.
- Check installation directories with ls -la.
- Verify permanent changes with fresh shell sessions.

**For Compatibility Problems:**
- Check tool versions and release dates for alignment.
- Look for older compatible versions when needed.
- Understand feature dependencies between components.
- Consider container environments for complex setups.

---

## 📋 Final Verification Checklist
- ✅ All required tools found in PATH (which commands succeed)  
- ✅ Tool versions compatible with each other  
- ✅ Unique test program compiles without errors  
- ✅ Unique test program runs and produces expected output  
- ✅ PATH configuration persists across shell sessions  
- ✅ No cross-compiler interference with native builds
