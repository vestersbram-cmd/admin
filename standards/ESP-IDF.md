# ESP-IDF Style Guide

This guide covers both ESP-IDF C code and Arduino C++ code for ESP32 development.

## File Structure

### ESP-IDF Project Structure
```
project_name/
├── main/
│   ├── CMakeLists.txt
│   ├── main.c             # App entry point
│   ├── main.h             # Header
│   └── component.mk
├── components/
│   ├── component_a/
│   │   ├── include/
│   │   │   └── component_a.h
│   │   ├── src/
│   │   │   └── component_a.c
│   │   ├── CMakeLists.txt
│   │   └── component.mk
│   └── component_b/
│       └── ...
├── test/
│   └── test_component.c
├── sdkconfig
├── CMakeLists.txt
└── README.md
```

### Arduino Project Structure
```
project_name/
├── project_name.ino       # Main sketch
├── Settings.h             # Board-specific pins
├── include/
│   └── helpers.h
├── src/
│   ├── MyClass.hpp
│   └── MyClass.cpp
└── lib/
    └── (local libraries)
```

## Formatting

- **Indent**: 4 spaces (no tabs)
- **Max line length**: 100 chars
- **Line endings**: LF (Unix-style), never CRLF
- **Braces**: K&R style (opening brace on same line)
- **Trailing whitespace**: Remove on save

## Naming Conventions

| Type | Convention | Example |
|:-----|:-----------|:--------|
| C structs / typedefs | `snake_case` with `_t` suffix | `vsapi_client_t`, `vsapi_err_t` |
| C enum values | `UPPER_SNAKE_CASE` | `VSAPI_ERR_OK`, `VSAPI_ERR_JSON` |
| C functions | `snake_case` | `vsapi_client_init()` |
| C macros / defines | `UPPER_SNAKE_CASE` | `VSAPI_MAX_URL_LEN`, `PIN_BTN_UP` |
| C constants | `UPPER_SNAKE_CASE` | `VSAPI_DEFAULT_TIMEOUT_MS` |
| C++ classes / structs | `CamelCase` | `WifiHandler`, `PrinterDriver` |
| C++ methods / members | `snake_case` | `is_connected`, `out_endpoint_addr` |
| C++ namespaces | `lower_snake_case` | `vsapi::http` |
| Private members (C++) | leading underscore optional | `_display`, `_wifi_multi` |
| Local variables | `snake_case` | `url_len`, `payload` |
| Static globals | `s_` prefix | `s_tag`, `s_transfer` |

## Naming Guidelines

- Use names that describe purpose clearly.
- Prefer consistent prefixes for public C symbols within a component.
- Avoid abbreviations unless the full name would become unwieldy.
- Do not invent restrictions that block ordinary terms such as `json`, `http`,
  `test`, `async`, or `sync`.

## File Organization

### Header Files (.h for C, .hpp for C++)

- Prefer `#pragma once` for public headers.
- For C headers used from C++, add `extern "C"` guards.
- Order includes:
  1. C standard library headers
  2. POSIX / system headers
  3. ESP-IDF headers
  4. Other component headers
  5. Current component headers
  6. Private headers
- Group related declarations with section comments when helpful.

```c
#pragma once

#include <stdbool.h>
#include <stdint.h>

#include "esp_err.h"

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @file vsapi_client.h
 * @brief ESP32-IDF C client for the VSAPI remote API
 */

typedef struct vsapi_client vsapi_client_t;
typedef enum {
    VSAPI_ERR_OK = 0,
    VSAPI_ERR_INIT,
    VSAPI_ERR_JSON,
} vsapi_err_t;

#ifdef __cplusplus
}
#endif
```

### Source Files (.c / .cpp / .ino)

* Keep files focused and easy to navigate.
* Prefer small functions.
* Keep related helpers close to the code that uses them.
* Order includes:

  1. C standard library headers
  2. POSIX / system headers
  3. ESP-IDF / Arduino headers
  4. Third-party headers
  5. Local headers

## Documentation

### ESP-IDF C Code

Use Doxygen for public APIs.

```c
/**
 * @brief Brief description.
 * @param client Client instance.
 * @return Return status.
 */
vsapi_err_t vsapi_function(vsapi_client_t *client);
```

### Arduino C++ Code

* Use short doc comments for public methods and classes.
* Add a brief file header when the file is not self-explanatory.

```cpp
// File: WifiHandler.hpp
// Purpose: Manages Wi-Fi connections with OLED menu interface.

class WifiHandler {
public:
    // Initialize with display reference.
    explicit WifiHandler(Adafruit_SSD1306 *display);

    // Add a known network to the auto-connect list.
    void add_known_network(const char *ssid, const char *password);
};
```

