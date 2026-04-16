# OS-Jackfruit: Mini Container Runtime with Kernel Memory Monitor



OS-Jackfruit is a lightweight container runtime written in C with a Linux kernel module for monitoring and enforcing container memory limits.

The project simulates a simplified Docker-like environment using:

* Linux namespaces
* `clone()`
* `chroot()`
* UNIX domain sockets
* Kernel `ioctl()` communication
* Logging threads and bounded buffers
* Kernel timers and workqueues

---

## Features

### User Space Runtime (`engine.c`)

* Start, stop, run, list, and log containers
* Process isolation using namespaces
* Root filesystem isolation using `chroot()`
* Per-container logs
* Supervisor-client architecture using UNIX sockets
* Threaded logging system with bounded buffer
* Graceful shutdown and cleanup

### Kernel Module (`monitor.c`)

* Tracks container memory usage
* Soft memory limit warnings
* Hard memory limit enforcement using `SIGKILL`
* Periodic monitoring using timers + workqueues
* Character device interface through `/dev/container_monitor`
* Communication with user space using `ioctl()`

---

## Technologies Used

* C
* Linux Kernel Module Programming
* POSIX Threads
* UNIX Domain Sockets
* Linux Namespaces
* `clone()`
* `chroot()`
* `ioctl()`
* Workqueues and Timers
* Git and GitHub

---

## Project Structure

```text
.
├── engine.c
├── monitor.c
├── monitor_ioctl.h
├── Makefile
├── monitor.ko
├── cpu_hog
├── memory_hog
├── rootfs-base/
├── rootfs-alpha/
├── rootfs-beta/
├── logs/
└── README.md
```

---

## How It Works

1. The supervisor starts and creates a control socket.
2. Containers are launched using `clone()` with isolated namespaces.
3. Each container uses its own root filesystem through `chroot()`.
4. The runtime registers containers with the kernel module.
5. The kernel module periodically checks memory usage.
6. If memory exceeds:

   * Soft limit → warning logged
   * Hard limit → process killed
7. Logs are collected and stored in separate files.

---

## Build Instructions

### 1. Install Required Packages

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r) git
```

### 2. Compile the User-Space Runtime

```bash
gcc engine.c -o engine -lpthread
```

### 3. Compile the Test Programs

```bash
gcc cpu_hog.c -o cpu_hog
```

```bash
gcc memory_hog.c -o memory_hog
```

### 4. Compile the Kernel Module

```bash
make
```

This generates:

* `monitor.ko`
* `monitor.o`
* `monitor.mod.o`

### 5. Create Root Filesystem Structure

```bash
mkdir -p rootfs-alpha/bin
mkdir -p rootfs-beta/bin
mkdir -p rootfs-base/bin
```

Copy required binaries:

```bash
cp /bin/sh rootfs-alpha/bin/
cp /bin/sh rootfs-beta/bin/
cp /bin/sh rootfs-base/bin/
```

Copy test programs:

```bash
cp cpu_hog rootfs-beta/
cp memory_hog rootfs-alpha/
```

For ARM64 systems, also copy required libraries:

```bash
mkdir -p rootfs-alpha/lib/aarch64-linux-gnu
mkdir -p rootfs-alpha/lib
mkdir -p rootfs-beta/lib/aarch64-linux-gnu
mkdir -p rootfs-beta/lib
```

```bash
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-alpha/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-alpha/lib/
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-beta/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-beta/lib/
```

---

## Commands Used

### Load Kernel Module

```bash
sudo insmod monitor.ko
ls /dev/container_monitor
```

### Start Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

### Start Container

```bash
sudo ./engine start alpha ./rootfs-alpha /memory_hog --soft-mib 40 --hard-mib 80
```

```bash
sudo ./engine start beta ./rootfs-beta /cpu_hog
```

### Run Container in Foreground

```bash
sudo ./engine run beta ./rootfs-beta "/cpu_hog 10"
```

### List Containers

```bash
sudo ./engine ps
```

### View Logs

```bash
sudo ./engine logs beta
```

### Stop Container

```bash
sudo ./engine stop beta
```

### View Kernel Logs

```bash
sudo dmesg | tail -20
```

---

## Screenshots

### 1. Project Folder Structure



### 2. Kernel Module Compilation



### 3. Loading Kernel Module



### 4. Supervisor Startup



### 5. Starting Alpha Container



### 6. Starting Beta Container



### 7. Container List (`ps`)



### 8. Viewing Logs



### 9. Memory Limit Trigger from Kernel Module



### 10. Stopping Container



### 11. Final Cleanup



---

## Important Concepts

### Namespaces Used

* `CLONE_NEWPID`
* `CLONE_NEWUTS`
* `CLONE_NEWNS`

### Memory Monitoring

* Soft limit generates a warning
* Hard limit kills the process

### Logging System

* Producer thread reads container logs
* Consumer thread writes logs to file
* Uses a bounded buffer to avoid overflow

### Kernel Techniques Used

* Linked lists
* Mutexes
* Workqueues
* Timers
* Character devices
* IOCTL communication

---

## Challenges Faced

* Permission issues with `clone()`
* Root vs non-root socket access
* Missing runtime libraries inside rootfs
* Dynamic `/proc` and `/sys` files causing Git issues
* GitHub authentication using personal access token

---

## Step-by-Step Reproduction Guide

If someone else wants to run this project from scratch, follow these steps in order.

### Step 1: Clone the Repository

```bash
git clone https://github.com/CJC1406/OS-Jackfruit.git
cd OS-Jackfruit
```

### Step 2: Install Dependencies

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r) git
```

