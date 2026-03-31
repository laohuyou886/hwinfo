# Network API Documentation

The `hwinfo::Network` class provides information about network interfaces.

## Table of Contents
- [Class Overview](#class-overview)
- [Public Methods](#public-methods)
- [Global Functions](#global-functions)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API Network {
public:
    ~Network() = default;
    
    const std::string& interfaceIndex() const;
    const std::string& description() const;
    const std::string& mac() const;
    const std::string& ip4() const;
    const std::string& ip6() const;

private:
    Network() = default;
    // ... private members
};

std::vector<Network> getAllNetworks();

} // namespace hwinfo
```

## Public Methods

### `interfaceIndex()`
```cpp
const std::string& interfaceIndex() const;
```
Returns the interface index/number of this network interface.

**Return Value:**
- `const std::string&` - Interface index identifier

---

### `description()`
```cpp
const std::string& description() const;
```
Returns the description/name of the network interface adapter.

**Return Value:**
- `const std::string&` - Interface description (e.g., "Intel(R) Gigabit Ethernet", "Wi-Fi", etc.)

---

### `mac()`
```cpp
const std::string& mac() const;
```
Returns the MAC (Media Access Control) address of the network interface.

**Return Value:**
- `const std::string&` - MAC address in colon-separated hex format (e.g., "00:1A:2B:3C:4D:5E")

---

### `ip4()`
```cpp
const std::string& ip4() const;
```
Returns the IPv4 address of the network interface.

**Return Value:**
- `const std::string&` - IPv4 address in dot-decimal notation (e.g., "192.168.1.100")
- Empty string if no IPv4 address is assigned

---

### `ip6()`
```cpp
const std::string& ip6() const;
```
Returns the IPv6 address of the network interface.

**Return Value:**
- `const std::string&` - IPv6 address in colon-separated hex notation
- Empty string if no IPv6 address is assigned

## Global Functions

### `getAllNetworks()`
```cpp
std::vector<Network> getAllNetworks();
```
Retrieves information for all network interfaces detected in the system.

**Return Value:**
- `std::vector<Network>` - Vector containing one `Network` object per network interface detected

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    std::vector<hwinfo::Network> networks = hwinfo::getAllNetworks();
    
    std::cout << "Found " << networks.size() << " network interface(s):" << std::endl;
    
    for (const auto& iface : networks) {
        std::cout << "\nInterface " << iface.interfaceIndex() << ":" << std::endl;
        std::cout << "  Description: " << iface.description() << std::endl;
        std::cout << "  MAC Address: " << iface.mac() << std::endl;
        if (!iface.ip4().empty()) {
            std::cout << "  IPv4: " << iface.ip4() << std::endl;
        }
        if (!iface.ip6().empty()) {
            std::cout << "  IPv6: " << iface.ip6() << std::endl;
        }
    }
    
    return 0;
}
```

## Output Example

```
Found 3 network interface(s):

Interface 0:
  Description: Intel(R) Wi-Fi 6 AX200
  MAC Address: 00:1A:2B:3C:4D:5E
  IPv4: 192.168.1.100
  IPv6: 2001:db8::1

Interface 1:
  Description: Ethernet
  MAC Address: 00:1A:2B:3C:4D:5F
  IPv4: 192.168.1.101

Interface 2:
  Description: Bluetooth Network Connection
  MAC Address: 00:1A:2B:3C:4D:60
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Interface Index | ✔️ | ✔️ | ✔️ |
| Description | ✔️ | ✔️ | ✔️ |
| MAC Address | ✔️ | ✔️ | ✔️ |
| IPv4 Address | ✔️ | ✔️ | ✔️ |
| IPv6 Address | ✔️ | ✔️ | ✔️ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- Includes all active network interfaces including Ethernet, Wi-Fi, Bluetooth, VPN, etc.
- Loopback interfaces are typically included
