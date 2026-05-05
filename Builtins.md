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

"The reason this exists is for a very specific pattern — type-erased interfaces. When you have a pointer to just one field (like a writer or reader interface), but you need to get back to the full struct that owns it."

## @embedFile("path")
**bake a file content into binary at compile time**
```zig
const default_config = @embedFile("defaults.json");
```

## @sizeOf(T)
**size of a type in bytes, comptime**
```zig
@sizeOf(u32) // 4



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

## @as(T, value)
**treat this value as type T**  
When assigning a value to a struct field of a particular type, we sometimes need to explicitly make it clear.
For example, if I want to assign null to a field that expects a pointer to a spec"ific type, I need a mechanism to say "This null, means a 'null' for this specific pointer that it expects"
So far, only encountered using this in conjuction with @ptrCast(), explained next:
```zig
attrs[i] = .{ .default_value_ptr = @ptrCast(&@as(?[]const u8, null)) };
```

## @ptrCast(ptr, value)
**erase this pointer to \*anyopaque**
Using the same example as above...  
```zig
attrs[i] = .{ .default_value_ptr = @ptrCast(&@as(?[]const u8, null)) };
```
At first glance, this looked like a "huh?" to me.  
There's a _really_ good reason for this.

Before we "erase" the pointer type, we cast it AS something...   why?  
"null" has no size. 
@as(?[]const u8, null) says "this is a null that occupies exactly the space of ?[]const u8".  
Now, with & (address of) provided, the compiler will know "Ok, I need to read x bytes to get the entire data".  

A slice in Zig is what's called a "fat pointer".  
It doesn't just provide the memory address.  
It also provides length in bytes.  
So \*anyopaque, able to take any kind of pointer, and:
 - can be casted back later
 - can hold the ptr temporarily
 - until the user "re-converts" it back into what type of data it was supposed to have been pointing to, maybe?