### Step 3: Compile the Project

```bash
gcc engine.c -o engine -lpthread
```

```bash
gcc cpu_hog.c -o cpu_hog
```

```bash
gcc memory_hog.c -o memory_hog
```

```bash
make
```

### Step 4: Create Root Filesystems

```bash
mkdir -p rootfs-alpha/bin
mkdir -p rootfs-beta/bin
mkdir -p rootfs-base/bin
```

Copy shell:

```bash
cp /bin/sh rootfs-alpha/bin/
cp /bin/sh rootfs-beta/bin/
cp /bin/sh rootfs-base/bin/
```

Copy test programs:

```bash
cp cpu_hog rootfs-beta/
cp memory_hog rootfs-alpha/
```

For ARM64 systems:

```bash
mkdir -p rootfs-alpha/lib/aarch64-linux-gnu
mkdir -p rootfs-alpha/lib
mkdir -p rootfs-beta/lib/aarch64-linux-gnu
mkdir -p rootfs-beta/lib
```

```bash
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-alpha/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-alpha/lib/
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-beta/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-beta/lib/
```

### Step 5: Load the Kernel Module

```bash
sudo insmod monitor.ko
ls /dev/container_monitor
```

### Step 6: Start the Supervisor

Open Terminal 1:

```bash
sudo ./engine supervisor ./rootfs-base
```

Keep this terminal open.

### Step 7: Start Containers

Open Terminal 2:

```bash
sudo ./engine start alpha ./rootfs-alpha /memory_hog --soft-mib 40 --hard-mib 80
```

```bash
sudo ./engine start beta ./rootfs-beta /cpu_hog
```

### Step 8: Check Running Containers

```bash
sudo ./engine ps
```

### Step 9: View Logs

```bash
sudo ./engine logs alpha
```

```bash
sudo ./engine logs beta
```

### Step 10: Check Kernel Monitor Output

```bash
sudo dmesg | tail -20
```

### Step 11: Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

### Step 12: Cleanup

```bash
sudo pkill -9 -f "./engine"
rm -f /tmp/engine_ctrl.sock
```

---

## Future Improvements

* CPU usage limits
* Network namespace support
* Cgroup integration
* Web dashboard for container monitoring
* Better log formatting
* Container restart policies

---

## Author

Chiranth J C
Deeksha G


