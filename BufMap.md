# BufMap

A BufMap is a key/value store datatype that requires an allocator, and stores only string / string pair.
It creates copes of both the key and the value owned by itself.
It frees all allocated and stores keys / values when .deinit() is called.

std.process.Init.envrion_map uses this datatype.

Example use:
```zig
    const environ_map = init.environ_map;
    var it = environ_map.iterator();

    while (it.next()) |item| {
        const key = item.key_ptr.*;
        const value = item.value_ptr.*;
        try stdout_writer.print("{s} = {s}\n", .{ key, value });
    }
    try stdout_writer.flush(); // Don't forget to flush!
```
