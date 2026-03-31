# Disk API Documentation

The `hwinfo::Disk` class provides information about storage devices (hard disks, SSDs, NVMe, etc.) detected in the system.

## Table of Contents
- [Class Overview](#class-overview)
- [Enumerations](#enumerations)
  - [Interface](#interface)
- [Public Methods](#public-methods)
- [Global Functions](#global-functions)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API Disk {
public:
    enum class Interface { ... };
    
    static constexpr std::uint32_t invalid_id = std::numeric_limits<std::uint32_t>::max();
    
    ~Disk() = default;
    
    const std::string& vendor() const;
    const std::string& model() const;
    const std::string& serial_number() const;
    std::uint64_t size() const;
    const std::vector<std::string>& mount_points() const;
    std::uint32_t id() const;
    Interface disk_interface() const;

private:
    Disk() = default;
    // ... private members
};

std::vector<Disk> getAllDisks();

std::ostream& operator<<(std::ostream& os, const Disk::Interface& disk_interface);

} // namespace hwinfo
```

## Enumerations

### Interface

Indicates the storage interface type used by the disk.

```cpp
enum class Interface {
    NVME,
    USB,
    USB1,
    USB2,
    USB3_5GBit,
    USB3_10GBit,
    USB3_20GBit,
    USB4_20GBit,
    USB4_40GBit,
    USB4_80GBit,
    SATA,
    SCSI,
    UNKNOWN
};
```

| Enum Value | Description |
|------------|-------------|
| `Interface::NVME` | NVMe PCIe interface |
| `Interface::USB` | Generic USB interface |
| `Interface::USB1` | USB 1.x (12 Mbit/s) |
| `Interface::USB2` | USB 2.0 (480 Mbit/s) |
| `Interface::USB3_5GBit` | USB 3.0 (5 Gbit/s) |
| `Interface::USB3_10GBit` | USB 3.1 (10 Gbit/s) |
| `Interface::USB3_20GBit` | USB 3.2 (20 Gbit/s) |
| `Interface::USB4_20GBit` | USB4 (20 Gbit/s) |
| `Interface::USB4_40GBit` | USB4 (40 Gbit/s) |
| `Interface::USB4_80GBit` | USB4 (80 Gbit/s) |
| `Interface::SATA` | SATA interface |
| `Interface::SCSI` | SCSI interface |
| `Interface::UNKNOWN` | Unknown interface type |

## Public Methods

### `id()`
```cpp
std::uint32_t id() const;
```
Returns the unique identifier of the disk.

**Return Value:**
- `std::uint32_t` - Disk ID starting from 0
- `Disk::invalid_id` if the ID is not set

---

### `vendor()`
```cpp
const std::string& vendor() const;
```
Returns the manufacturer/vendor of the disk.

**Return Value:**
- `const std::string&` - Vendor name (e.g., "WD_BLACK", "Samsung", "Kingston", etc.)
- `"<unknown>"` if vendor information is not available

---

### `model()`
```cpp
const std::string& model() const;
```
Returns the model name of the disk.

**Return Value:**
- `const std::string&` - Model name (e.g., "SN850 Heatsink 1TB", "EVO 860 500GB", etc.)
- `"<unknown>"` if model information is not available

---

### `serial_number()`
```cpp
const std::string& serial_number() const;
```
Returns the serial number of the disk.

**Return Value:**
- `const std::string&` - Disk serial number
- `"<unknown>"` if serial number is not available

---

### `size()`
```cpp
std::uint64_t size() const;
```
Returns the total capacity of the disk in bytes.

**Return Value:**
- `std::uint64_t` - Total disk size in bytes

---

### `mount_points()`
```cpp
const std::vector<std::string>& mount_points() const;
```
Returns the list of mount points for this disk (volumes).

**Return Value:**
- `const std::vector<std::string>&` - Vector of mount point paths
- On Windows: Typically drive letters (e.g., `C:\`, `D:\`)
- On Linux/macOS: Filesystem mount paths (e.g., `/`, `/home`)

---

### `disk_interface()`
```cpp
Interface disk_interface() const;
```
Returns the storage interface type of the disk.

**Return Value:**
- `Interface` - [Interface](#interface) enum value indicating the connection type

## Global Functions

### `getAllDisks()`
```cpp
std::vector<Disk> getAllDisks();
```
Retrieves information for all disks detected in the system.

**Return Value:**
- `std::vector<Disk>` - Vector containing one `Disk` object per physical disk detected

### `operator<<` for Interface
```cpp
std::ostream& operator<<(std::ostream& os, const Disk::Interface& disk_interface);
```
Stream output operator that converts `Interface` enum to human-readable string.

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    std::vector<hwinfo::Disk> disks = hwinfo::getAllDisks();
    
    std::cout << "Found " << disks.size() << " disk(s):" << std::endl;
    
    for (size_t i = 0; i < disks.size(); ++i) {
        const hwinfo::Disk& disk = disks[i];
        std::cout << "\nDisk " << disk.id() << ":" << std::endl;
        std::cout << "  Vendor: " << disk.vendor() << std::endl;
        std::cout << "  Model: " << disk.model() << std::endl;
        std::cout << "  Serial Number: " << disk.serial_number() << std::endl;
        
        double sizeGB = static_cast<double>(disk.size()) / (1000.0 * 1000.0 * 1000.0);
        std::cout << "  Size: " << sizeGB << " GB" << std::endl;
        
        std::cout << "  Interface: " << disk.disk_interface() << std::endl;
        
        if (!disk.mount_points().empty()) {
            std::cout << "  Mount Points:" << std::endl;
            for (const auto& mp : disk.mount_points()) {
                std::cout << "    " << mp << std::endl;
            }
        }
    }
    
    return 0;
}
```

## Output Example

```
Found 5 disk(s):

Disk 0:
  Vendor: (Standard disk drives)
  Model: WD_BLACK SN850 Heatsink 1TB
  Serial Number: ***
  Size: 1000.2 GB
  Interface: NVME
  Mount Points:
    C:\

Disk 1:
  Vendor: (Standard disk drives)
  Model: Intenso SSD Sata III
  Serial Number: ***
  Size: 120.0 GB
  Interface: SATA
  Mount Points:
    D:\

Disk 2:
  Vendor: (Standard disk drives)
  Model: KINGSTON SA400S37240G
  Serial Number: ***
  Size: 240.0 GB
  Interface: SATA
  Mount Points:
    E:\
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Vendor | ✔️ | ✔️ | ✔️ |
| Model | ✔️ | ✔️ | ✔️ |
| Serial Number | ✔️ | ✔️ | ✔️ |
| Size | ✔️ | ✔️ | ✔️ |
| Free Size | ✔️ | ✔️ | ✔️ |
| Volumes/Mount Points | ✔️ | ✔️ | ✔️ |
| Interface Type Detection | ✔️ | ✔️ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- Free size information is available through the [monitoring API](./Monitoring.md#disk-monitoring)

## See Also
- [Monitoring.md](./Monitoring.md#disk-monitoring) - Disk monitoring for real-time free space checking
