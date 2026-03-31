# OpenCL Device API Documentation

The OpenCL device API provides detailed information about OpenCL-capable compute devices (GPUs and CPUs). This is only available when `HWINFO_GPU_OPENCL` CMake option is enabled.

**Header:** `<hwinfo/opencl/device.h>`

## Table of Contents
- [Device Class](#device-class)
  - [Enumerations](#enumerations)
  - [Public Methods](#public-methods)
- [DeviceManager Class](#devicemanager-class)
  - [Filter Enumeration](#filter-enumeration)
  - [Public Methods](#public-methods_1)
- [Example Usage](#example-usage)
- [Building with OpenCL](#building-with-opencl)

## Device Class

```cpp
namespace opencl_ {

class Device {
public:
    enum Type { GPU, CPU };

    Device(uint32_t id, cl::Device cl_device);
    ~Device();

    Device(const Device& device) = delete;
    Device& operator=(const Device& device) = delete;

    Device(Device&& device) noexcept;
    Device& operator=(Device&& device) noexcept;

    uint32_t get_id() const;
    const cl::Device& get_cl_device() const;
    cl::Device& get_cl_device();
    std::string name() const;
    std::string vendor() const;
    std::string driver_version() const;
    std::string opencl_c_version() const;
    uint64_t memory_Bytes() const;
    uint64_t memory_used_Bytes() const;
    uint64_t global_cache_Bytes() const;
    uint64_t local_cache_Bytes() const;
    uint64_t max_global_buffer_Bytes() const;
    uint64_t max_constant_buffer_Bytes() const;
    uint64_t compute_units() const;
    uint64_t cores() const;
    uint64_t clock_frequency_MHz() const;
    Type type() const;
    uint64_t fp64() const;
    uint64_t fp32() const;
    uint64_t fp16() const;
    uint64_t int64() const;
    uint64_t int32() const;
    uint64_t int16() const;
    uint64_t int8() const;
    uint64_t estimated_flops() const;
    bool intel_gt_4gb_buffer_required() const;
};

std::ostream& operator<<(std::ostream& os, const Device& device);
std::ostream& operator<<(std::ostream& os, const Device::Type& type);

} // namespace opencl_
```

### Enumerations

#### Type

Indicates the type of OpenCL device.

```cpp
enum Type {
    GPU,
    CPU
};
```

| Enum Value | Description |
|------------|-------------|
| `Type::GPU` | GPU device |
| `Type::CPU` | CPU device |

### Public Methods

#### `Device()`
```cpp
Device(uint32_t id, cl::Device cl_device);
```
Constructor that creates a Device from an existing OpenCL device.

**Parameters:**
- `id` - Unique identifier for this device
- `cl_device` - OpenCL `cl::Device` object

**Note:** It's recommended to use `DeviceManager` instead of constructing devices manually.

---

#### `get_id()`
```cpp
uint32_t get_id() const;
```
Returns the unique identifier of the device.

**Return Value:**
- `uint32_t` - Device ID

---

#### `get_cl_device()` (const)
```cpp
const cl::Device& get_cl_device() const;
```
Returns the underlying OpenCL device object.

**Return Value:**
- `const cl::Device&` - Reference to the underlying OpenCL device

---

#### `get_cl_device()` (non-const)
```cpp
cl::Device& get_cl_device();
```
Returns the underlying OpenCL device object (non-const).

**Return Value:**
- `cl::Device&` - Reference to the underlying OpenCL device

---

#### `name()`
```cpp
std::string name() const;
```
Returns the name of the device.

**Return Value:**
- `std::string` - Device name

---

#### `vendor()`
```cpp
std::string vendor() const;
```
Returns the vendor of the device.

**Return Value:**
- `std::string` - Vendor name

---

#### `driver_version()`
```cpp
std::string driver_version() const;
```
Returns the driver version.

**Return Value:**
- `std::string` - Driver version string

---

#### `opencl_c_version()`
```cpp
std::string opencl_c_version() const;
```
Returns the OpenCL C version supported by the device.

**Return Value:**
- `std::string` - OpenCL C version string

---

#### `memory_Bytes()`
```cpp
uint64_t memory_Bytes() const;
```
Returns the total global memory size in bytes.

**Return Value:**
- `uint64_t` - Total memory in bytes

---

#### `memory_used_Bytes()`
```cpp
uint64_t memory_used_Bytes() const;
```
Returns the amount of memory currently used by OpenCL allocations.

**Return Value:**
- `uint64_t` - Used memory in bytes

**Note:** This is only accurate if you use the mcl memory management.

---

#### `global_cache_Bytes()`
```cpp
uint64_t global_cache_Bytes() const;
```
Returns the size of the global cache in bytes.

**Return Value:**
- `uint64_t` - Global cache size in bytes

---

#### `local_cache_Bytes()`
```cpp
uint64_t local_cache_Bytes() const;
```
Returns the size of the local (per compute unit) cache in bytes.

**Return Value:**
- `uint64_t` - Local cache size in bytes

---

#### `max_global_buffer_Bytes()`
```cpp
uint64_t max_global_buffer_Bytes() const;
```
Returns the maximum size of a single global memory buffer.

**Return Value:**
- `uint64_t` - Maximum global buffer size in bytes

---

#### `max_constant_buffer_Bytes()`
```cpp
uint64_t max_constant_buffer_Bytes() const;
```
Returns the maximum size of a constant buffer.

**Return Value:**
- `uint64_t` - Maximum constant buffer size in bytes

---

#### `compute_units()`
```cpp
uint64_t compute_units() const;
```
Returns the number of compute units.

**Return Value:**
- `uint64_t` - Number of compute units

---

#### `cores()`
```cpp
uint64_t cores() const;
```
Returns the estimated total number of compute cores.

**Return Value:**
- `uint64_t` - Total number of cores

**Note:** This is estimated based on vendor-specific detection.

---

#### `clock_frequency_MHz()`
```cpp
uint64_t clock_frequency_MHz() const;
```
Returns the clock frequency in MHz.

**Return Value:**
- `uint64_t` - Clock frequency in MHz

---

#### `type()`
```cpp
Type type() const;
```
Returns the device type (GPU or CPU).

**Return Value:**
- `Type` - [Type](#type) enum value

---

#### `fp64()`
```cpp
uint64_t fp64() const;
```
Returns the native vector width for 64-bit floating point.

**Return Value:**
- `uint64_t` - Native vector width
- `0` if double precision is not supported

---

#### `fp32()`
```cpp
uint64_t fp32() const;
```
Returns the native vector width for 32-bit floating point.

**Return Value:**
- `uint64_t` - Native vector width

---

#### `fp16()`
```cpp
uint64_t fp16() const;
```
Returns the native vector width for 16-bit floating point.

**Return Value:**
- `uint64_t` - Native vector width
- `0` if half precision is not supported

---

#### `int64()`
```cpp
uint64_t int64() const;
```
Returns the native vector width for 64-bit integers.

**Return Value:**
- `uint64_t` - Native vector width

---

#### `int32()`
```cpp
uint64_t int32() const;
```
Returns the native vector width for 32-bit integers.

**Return Value:**
- `uint64_t` - Native vector width

---

#### `int16()`
```cpp
uint64_t int16() const;
```
Returns the native vector width for 16-bit integers.

**Return Value:**
- `uint64_t` - Native vector width

---

#### `int8()`
```cpp
uint64_t int8() const;
```
Returns the native vector width for 8-bit integers.

**Return Value:**
- `uint64_t` - Native vector width

---

#### `estimated_flops()`
```cpp
uint64_t estimated_flops() const;
```
Returns the estimated peak FLOPS in FLOPS/second.

**Return Value:**
- `uint64_t` - Estimated peak FLOPS

**Note:** This is an estimation based on cores × frequency × instructions per cycle. Actual performance will be lower.

---

#### `intel_gt_4gb_buffer_required()`
```cpp
bool intel_gt_4gb_buffer_required() const;
```
Checks if this is an Intel GT device that requires buffer addresses to be below 4GB.

**Return Value:**
- `true` if the device is an Intel IGPs requiring 4GB address limitation
- `false` otherwise

---

## DeviceManager Class

The `DeviceManager` singleton provides easy access to OpenCL devices with filtering capabilities.

```cpp
namespace opencl_ {

enum class Filter {
    MAX_MEMORY,
    MIN_MEMORY,
    MAX_FLOPS,
    MIN_FLOPS,
    GPU,
    CPU,
    ID,
    ALL
};

class DeviceManager {
public:
    template <Filter T>
    static Device* get();

    template <Filter T>
    static std::vector<Device*> get_list();

    template <Filter T>
    static Device* get(uint32_t value);

private:
    DeviceManager();
    static DeviceManager& get_instance();

    std::vector<Device> _devices;
};

} // namespace opencl_
```

### Filter Enumeration

| Filter | Description |
|--------|-------------|
| `Filter::MAX_MEMORY` | Device with maximum memory |
| `Filter::MIN_MEMORY` | Device with minimum memory |
| `Filter::MAX_FLOPS` | Device with maximum estimated FLOPS |
| `Filter::MIN_FLOPS` | Device with minimum estimated FLOPS |
| `Filter::GPU` | All GPU devices |
| `Filter::CPU` | All CPU devices |
| `Filter::ID` | Device by specific ID |
| `Filter::ALL` | All devices |

### Public Methods

#### `get()` (single device by filter)
```cpp
template <Filter T>
static Device* get();
```
Get a single device matching the filter.

**Supported for:** `MAX_MEMORY`, `MIN_MEMORY`, `MAX_FLOPS`, `MIN_FLOPS`

**Return Value:**
- `Device*` - Pointer to the matching device
- `nullptr` if no devices found

---

#### `get_list()` (multiple devices by filter)
```cpp
template <Filter T>
static std::vector<Device*> get_list();
```
Get a list of devices matching the filter.

**Supported for:** `GPU`, `CPU`, `ALL`

**Return Value:**
- `std::vector<Device*>` - Vector of pointers to matching devices

---

#### `get()` (device by ID)
```cpp
template <Filter T>
static Device* get(uint32_t value);
```
Get a specific device by ID.

**Parameters:**
- `value` - Device ID

**Supported for:** `ID`

**Return Value:**
- `Device*` - Pointer to the device with the given ID
- `nullptr` if not found

## Example Usage

```cpp
#include <hwinfo/opencl/device.h>
#include <iostream>

int main() {
    // Get the GPU with the most memory
    opencl_::Device* gpu = opencl_::DeviceManager::get<opencl_::Filter::MAX_MEMORY>();

    if (!gpu) {
        std::cout << "No OpenCL GPU found." << std::endl;
        return 1;
    }

    std::cout << "Best OpenCL GPU:" << std::endl;
    std::cout << "  ID: " << gpu->get_id() << std::endl;
    std::cout << "  Name: " << gpu->name() << std::endl;
    std::cout << "  Vendor: " << gpu->vendor() << std::endl;
    std::cout << "  Driver: " << gpu->driver_version() << std::endl;

    double memoryGiB = static_cast<double>(gpu->memory_Bytes()) / (1024.0 * 1024.0 * 1024.0);
    std::cout << "  Memory: " << memoryGiB << " GiB" << std::endl;

    std::cout << "  Compute Units: " << gpu->compute_units() << std::endl;
    std::cout << "  Cores: " << gpu->cores() << std::endl;
    std::cout << "  Clock: " << gpu->clock_frequency_MHz() << " MHz" << std::endl;

    double flopsGF = static_cast<double>(gpu->estimated_flops()) / 1e9;
    std::cout << "  Estimated FLOPS: " << flopsGF << " GFLOPS" << std::endl;

    std::cout << "  Supports FP64: " << (gpu->fp64() > 0 ? "Yes" : "No") << std::endl;
    std::cout << "  Supports FP16: " << (gpu->fp16() > 0 ? "Yes" : "No") << std::endl;

    // Get all GPUs
    std::vector<opencl_::Device*> allGPUs = opencl_::DeviceManager::get_list<opencl_::Filter::GPU>();
    std::cout << "\nFound " << allGPUs.size() << " OpenCL GPU(s) total." << std::endl;

    return 0;
}
```

## Building with OpenCL

To enable OpenCL support, configure CMake with:

```bash
cmake -B build -DHWINFO_GPU_OPENCL=ON
cmake --build build
```

The OpenCL dependency must be available on your system. When enabled as a submodule, hwinfo will automatically fetch OpenCL-CLHPP.

**CMake option:**
- `HWINFO_GPU_OPENCL` - Enable OpenCL GPU information (default: `OFF`)

## Notes

- OpenCL support is optional and disabled by default
- The OpenCL API is in the `opencl_` namespace (not `hwinfo`)
- Only one `DeviceManager` singleton is created and it enumerates devices once at first access
- Memory usage tracking (`memory_used_Bytes()`) requires using the mcl memory management system
