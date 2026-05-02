# Optionals

## What is an Optional?
An "optional" is an "item" that can be two things:
- null
- a value

In its simplest form, an optional is this:
```zig
var name: ?[]const u8 = null;   // no value yet
name = "Alice";                 // now it has a value
```

## Unwrapping
There are 3 ways to "unwrap" an optional (take value if exist)
A. IF -> Only run a block if a value exists
```zig
if (maybeO) |value| {
    std.debug.print("{s}\n", .{value});
}
```

B. ORELSE -> Use value if it exists, otherwise alternative specified value
```zig
const result = maybe orelse "default";
```

C. .? -> forcibly unwraps, but panics if the optional is null (only for quick testing purposes)
```zig
const result = maybe.?;  // crashes if maybe is null
```

## Example use cases
Return an error if an optional is null
```zig
const home = environ_map.get("HOME") orelse return error.HomeDirNotSet;
```
