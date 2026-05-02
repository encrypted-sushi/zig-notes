# Useful (and encountered) Buildins

## @This()
**The current type/file**  
I use this when using the "file as a struct" so that I can refer to fields and functions with StructName.*
```zig
const XdgDirs = @This();
```

## @typeInfo(T)
**Inspect a type T at comptime**
```zig
inline for (@typeInfo(XdgDirs).@"struct".fields) |field| {
```

## @field(value, "name")
**Runtime-driven field access**
Useful for deallocating in a loop, like this:
```zig
pub fn deinit(self: XdgDirs, allocator: std.mem.Allocator) void {
    inline for (@typeInfo(XdgDirs).@"struct".fields) |field| {
        if (field.type == []const u8) {
            allocator.free(@field(self, field.name));
        }
    }
    if (self.runtime_dir) |dir| allocator.free(dir);
}
```

## @fieldParentPtr("field_name", field_ptr)
**Gets Struct for the given field_name and field_ptr provided**
This is used for what's known as "type-erased interfaces".
Long story short, this builtin allows the Reader/Writer interfaces to work its magic.


## @embedFile("path")
**bake a file content into binary at compile time**
```zig
const default_config = @embedFile("defaults.json");
```

## @sizeOf(T)
**size of a type in bytes, comptime**
```zig
@sizeOf(u32) // 4
```

## @TypeOf(value)
**get the type of a value**
```zig
const x = 42;
const T = @TypeOf(x); // T is comptime_int
```

## @intFromEnum(val) and @enumFromInt(n)
**convert between enums and integers**
```zig
const n = @intFromEnum(MyEnum.some_variant); // 0, 1, 2...
```

## @panic("message")
**immediate crash with a message**
Use this to crash the program

## @compileError("message")
**Custom compilation error**
causes a compile error with a message
```zig
if (some_comptime_condition) {
    @compileError("you can't use this type here");
}
```

