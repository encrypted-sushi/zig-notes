# Structs
Structs are a collection of data types in one "thing".

Structs can have functions defined in them, like methods of a Class.


## Constructor (the init() function)
Some structs define .init() while other may not.  The ones that don't seem to be those that are simply setting values of the struct fields.

The .init(), if it exists or written, shall return a copy of the struct, because most structs lives on the stack.
```zig
pub const Rectangle = struct {
    length: u32,
    width: u32,

    pub fn init(length: u32, width: u32) Rectangle {
        return Rectangle { .length = length, .width = width };
    }
};  
```

If I want to explicitly create the struct on the heap, I would use an allocator.
```zig
allocator.create(Rectangle);
```

## Functions (methods?)
A function declared in a struct has 3 "forms":
1. If it is a constructor, the struct is not in the list of arguments.
```zig
pub fn init(args)
```
2. If the function modifies the struct, the first argument should be a pointer to the struct.  By convention "self" seem to be used idiomatically.
```zig
pub fn foo(self: *Rectangle)
```
3. If the function only reads from the struct, the first argument should be a copy.
```zig
pub fn bar(self: Rectangle)
```


## Allocators: To hold, or not to hold a pointer to an allocator
Structs should store a pointer to the passed in allocator if dynamic allocations are needed during the lifetime of its existence.
```zig
pub const DynamicHashMap = struct {
    allocator: std.mem.Allocator
}
```

If the struct only allocates memory during .init(), then do not store the allocator.  Instead, make sure that the destructor (.deinit()) also takes an allocator as its argument.  
NOTE: The destructor usually only reads the struct, and deallocates.  This is why the first argument is not a pointer to the struct.
```zig
pub fn deinit(self: DynamicHashMap, allocator: std.mem.Allocator)
```

## Destructors
If I want to iterate over the fields of a struct in a destructor, Zig provides a comptime trick that allows this.
```zig
pub fn deinit(self: XdgDirs, allocator: std.mem.Allocator) void {
    inline for (@typeInfo(XdgDirs).@"struct".fields) |field| {
        if (field.field_type == []const u8) {
            allocator.free(@field(self, field.name));
        }
    }
    if (self.runtime_dir) |dir| allocator.free(dir);
}
```
