# GPU API Documentation

The `hwinfo::GPU` class provides information about GPU (Graphics Processing Unit) devices detected in the system.

## Table of Contents
- [Class Overview](#class-overview)
- [Public Methods](#public-methods)
- [Global Functions](#global-functions)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API GPU {
public:
    static constexpr std::uint32_t invalid_id = std::numeric_limits<std::uint32_t>::max();
    
    ~GPU() = default;
    
    const std::string& vendor() const;
    const std::string& name() const;
    const std::string& driverVersion() const;
    uint64_t dedicated_memory_Bytes() const;
    uint64_t shared_memory_Bytes() const;
    uint64_t frequency_hz() const;
    std::uint64_t num_cores() const;
    std::uint32_t id() const;
    const std::string& vendor_id() const;
    const std::string& device_id() const;

private:
    GPU() = default;
    // ... private members
};

std::vector<GPU> getAllGPUs();

} // namespace hwinfo
```

## Public Methods

### `id()`
```cpp
std::uint32_t id() const;
```
Returns the unique identifier of the GPU.

**Return Value:**
- `std::uint32_t` - GPU ID starting from 0
- `GPU::invalid_id` if the ID is not set

---

### `vendor()`
```cpp
const std::string& vendor() const;
```
Returns the vendor name of the GPU.

**Return Value:**
- `const std::string&` - GPU vendor name (e.g., "NVIDIA", "AMD", "Intel", "Microsoft", etc.)

---

### `name()`
```cpp
const std::string& name() const;
```
Returns the model name of the GPU.

**Return Value:**
- `const std::string&` - GPU model name (e.g., "NVIDIA GeForce RTX 3070 Ti")

---

### `driverVersion()`
```cpp
const std::string& driverVersion() const;
```
Returns the version of the installed GPU driver.

**Return Value:**
- `const std::string&` - Driver version string

---

### `dedicated_memory_Bytes()`
```cpp
uint64_t dedicated_memory_Bytes() const;
```
Returns the amount of dedicated VRAM in bytes.

**Return Value:**
- `uint64_t` - Dedicated video memory size in bytes

**Note:**
- Dedicated memory is memory that belongs exclusively to the GPU
- Convert to MiB: `size / (1024 * 1024)` to get mebibytes

---

### `shared_memory_Bytes()`
```cpp
uint64_t shared_memory_Bytes() const;
```
Returns the amount of shared system memory allocated to the GPU in bytes.

**Return Value:**
- `uint64_t` - Shared system memory size in bytes

**Note:**
- Shared memory is system RAM that can be used by the GPU when dedicated memory is full
- This is primarily used by integrated graphics solutions

---

### `frequency_hz()`
```cpp
uint64_t frequency_hz() const;
```
Returns the current GPU clock frequency in Hertz.

**Return Value:**
- `uint64_t` - GPU core clock frequency in Hz
- `0` if frequency information is not available

---

### `num_cores()`
```cpp
std::uint64_t num_cores() const;
```
Returns the number of GPU compute units/cores.

**Return Value:**
- `std::uint64_t` - Number of GPU compute units/cores

---

### `vendor_id()`
```cpp
const std::string& vendor_id() const;
```
Returns the PCI vendor ID of the GPU.

**Return Value:**
- `const std::string&` - PCI vendor ID as hex string

---

### `device_id()`
```cpp
const std::string& device_id() const;
```
Returns the PCI device ID of the GPU.

**Return Value:**
- `const std::string&` - PCI device ID as hex string

## Global Functions

### `getAllGPUs()`
```cpp
std::vector<GPU> getAllGPUs();
```
Retrieves information for all GPUs detected in the system.

**Return Value:**
- `std::vector<GPU>` - Vector containing one `GPU` object per GPU detected

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    std::vector<hwinfo::GPU> gpus = hwinfo::getAllGPUs();
    
    std::cout << "Found " << gpus.size() << " GPU(s):" << std::endl;
    
    for (size_t i = 0; i < gpus.size(); ++i) {
        const hwinfo::GPU& gpu = gpus[i];
        std::cout << "\nGPU " << gpu.id() << ":" << std::endl;
        std::cout << "  Vendor: " << gpu.vendor() << std::endl;
        std::cout << "  Model: " << gpu.name() << std::endl;
        std::cout << "  Driver Version: " << gpu.driverVersion() << std::endl;
        
        uint64_t dedicatedMiB = gpu.dedicated_memory_Bytes() / (1024 * 1024);
        std::cout << "  Dedicated Memory: " << dedicatedMiB << " MiB" << std::endl;
        
        if (gpu.shared_memory_Bytes() > 0) {
            uint64_t sharedMiB = gpu.shared_memory_Bytes() / (1024 * 1024);
            std::cout << "  Shared Memory: " << sharedMiB << " MiB" << std::endl;
        }
        
        if (gpu.frequency_hz() > 0) {
            std::cout << "  Frequency: " << (gpu.frequency_hz() / 1000000) << " MHz" << std::endl;
        }
        
        if (gpu.num_cores() > 0) {
            std::cout << "  Number of Cores: " << gpu.num_cores() << std::endl;
        }
        
        std::cout << "  PCI Vendor ID: " << gpu.vendor_id() << std::endl;
        std::cout << "  PCI Device ID: " << gpu.device_id() << std::endl;
    }
    
    return 0;
}
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Vendor | ✔️ | ❌ | ✔️ |
| Model | ✔️ | ❌ | ✔️ |
| Memory Size | ❌ | ❌ | ✔️ |
| Driver Version | ✔️ | ❌ | ✔️ |
| Core Count | ✔️ | ❌ | ✔️ |
| Frequency | ✔️ | ❌ | ✔️ |
| Vendor ID | ✔️ | ❌ | ✔️ |
| Device ID | ✔️ | ❌ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

## See Also
- [OpenCL.md](./OpenCL.md) - OpenCL device information for additional GPU details
