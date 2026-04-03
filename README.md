# nsx-nanopb

Vendored [nanopb](https://github.com/nanopb/nanopb) v0.4.9.1 — a small,
zero-dynamic-memory Protocol Buffers implementation in ANSI C.

This module provides the **runtime library only** (`pb_encode`, `pb_decode`,
`pb_common`).  Pre-generated `.pb.c` / `.pb.h` files produced from your
`.proto` schemas must be compiled into your own application target, not into
this library.

## Features

- Pure portable C — no Ambiq or SoC dependency
- Zero dynamic memory (`malloc` is never called)
- Deterministic, bounded code size (~3–5 KB flash)
- Suitable for structured messaging over USB, UART, SPI, etc.

## Public headers

```c
#include "pb_encode.h"   // pb_encode(), pb_ostream_from_buffer()
#include "pb_decode.h"   // pb_decode(), pb_istream_from_buffer()
#include "pb.h"          // Low-level types, field descriptors
```

## CMake usage

Add `nsx-nanopb` as a dependency in your `nsx.yml`:

```yaml
modules:
  - nsx-nanopb
```

Then link against the library in your `CMakeLists.txt`:

```cmake
target_link_libraries(my_app PRIVATE nsx_nanopb)
```

The public include path (`includes-api/`) is automatically available to any
target that links `nsx_nanopb`.

## Compile-time options

| Define            | Default | Description                              |
|-------------------|---------|------------------------------------------|
| `PB_NO_ERRMSG`   | `1`     | Omit error-string table (~1 KB savings)  |
| `PB_ENABLE_MALLOC`| *off*  | Enable dynamic allocation (not recommended on MCU) |

Override on your own target if needed:

```cmake
target_compile_definitions(my_app PRIVATE PB_ENABLE_MALLOC=1)
```

## Generating .pb.c / .pb.h from .proto files

Install the nanopb generator (Python) or use the prebuilt binary from the
[nanopb releases](https://github.com/nanopb/nanopb/releases):

```sh
# Using the nanopb protoc plugin
protoc --nanopb_out=. my_message.proto
```

This produces `my_message.pb.c` and `my_message.pb.h`.  Add both to your
application's source list — do **not** add them to this module.

For field-level size constraints (e.g. `max_size`, `max_count`), create a
matching `.options` file alongside your `.proto`:

```
# my_message.options
MyMessage.name   max_size:32
MyMessage.data   max_size:256
```

## Example

See `neuralspotx/examples/usb_rpc/` for a complete USB RPC demo that uses
this module for structured device ↔ host messaging.

## License

Zlib — see the [nanopb upstream repository](https://github.com/nanopb/nanopb)
for the full license text.
