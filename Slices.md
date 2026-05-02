# Slices

Slices are fat pointers that hold the address of the underlying array, and the length in bytes.

There are four "variants" of a slice that I always get lost on:

// 1. var slice, const contents (most common for strings)
```zig
var name: []const u8 = "Alice";
name = "Bob";     // ✅ can repoint
name[0] = 'Z';    // ❌ can't touch the characters
```

// 2. const slice, const contents (fully locked)
const name: []const u8 = "Alice";
name = "Bob";     // ❌ can't repoint
name[0] = 'Z';    // ❌ can't touch the characters

// 3. var slice, mutable contents
var name: []u8 = some_buffer[0..5];
name = other_buffer[0..3];  // ✅ can repoint
name[0] = 'Z';              // ✅ can modify characters

// 4. const slice, mutable contents (rare)
const name: []u8 = some_buffer[0..5];
name = other_buffer[0..3];  // ❌ can't repoint
name[0] = 'Z';              // ✅ can still modify characters


The underlying array of a slice is read-only when []const u8:
[]const u8
  │      │
  │      └── the characters it points to are read-only
  └── this part (pointer + length) can be var or const
      this depends on how the variable is declared:  var | const

The underlying array of a slice is mutable when []u8
[]u8
