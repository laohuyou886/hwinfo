# MainBoard API Documentation

The `hwinfo::MainBoard` class provides information about the system's mainboard (motherboard).

## Table of Contents
- [Class Overview](#class-overview)
- [Public Methods](#public-methods)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API MainBoard {
public:
    MainBoard();
    ~MainBoard() = default;
    
    const std::string& vendor() const;
    const std::string& name() const;
    const std::string& version() const;
    const std::string& serialNumber() const;

private:
    std::string _vendor;
    std::string _name;
    std::string _version;
    std::string _serial_number;
};

} // namespace hwinfo
```

## Public Methods

### `MainBoard()`
```cpp
MainBoard();
```
Constructor that automatically retrieves mainboard information from the system.

---

### `vendor()`
```cpp
const std::string& vendor() const;
```
Returns the manufacturer/vendor of the mainboard.

**Return Value:**
- `const std::string&` - Vendor name (e.g., "ASUSTeK COMPUTER INC.", "Gigabyte Technology Co., Ltd.", "MSI", etc.)

---

### `name()`
```cpp
const std::string& name() const;
```
Returns the model name of the mainboard.

**Return Value:**
- `const std::string&` - Mainboard model name (e.g., "PRIME Z490-A", "X570 AORUS Elite", etc.)

---

### `version()`
```cpp
const std::string& version() const;
```
Returns the version/revision of the mainboard.

**Return Value:**
- `const std::string&` - Version string (e.g., "Rev 1.xx")

---

### `serialNumber()`
```cpp
const std::string& serialNumber() const;
```
Returns the serial number of the mainboard.

**Return Value:**
- `const std::string&` - Mainboard serial number

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    hwinfo::MainBoard mainBoard;
    
    std::cout << "Mainboard Information:" << std::endl;
    std::cout << "  Vendor: " << mainBoard.vendor() << std::endl;
    std::cout << "  Name: " << mainBoard.name() << std::endl;
    std::cout << "  Version: " << mainBoard.version() << std::endl;
    std::cout << "  Serial Number: " << mainBoard.serialNumber() << std::endl;
    
    return 0;
}
```

## Output Example

```
Mainboard Information:
  Vendor: ASUSTeK COMPUTER INC.
  Name: PRIME Z490-A
  Version: Rev 1.xx
  Serial Number: ***
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Vendor | ✔️ | ✔️ | ✔️ |
| Model | ✔️ | ❌ | ✔️ |
| Version | ✔️ | ❌ | ✔️ |
| Serial Number | ❌ | ✔️ | ✔️ |
| BIOS | ❌ | ❌ | ❌ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- BIOS information is currently not implemented on any platform
