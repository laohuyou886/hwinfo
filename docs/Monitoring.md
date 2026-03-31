# Real-time Monitoring API Documentation

The hwinfo library provides real-time monitoring functionality for CPU, RAM, and disk devices. This allows you to track utilization, frequencies, and available space dynamically.

## Table of Contents
- [Monitor Template](#monitor-template)
- [CPU Monitoring](#cpu-monitoring)
- [RAM Monitoring](#ram-monitoring)
- [Disk Monitoring](#disk-monitoring)
- [Example: Background Monitoring](#example-background-monitoring)

## Monitor Template

The `hwinfo::monitoring::Monitor<T>` template class provides a generic background monitor that periodically fetches data and invokes a callback.

```cpp
namespace hwinfo::monitoring {

template <typename T>
class Monitor {
public:
    using Callback = std::function<void(const T&)>;
    using FetchFn = std::function<T()>;

    Monitor(Fetch fetch, Callback callback, std::chrono::milliseconds interval)
        : _fetch(std::move(fetch)), _callback(std::move(callback)), _interval(interval) {}

    ~Monitor() { stop(); }

    Monitor(const Monitor&) = delete;
    Monitor& operator=(const Monitor&) = delete;
    Monitor(Monitor&&) = delete;
    Monitor& operator=(Monitor&&) = delete;

    void start();
    void stop();
    bool is_running() const;
};

} // namespace hwinfo::monitoring
```

### Constructor
```cpp
Monitor(FetchFn fetch, Callback callback, std::chrono::milliseconds interval)
```
Creates a new monitor with the given fetch function, callback, and update interval.

**Parameters:**
- `fetch` - Function that fetches a snapshot of monitoring data (`T()`)
- `callback` - Function that receives the data (`void(const T&)`)
- `interval` - How often to fetch new data

### `start()`
```cpp
void start();
```
Starts the background monitoring thread. The monitor will begin calling the callback with updated data at the specified interval.

### `stop()`
```cpp
void stop();
```
Stops the background monitoring thread. Called automatically in the destructor.

### `is_running()`
```cpp
bool is_running() const;
```
Checks if the monitor is currently running.

**Return Value:**
- `true` if monitor is running
- `false` otherwise

---

## CPU Monitoring

CPU monitoring provides real-time CPU utilization and frequency information. Located in `<hwinfo/monitoring/cpu.h>`.

### Data Structure

```cpp
namespace hwinfo::monitoring::cpu {

struct Data {
    double utilization;                       // average across all threads [0, 1]
    std::vector<double> thread_utilization;   // per-thread utilization [0, 1]
    std::vector<int64_t> thread_frequency_hz; // per-thread current clock rate in Hz
};

} // namespace hwinfo::monitoring::cpu
```

| Field | Type | Description |
|-------|------|-------------|
| `utilization` | `double` | Average CPU utilization across all logical threads (0.0 = idle, 1.0 = fully utilized) |
| `thread_utilization` | `std::vector<double>` | Utilization for each individual logical thread |
| `thread_frequency_hz` | `std::vector<int64_t>` | Current clock frequency for each thread in Hertz |

### Free Functions

#### `utilization()`
```cpp
double utilization(std::chrono::milliseconds sleep = 200ms);
```
Returns the average CPU utilization across all logical threads.

**Parameters:**
- `sleep` - How long to measure CPU usage (more time = more accurate result)

**Return Value:**
- `double` - Average utilization in range [0, 1]

---

#### `thread_utilization()`
```cpp
std::vector<double> thread_utilization(std::chrono::milliseconds sleep = 200ms);
```
Returns per-thread CPU utilization.

**Parameters:**
- `sleep` - How long to measure CPU usage

**Return Value:**
- `std::vector<double>` - Utilization for each thread in range [0, 1]

---

#### `thread_frequency_hz()`
```cpp
std::vector<int64_t> thread_frequency_hz();
```
Returns the current clock frequency for each logical thread.

**Return Value:**
- `std::vector<int64_t>` - Current frequency of each thread in Hertz

---

#### `fetch()`
```cpp
Data fetch(std::chrono::milliseconds sleep = 200ms);
```
Fetches a complete monitoring snapshot.

**Parameters:**
- `sleep` - How long to measure CPU usage

**Return Value:**
- `Data` - [Data](#data-structure) structure containing all CPU monitoring data

---

### Monitor Type Alias
```cpp
using Monitor = hwinfo::monitoring::Monitor<Data>;
```
Convenient alias for a CPU monitor.

### Platform Support
| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Average Utilization | ✔️ | ✔️ | ✔️ |
| Per-thread Utilization | ✔️ | ✔️ | ✔️ |
| Current Frequency | ✔️ | ❌ | ✔️ |

---

## RAM Monitoring

RAM monitoring provides real-time free and available memory information. Located in `<hwinfo/monitoring/ram.h>`.

### Data Structure

```cpp
namespace hwinfo::monitoring::ram {

struct Data {
    uint64_t free_bytes;      // memory not in use at all
    uint64_t available_bytes; // memory available for new allocations (includes reclaimable)
};

} // namespace hwinfo::monitoring::ram
```

| Field | Type | Description |
|-------|------|-------------|
| `free_bytes` | `uint64_t` | Amount of completely free memory in bytes |
| `available_bytes` | `uint64_t` | Amount of memory available for allocation (includes free + reclaimable page cache) |

### Free Functions

#### `free_bytes()`
```cpp
uint64_t free_bytes();
```
Returns the amount of completely free physical memory.

**Return Value:**
- `uint64_t` - Free memory in bytes

---

#### `available_bytes()`
```cpp
uint64_t available_bytes();
```
Returns the amount of memory available for new allocations.

**Return Value:**
- `uint64_t` - Available memory in bytes

**Note:** This is the value you usually want for checking how much memory is available.

---

#### `fetch()`
```cpp
Data fetch();
```
Fetches a complete monitoring snapshot.

**Return Value:**
- `Data` - [Data](#data-structure_1) structure containing all RAM monitoring data

---

### Monitor Type Alias
```cpp
using Monitor = hwinfo::monitoring::Monitor<Data>;
```
Convenient alias for a RAM monitor.

### Platform Support
| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Free Bytes | ✔️ | ✔️ | ✔️ |
| Available Bytes | ✔️ | ✔️ | ✔️ |

---

## Disk Monitoring

Disk monitoring provides real-time free space checking for disks. Located in `<hwinfo/monitoring/disk.h>`.

### Data Structure

```cpp
namespace hwinfo::monitoring::disk {

struct Data {
    std::string mount_point;
    uint64_t free_bytes;
};

} // namespace hwinfo::monitoring::disk
```

| Field | Type | Description |
|-------|------|-------------|
| `mount_point` | `std::string` | The mount point that was queried |
| `free_bytes` | `uint64_t` | Free space available on the mount point in bytes |

### Free Functions

#### `get_free_size()` (by mount point)
```cpp
std::uint64_t get_free_size(const std::string& mount_point);
```
Returns the free space available at the given mount point.

**Parameters:**
- `mount_point` - Path to the mount point (e.g., `"/"`, `"C:\\"`)

**Return Value:**
- `std::uint64_t` - Free space in bytes

---

#### `get_free_size()` (by Disk)
```cpp
std::uint64_t get_free_size(const Disk& disk);
```
Returns the sum of free space across all mount points belonging to the given disk.

**Parameters:**
- `disk` - A `hwinfo::Disk` object

**Return Value:**
- `std::uint64_t` - Total free space across all mount points of the disk in bytes

---

#### `fetch()`
```cpp
Data fetch(const std::string& mount_point);
```
Fetches a monitoring snapshot for a specific mount point.

**Parameters:**
- `mount_point` - Path to the mount point

**Return Value:**
- `Data` - [Data](#data-structure_2) structure containing disk monitoring data

---

### Monitor Type Alias
```cpp
using Monitor = hwinfo::monitoring::Monitor<Data>;
```
Convenient alias for a disk monitor.

### Platform Support
| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Free Size by Mount Point | ✔️ | ✔️ | ✔️ |
| Free Size by Disk | ✔️ | ✔️ | ✔️ |

---

## Example: Background Monitoring

```cpp
#include <hwinfo/hwinfo.h>
#include <hwinfo/monitoring/cpu.h>
#include <hwinfo/monitoring/ram.h>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    using namespace hwinfo::monitoring;

    // Monitor CPU every second
    auto cpu_monitor = cpu::Monitor(
        []() { return cpu::fetch(std::chrono::milliseconds(200)); },
        [](const cpu::Data& data) {
            std::cout << "CPU Utilization: " << (data.utilization * 100) << "%" << std::endl;
        },
        std::chrono::seconds(1)
    );

    // Monitor RAM every 5 seconds
    auto ram_monitor = ram::Monitor(
        []() { return ram::fetch(); },
        [](const ram::Data& data) {
            double availableMiB = static_cast<double>(data.available_bytes) / (1024.0 * 1024.0);
            std::cout << "Available RAM: " << availableMiB << " MiB" << std::endl;
        },
        std::chrono::seconds(5)
    );

    // Start monitoring
    cpu_monitor.start();
    ram_monitor.start();

    std::cout << "Monitoring started. Press Enter to stop..." << std::endl;
    std::cin.get();

    // Monitors stop automatically in destructor
    return 0;
}
```

## Example: Single-shot Query

```cpp
#include <hwinfo/hwinfo.h>
#include <hwinfo/monitoring/cpu.h>
#include <hwinfo/monitoring/ram.h>
#include <hwinfo/monitoring/disk.h>
#include <iostream>

int main() {
    // Get current CPU utilization
    double cpu_usage = hwinfo::monitoring::cpu::utilization();
    std::cout << "CPU Usage: " << (cpu_usage * 100) << "%" << std::endl;

    // Get current available memory
    uint64_t available = hwinfo::monitoring::ram::available_bytes();
    double availableGiB = static_cast<double>(available) / (1024.0 * 1024.0 * 1024.0);
    std::cout << "Available Memory: " << availableGiB << " GiB" << std::endl;

    // Get free space on C:
    uint64_t free_c = hwinfo::monitoring::disk::get_free_size("C:\\");
    double free_cGiB = static_cast<double>(free_c) / (1024.0 * 1024.0 * 1024.0);
    std::cout << "C:\\ free: " << free_cGiB << " GiB" << std::endl;

    return 0;
}
```

## Notes

- CPU utilization measurement requires a blocking sleep to calculate the delta between two samples
- Shorter sleep times give more responsive but less accurate results
- 200ms is the default and provides a good balance
- The `Monitor` class runs the callback on a background thread - make sure your callback is thread-safe
