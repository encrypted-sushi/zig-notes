# Comptime

## StructField: `.type` vs `.default_value_ptr` vs `.is_comptime`

When building a struct field dynamically at comptime, three fields work together:

- `.type` — the label. Tells Zig what type this field is at runtime.
- `.default_value_ptr` — the data. A type-erased pointer to the actual default value.
  Must point to a value of the same type as `.type` before erasure.
- `.is_comptime` — almost always `false`. A `true` field exists only at comptime
  and is erased from the runtime struct entirely.

### Example
```zig
.type = ?[:0]const u8,
.default_value_ptr = @ptrCast(&@as(?[:0]const u8, null)),
.is_comptime = false,
```
`.type` says "this is an optional string".
`@as` gives `null` a concrete type before `@ptrCast` erases it.
Zig uses `.type` to safely recolor the pointer back when needed.
