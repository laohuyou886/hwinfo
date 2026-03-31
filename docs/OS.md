# OS API Documentation

The `hwinfo::OS` class provides information about the running operating system.

## Table of Contents
- [Class Overview](#class-overview)
- [Public Methods](#public-methods)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API OS {
public:
    OS();
    ~OS() = default;
    
    std::string name() const;
    std::string version() const;
    std::string kernel() const;
    bool is32bit() const;
    bool is64bit() const;
    bool isBigEndian() const;
    bool isLittleEndian() const;

private:
    std::string _name;
    std::string _version;
    std::string _kernel;
    bool _32bit = false;
    bool _64bit = false;
    bool _bigEndian = false;
    bool _littleEndian = false;
};

} // namespace hwinfo
```

## Public Methods

### `OS()`
```cpp
OS();
```
Constructor that automatically retrieves operating system information from the system.

---

### `name()`
```cpp
std::string name() const;
```
Returns the full name of the operating system.

**Return Value:**
- `std::string` - Full OS name (e.g., "Microsoft Windows 11 Professional", "Ubuntu 22.04 LTS", "macOS Ventura")

---

### `version()`
```cpp
std::string version() const;
```
Returns the operating system version.

**Return Value:**
- `std::string` - Operating system version
- Empty string if version information is not available

---

### `kernel()`
```cpp
std::string kernel() const;
```
Returns the kernel version.

**Return Value:**
- `std::string` - Kernel version string
- Empty string if kernel information is not available

---

### `is32bit()`
```cpp
bool is32bit() const;
```
Checks if the operating system is 32-bit.

**Return Value:**
- `true` if running a 32-bit OS
- `false` otherwise

---

### `is64bit()`
```cpp
bool is64bit() const;
```
Checks if the operating system is 64-bit.

**Return Value:**
- `true` if running a 64-bit OS
- `false` otherwise

---

### `isBigEndian()`
```cpp
bool isBigEndian() const;
```
Checks if the system uses big-endian byte order.

**Return Value:**
- `true` if big-endian
- `false` otherwise

---

### `isLittleEndian()`
```cpp
bool isLittleEndian() const;
```
Checks if the system uses little-endian byte order.

**Return Value:**
- `true` if little-endian
- `false` otherwise

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    hwinfo::OS os;
    
    std::cout << "Operating System Information:" << std::endl;
    std::cout << "  Name: " << os.name() << std::endl;
    std::cout << "  Version: " << os.version() << std::endl;
    std::cout << "  Kernel: " << os.kernel() << std::endl;
    std::cout << "  Architecture: " << (os.is64bit() ? "64 bit" : "32 bit") << std::endl;
    std::cout << "  Endianess: " << (os.isLittleEndian() ? "little endian" : "big endian") << std::endl;
    
    return 0;
}
```

## Output Example

```
Operating System Information:
  Name: Microsoft Windows 11 Professional
  Version: <unknown>
  Kernel: <unknown>
  Architecture: 64 bit
  Endianess: little endian
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Name | ✔️ | ✔️ | ✔️ |
| Short Name | ✔️ | ✔️ | ✔️ |
| Version | ✔️ | ✔️ | ❌ |
| Kernel | ✔️ | ✔️ | ❌ |
| Architecture | ✔️ | ✔️ | ✔️ |
| Endianess | ✔️ | ✔️ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- On Windows, version and kernel information are currently not implemented
- Most modern consumer systems use little-endian byte order
