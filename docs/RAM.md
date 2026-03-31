# Memory (RAM) API Documentation

The `hwinfo::Memory` class provides information about system RAM (Random Access Memory) including details about individual memory modules and available memory statistics.

## Table of Contents
- [Class Overview](#class-overview)
- [Nested Structures](#nested-structures)
  - [Module](#module)
- [Public Methods](#public-methods)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API Memory {
public:
    static constexpr std::uint32_t invalid_id = std::numeric_limits<std::uint32_t>::max();
    
    struct Module { ... };
    
    Memory();
    ~Memory() = default;
    
    const std::vector<Memory::Module>& modules() const;
    uint64_t size() const;
    uint64_t free() const;
    uint64_t available() const;

private:
    std::vector<Memory::Module> _modules;
};

} // namespace hwinfo
```

## Nested Structures

### Module

Information about an individual physical memory module (DIMM).

```cpp
struct Module {
    std::uint32_t id = invalid_id;
    std::string vendor;
    std::string name;
    std::string model;
    std::string serial_number;
    uint64_t _size_bytes = 0;
    uint64_t frequency_hz = 0;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `std::uint32_t` | Unique identifier of the memory module |
| `vendor` | `std::string` | Vendor/manufacturer name |
| `name` | `std::string` | Name of the memory module |
| `model` | `std::string` | Model name/number |
| `serial_number` | `std::string` | Serial number of the module |
| `_size_bytes` | `uint64_t` | Total capacity of the module in bytes |
| `frequency_hz` | `uint64_t` | Operating frequency in Hertz |

## Public Methods

### `Memory()`
```cpp
Memory();
```
Constructor that automatically retrieves memory information from the system.

---

### `modules()`
```cpp
const std::vector<Memory::Module>& modules() const;
```
Returns information about all detected physical memory modules (DIMMs).

**Return Value:**
- `const std::vector<Memory::Module>&` - Vector of [Module](#module) structures for each detected memory module

---

### `size()`
```cpp
uint64_t size() const;
```
Returns the total physical memory capacity of the system.

**Return Value:**
- `uint64_t` - Total system RAM in bytes

---

### `free()`
```cpp
uint64_t free() const;
```
Returns the amount of completely free physical memory.

**Return Value:**
- `uint64_t` - Free memory in bytes

**Note:**
- Free memory is not currently in use by any process or the OS kernel
- Convert to MiB: `size / (1024 * 1024)`

---

### `available()`
```cpp
uint64_t available() const;
```
Returns the amount of memory available for allocation to new processes.

**Return Value:**
- `uint64_t` - Available memory in bytes

**Note:**
- Available memory includes free memory plus reclaimable memory from page cache and buffers
- This is generally the value you want to use when checking how much memory is available for use

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    hwinfo::Memory memory;
    
    std::cout << "System Memory Information:" << std::endl;
    
    uint64_t totalMiB = memory.size() / (1024 * 1024);
    uint64_t freeMiB = memory.free() / (1024 * 1024);
    uint64_t availableMiB = memory.available() / (1024 * 1024);
    
    std::cout << "  Total: " << totalMiB << " MiB" << std::endl;
    std::cout << "  Free: " << freeMiB << " MiB" << std::endl;
    std::cout << "  Available: " << availableMiB << " MiB" << std::endl;
    
    const auto& modules = memory.modules();
    std::cout << "\nMemory Modules (" << modules.size() << "):" << std::endl;
    
    for (const auto& module : modules) {
        std::cout << "\nModule " << module.id << ":" << std::endl;
        if (!module.vendor.empty()) {
            std::cout << "  Vendor: " << module.vendor << std::endl;
        }
        if (!module.name.empty()) {
            std::cout << "  Name: " << module.name << std::endl;
        }
        if (!module.model.empty()) {
            std::cout << "  Model: " << module.model << std::endl;
        }
        if (!module.serial_number.empty()) {
            std::cout << "  Serial Number: " << module.serial_number << std::endl;
        }
        uint64_t sizeMiB = module._size_bytes / (1024 * 1024);
        std::cout << "  Size: " << sizeMiB << " MiB" << std::endl;
        if (module.frequency_hz > 0) {
            std::cout << "  Frequency: " << (module.frequency_hz / 1000000) << " MHz" << std::endl;
        }
    }
    
    return 0;
}
```

## Output Example

```
System Memory Information:
  Total: 65437 MiB
  Free: 54405 MiB
  Available: 54405 MiB

Memory Modules (2):

Module 0:
  Vendor: Corsair
  Name: Physical Memory
  Model: CMK32GX4M2Z3600C18
  Serial Number: ***
  Size: 32768 MiB
  Frequency: 3600 MHz

Module 1:
  Vendor: Corsair
  Name: Physical Memory
  Model: CMK32GX4M2Z3600C18
  Serial Number: ***
  Size: 32768 MiB
  Frequency: 3600 MHz
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Total Size | ✔️ | ✔️ | ✔️ |
| Free Size | ✔️ | ✔️ | ✔️ |
| Available Size | ✔️ | ✔️ | ✔️ |
| Module Vendor | ❌ | ❌ | ✔️ |
| Module Model | ❌ | ❌ | ✔️ |
| Module Name | ❌ | ❌ | ✔️ |
| Module Serial Number | ❌ | ❌ | ✔️ |
| Module Frequency | ❌ | ❌ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- On Linux and macOS, module information is not currently implemented, but total/available memory is still available
- On Windows, full DIMM module information is available via SPD

## See Also
- [Monitoring.md](./Monitoring.md) - RAM monitoring API for real-time memory usage tracking
