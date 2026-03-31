# Battery API Documentation

The `hwinfo::Battery` class provides information about laptop batteries.

## Table of Contents
- [Class Overview](#class-overview)
- [Enumerations](#enumerations)
  - [State](#state)
- [Public Methods](#public-methods)
- [Global Functions](#global-functions)
- [Example Usage](#example-usage)
- [Platform Support](#platform-support)

## Class Overview

```cpp
namespace hwinfo {

class HWINFO_API Battery {
public:
    static constexpr std::uint32_t invalid_id = std::numeric_limits<std::uint32_t>::max();
    
    enum class State { CHARGING, DISCHARGING, UNKNOWN };
    
    explicit Battery(std::uint32_t = 0);
    ~Battery() = default;
    
    const std::string& vendor() const;
    const std::string& model() const;
    const std::string& serialNumber() const;
    const std::string& technology() const;
    uint32_t energyFull() const;
    double capacity() const;
    std::uint32_t id() const;
    uint32_t energyNow() const;
    bool charging() const;
    bool discharging() const;
    State state() const;

private:
    std::uint32_t _id = invalid_id;
    std::string _vendor = "<unknown>";
    std::string _model = "<unknown>";
    std::string _serial_number = "<unknown>";
    std::string _technology = "<unknown>";
    uint32_t _energyFull = 0;
};

std::vector<Battery> getAllBatteries();

std::ostream& operator<<(std::ostream& os, const Battery::State& state);
std::ostream& operator<<(std::ostream& os, const Battery& battery);

} // namespace hwinfo
```

## Enumerations

### State

Indicates the current charging state of the battery.

```cpp
enum class State {
    CHARGING,
    DISCHARGING,
    UNKNOWN
};
```

| Enum Value | Description |
|------------|-------------|
| `State::CHARGING` | Battery is currently charging |
| `State::DISCHARGING` | Battery is currently discharging (not connected to power) |
| `State::UNKNOWN` | State could not be determined |

## Public Methods

### `Battery()`
```cpp
explicit Battery(std::uint32_t id = 0);
```
Constructor that retrieves information for a specific battery by ID.

**Parameters:**
- `id` - Battery identifier (default: 0)

---

### `id()`
```cpp
std::uint32_t id() const;
```
Returns the unique identifier of the battery.

**Return Value:**
- `std::uint32_t` - Battery ID starting from 0
- `Battery::invalid_id` if the ID is not set

---

### `vendor()`
```cpp
const std::string& vendor() const;
```
Returns the manufacturer/vendor of the battery.

**Return Value:**
- `const std::string&` - Vendor name
- `"<unknown>"` if vendor information is not available

---

### `model()`
```cpp
const std::string& model() const;
```
Returns the model name of the battery.

**Return Value:**
- `const std::string&` - Model name
- `"<unknown>"` if model information is not available

---

### `serialNumber()`
```cpp
const std::string& serialNumber() const;
```
Returns the serial number of the battery.

**Return Value:**
- `const std::string&` - Serial number
- `"<unknown>"` if serial number is not available

---

### `technology()`
```cpp
const std::string& technology() const;
```
Returns the battery technology (chemistry).

**Return Value:**
- `const std::string&` - Technology description (e.g., "Li-ion", "NiMH", etc.)
- `"<unknown>"` if technology information is not available

---

### `energyFull()`
```cpp
uint32_t energyFull() const;
```
Returns the designed/full capacity of the battery (when new).

**Return Value:**
- `uint32_t` - Maximum energy capacity in mWh (milliwatt-hours)

---

### `capacity()`
```cpp
double capacity() const;
```
Returns the current remaining capacity as a fraction of the full capacity.

**Return Value:**
- `double` - Current capacity in range [0, 1]
  - `0.0` = completely worn out
  - `1.0` = capacity same as when new

**Example:**
- `0.85` means the battery currently holds 85% of its original designed capacity

---

### `energyNow()`
```cpp
uint32_t energyNow() const;
```
Returns the current remaining energy in the battery.

**Return Value:**
- `uint32_t` - Current remaining energy in mWh (milliwatt-hours)

---

### `charging()`
```cpp
bool charging() const;
```
Checks if the battery is currently charging.

**Return Value:**
- `true` if battery is charging
- `false` otherwise

---

### `discharging()`
```cpp
bool discharging() const;
```
Checks if the battery is currently discharging.

**Return Value:**
- `true` if battery is discharging
- `false` otherwise

---

### `state()`
```cpp
State state() const;
```
Returns the current charging state of the battery.

**Return Value:**
- `State` - [State](#state) enum value indicating current state

## Global Functions

### `getAllBatteries()`
```cpp
std::vector<Battery> getAllBatteries();
```
Retrieves information for all batteries detected in the system.

**Return Value:**
- `std::vector<Battery>` - Vector containing one `Battery` object per detected battery
- Empty vector if no battery is present (desktop systems)

### `operator<<` for State
```cpp
std::ostream& operator<<(std::ostream& os, const Battery::State& state);
```
Stream output operator that converts `State` enum to human-readable string.

### `operator<<` for Battery
```cpp
std::ostream& operator<<(std::ostream& os, const Battery& battery);
```
Stream output operator that prints battery information to an output stream.

## Example Usage

```cpp
#include <hwinfo/hwinfo.h>
#include <iostream>

int main() {
    std::vector<hwinfo::Battery> batteries = hwinfo::getAllBatteries();
    
    if (batteries.empty()) {
        std::cout << "No batteries detected (desktop system)." << std::endl;
        return 0;
    }
    
    std::cout << "Found " << batteries.size() << " battery(ies):" << std::endl;
    
    for (size_t i = 0; i < batteries.size(); ++i) {
        const hwinfo::Battery& battery = batteries[i];
        std::cout << "\nBattery " << battery.id() << ":" << std::endl;
        std::cout << "  Vendor: " << battery.vendor() << std::endl;
        std::cout << "  Model: " << battery.model() << std::endl;
        if (!battery.serialNumber().empty() && battery.serialNumber() != "<unknown>") {
            std::cout << "  Serial Number: " << battery.serialNumber() << std::endl;
        }
        std::cout << "  Technology: " << battery.technology() << std::endl;
        std::cout << "  Design Capacity: " << battery.energyFull() << " mWh" << std::endl;
        std::cout << "  Current Energy: " << battery.energyNow() << " mWh" << std::endl;
        std::cout << "  Current Capacity: " << (battery.capacity() * 100) << "%" << std::endl;
        std::cout << "  State: " << battery.state() << std::endl;
        std::cout << "  Charging: " << (battery.charging() ? "Yes" : "No") << std::endl;
    }
    
    return 0;
}
```

## Output Example

```
Found 1 battery(ies):

Battery 0:
  Vendor: SMP
  Model: bq401b
  Serial Number: 12345
  Technology: Li-ion
  Design Capacity: 55000 mWh
  Current Energy: 46750 mWh
  Current Capacity: 85%
  State: DISCHARGING
  Charging: No
```

## Platform Support

| Feature | Linux | macOS | Windows |
|---------|:-----:|:-----:|:-------:|
| Vendor | ✔️ | ❌ | ❌ |
| Model | ✔️ | ❌ | ❌ |
| Serial Number | ✔️ | ✔️ | ❌ |
| Technology | ✔️ | ❌ | ❌ |
| Capacity | ✔️ | ✔️ | ❌ |
| Charging State | ✔️ | ✔️ | ❌ |

**Legend:**
- ✔️ = Supported
- ❌ = Not implemented

**Notes:**
- Battery information is primarily for laptops; desktops typically have no battery
- Capacity degradation can be monitored by comparing `energyFull()` with factory capacity
