# CPU API Documentation

The `hwinfo::CPU` class provides information about CPU (Central Processing Unit) devices detected in the system.

## Table of Contents
- [Class Overview](#class-overview)
- [Nested Structures](#nested-structures)
  - [Cache](#cache)
  - [Core](#core)
- [Public Methods](#public-methods)
- [Global Functions](#global-functions)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API CPU {
public:
    static constexpr std::uint32_t invalid_id = std::numeric_limits<std::uint32_t>::max();
    
    struct Cache { ... };
    struct Core { ... };
    
    ~CPU() = default;
    
    std::uint32_t id() const;
    const std::string& modelName() const;
    const std::string& vendor() const;
    std::uint64_t numPhysicalCores() const;
    std::uint64_t numLogicalCores() const;
    const std::vector<std::string>& flags() const;
    const std::vector<Core>& cores() const;

private:
    CPU() = default;
    // ... private members
};

std::vector<CPU> getAllCPUs();

} // namespace hwinfo
```

## Nested Structures

### Cache

Cache information for a CPU core.

```cpp
struct Cache {
    std::uint64_t l1_data;        // L1 data cache size in bytes
    std::uint64_t l1_instruction; // L1 instruction cache size in bytes
    std::uint64_t l2;             // L2 cache size in bytes
    std::uint64_t l3;             // L3 cache size in bytes
};
```

| Field | Type | Description |
|-------|------|-------------|
| `l1_data` | `std::uint64_t` | Size of L1 data cache in bytes |
| `l1_instruction` | `std::uint64_t` | Size of L1 instruction cache in bytes |
| `l2` | `std::uint64_t` | Size of L2 cache in bytes |
| `l3` | `std::uint64_t` | Size of L3 cache in bytes |

### Core

Information about an individual CPU core.

```cpp
struct Core {
    std::uint64_t id;
    Cache cache;
    std::uint64_t regular_frequency_hz;
    std::uint64_t max_frequency_hz;
    bool smt;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `std::uint64_t` | Unique identifier of the core |
| `cache` | `Cache` | Cache information for this core |
| `regular_frequency_hz` | `std::uint64_t` | Default/regular clock frequency in Hertz |
| `max_frequency_hz` | `std::uint64_t` | Maximum turbo frequency in Hertz |
| `smt` | `bool` | Indicates whether Simultaneous Multithreading is enabled |

## Public Methods

### `id()`
```cpp
std::uint32_t id() const;
```
Returns the unique identifier of the CPU socket.

**Return Value:**
- `std::uint32_t` - CPU socket ID starting from 0
- `CPU::invalid_id` if the ID is not set

---

### `modelName()`
```cpp
const std::string& modelName() const;
```
Returns the model name of the CPU as reported by the system.

**Return Value:**
- `const std::string&` - CPU model name (e.g., "Intel(R) Core(TM) i7-10700K CPU @ 3.80GHz")

---

### `vendor()`
```cpp
const std::string& vendor() const;
```
Returns the vendor identifier string.

**Return Value:**
- `const std::string&` - CPU vendor string (e.g., "GenuineIntel", "AuthenticAMD")

---

### `numPhysicalCores()`
```cpp
std::uint64_t numPhysicalCores() const;
```
Returns the number of physical cores available on this CPU.

**Return Value:**
- `std::uint64_t` - Number of physical cores

**Note:**
- A physical core is a hardware core distinct from logical threads

---

### `numLogicalCores()`
```cpp
std::uint64_t numLogicalCores() const;
```
Returns the number of logical threads/cores available.

**Return Value:**
- `std::uint64_t` - Number of logical cores/threads

**Note:**
- When hyper-threading/SMT is enabled, this value is typically double the number of physical cores

---

### `flags()`
```cpp
const std::vector<std::string>& flags() const;
```
Returns CPU feature flags supported by the CPU.

**Return Value:**
- `const std::vector<std::string>&` - List of CPU feature flags (e.g., "sse", "avx", "avx2", "avx512f")

---

### `cores()`
```cpp
const std::vector<Core>& cores() const;
```
Returns detailed information for each physical core.

**Return Value:**
- `const std::vector<Core>&` - Vector of [Core](#core) structures with detailed per-core information

## Global Functions

### `getAllCPUs()`
```cpp
std::vector<CPU> getAllCPUs();
```
Retrieves information for all CPUs detected in the system.

**Return Value:**
- `std::vector<CPU>` - Vector containing one `CPU` object per CPU socket detected

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    std::vector<hwinfo::CPU> cpus = hwinfo::getAllCPUs();
    
    std::cout << "Found " << cpus.size() << " CPU socket(s):" << std::endl;
    
    for (size_t i = 0; i < cpus.size(); ++i) {
        const hwinfo::CPU& cpu = cpus[i];
        std::cout << "\nSocket " << cpu.id() << ":" << std::endl;
        std::cout << "  Vendor: " << cpu.vendor() << std::endl;
        std::cout << "  Model: " << cpu.modelName() << std::endl;
        std::cout << "  Physical Cores: " << cpu.numPhysicalCores() << std::endl;
        std::cout << "  Logical Cores: " << cpu.numLogicalCores() << std::endl;
        
        std::cout << "  Cores:" << std::endl;
        for (const auto& core : cpu.cores()) {
            std::cout << "    Core " << core.id << ":" << std::endl;
            std::cout << "      L3 Cache: " << core.cache.l3 << " bytes" << std::endl;
            std::cout << "      Default Frequency: " << core.regular_frequency_hz << " Hz" << std::endl;
            std::cout << "      Max Frequency: " << core.max_frequency_hz << " Hz" << std::endl;
            std::cout << "      SMT Enabled: " << (core.smt ? "Yes" : "No") << std::endl;
        }
        
        std::cout << "  Supported flags: ";
        for (const auto& flag : cpu.flags()) {
            std::cout << flag << " ";
        }
        std::cout << std::endl;
    }
    
    return 0;
}
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Vendor | ✔️ | ✔️ | ✔️ |
| Model | ✔️ | ✔️ | ✔️ |
| Physical Cores | ✔️ | ✔️ | ✔️ |
| Logical Cores | ✔️ | ✔️ | ✔️ |
| Cache Size | ✔️ | ✔️ | ✔️ |
| Frequency | ✔️ | ❌ | ✔️ |
| Flags | ✔️ | ✔️ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

## See Also
- [Monitoring.md](./Monitoring.md) - CPU monitoring API for real-time utilization and frequency