## Code Organization Patterns

### ESP-IDF C Code Structure

* Put internal helpers near the top of the file.
* Keep public API functions grouped together.
* Use `static` for file-local helpers and state.

```c
/* ============================================================================
 * Internal helpers
 * ========================================================================== */

static esp_err_t internal_helper(void)
{
    return ESP_OK;
}

/* ============================================================================
 * Public API
 * ========================================================================== */

esp_err_t public_function(void)
{
    return internal_helper();
}
```

### Arduino C++ Class Structure

```cpp
class MyClass {
public:
    explicit MyClass(Dependency *dep) : _dep(dep) {}

    void begin();
    void process();

private:
    Dependency *_dep;
    int _state = 0;

    void internal_helper();
};
```

## Common Patterns

### Error Handling (ESP-IDF)

* Check `esp_err_t` return values.
* Log failures with `ESP_LOGE` or handle them at the boundary.
* Use `ESP_ERROR_CHECK()` only when failure is unrecoverable.

```c
esp_err_t err = some_operation();
if (err != ESP_OK) {
    ESP_LOGE(TAG, "Operation failed: %s", esp_err_to_name(err));
    return err;
}
```

### Memory Safety

* Check allocations before use.
* Free owned memory on all exit paths.
* Prefer bounded writes and explicit termination.

```c
char *buffer = malloc(size);
if (!buffer) {
    return VSAPI_ERR_INIT;
}

/* Use buffer here. */

free(buffer);
```

### FreeRTOS Task Creation

* Prefer clear task names.
* Keep stack sizes explicit and justified.
* Use `vTaskDelay()` / `vTaskDelayUntil()` for task timing.

```c
xTaskCreate(my_task, "task_name", 4096, arg, priority, NULL);
```

### JSON Handling (cJSON)

* Check parse results.
* Validate types before reading values.
* Always delete the root object.

```c
cJSON *root = cJSON_Parse(json_string);
if (!root) {
    return VSAPI_ERR_JSON;
}

cJSON *item = cJSON_GetObjectItem(root, "key");
if (cJSON_IsString(item)) {
    /* Use item->valuestring. */
}

cJSON_Delete(root);
```

## Includes

### ESP-IDF / C Standard Headers

```c
#include <stdbool.h>
#include <stdint.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
```

### ESP-IDF Framework Headers

```c
#include "esp_err.h"
#include "esp_log.h"
#include "esp_http_client.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
```

### Arduino Headers

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WiFiMulti.h>
#include <Wire.h>
```

## Preferred Libraries

| Purpose         | Library                                   |
| :-------------- | :---------------------------------------- |
| Wi-Fi (Arduino) | `WiFi.h`, `WiFiMulti.h`                   |
| HTTP (ESP-IDF)  | `esp_http_client.h`                       |
| JSON            | `cJSON.h`                                 |
| OLED Display    | `Adafruit_SSD1306.h`, `Adafruit_GFX.h`    |
| USB Host        | `usb/usb_host.h`                          |
| Logging         | `esp_log.h` (ESP-IDF), `Serial` (Arduino) |
| Semaphores      | `freertos/semphr.h`                       |

## Build Configuration

### Arduino

* Put board-specific pins and local defaults in `Settings.h`.
* Use compile-time constants for fixed configuration.
* Document hardware connections in comments.

### ESP-IDF

* Use `sdkconfig` or `menuconfig` for system-level config.
* Use component defaults for overridable values.
* Prefix component macros with the component name.

```c
#define VSAPI_DEFAULT_TIMEOUT_MS 30000
#define VSAPI_MAX_URL_LEN 256
```

## Common Mistakes and Correct Implementations

### 1. Do not use blocking delays in tasks

```cpp
void loop()
{
    static unsigned long last_time = 0;

    if (millis() - last_time >= 1000) {
        last_time = millis();
        /* Do work. */
    }
}
```

### 2. Check ESP error codes

```c
esp_err_t err = esp_wifi_init(&cfg);
if (err != ESP_OK) {
    ESP_LOGE(TAG, "Wi-Fi init failed: %s", esp_err_to_name(err));
    return err;
}
```

### 3. Avoid large stack buffers

```c
void process(void)
{
    char *buffer = malloc(8192);
    if (!buffer) {
        return;
    }

    /* Use buffer. */

    free(buffer);
}
```

### 4. Avoid Arduino String in critical paths

```cpp
char buffer[64];
snprintf(buffer, sizeof(buffer), "Data: %d", sensor_value);
```