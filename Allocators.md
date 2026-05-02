# Allocators

In Zig, any heap allocation must be done using an allocator.  
Any function that needs to allocate heap memory must take a std.mem.Allocator as an argument.  

This follows Zig's one simple rule about memory allocation: 
> Every heap allocation is explicit, and the caller chooses the allocator.

std.mem.Allocator is just an interface — a pointer to some implementation plus a vtable of function pointers.  
The actual memory strategy is chosen by the caller, who supplies their preferred allocator.  

## General Purpose Allocator
**as a newb, use this**
Safe, detects leaks in debug mode.
Juicy Main give us a gpa pre-set
```zig
puf fn main(init: std.process.Init) !void {
    gpa = init.gpa;
}
```

If not using Juicy Main
```zig
var gpa = std.heap.DebugAllocator(.{}).init;
const gpa = gpa.allocator();
```

## Arena Allocator
**incredibly fast, no manual intervention**
This allocates freely, frees everything at once at the end.  
Perfect for "allocate a bunch of stuff, use it, throw it all away."
NOTE: even with Juicy Main, alllocator() must be called for arena allocator (init.arena is a *ArenaAllocator, as opposed to std.mem.Allocator for init.gpa)
```zig
puf fn main(init: std.process.Init) !void {
    // arena example -- no individual frees needed!
    const allocator = init.arena.allocator();   <= calling allocator()
    const a = try allocator.dupe(u8, "hello");
    const b = try std.fmt.allocPrint(allocator, "{s} world", .{a});
    // no free() calls needed -- arena frees everything at once
}
```

If not using Juicy Main, things get a little hairier, because the Arena allocator apparently needs a backing allocator.
```zig
// GPA
var gpa = std.heap.DebugAllocator(.{}).init;
const allocator = gpa.allocator();  // ← need this

// Arena
var arena = std.heap.ArenaAllocator.init(gpa.allocator());
const allocator = arena.allocator();  // ← need this too
```

## Debug Allocator
**use this if debugging heap allocated memory issues**
```zig
var gpa = std.heap.DebugAllocator(.{}).init;
const allocator = gpa.allocator();  // ← need this
```

## Testing Allocator
**not sure when I would use this, but presumably when running unit tests?**
```zig
test "my test" {
    const allocator = std.testing.allocator;
    // automatically reports leaks when test ends
    const s = try allocator.dupe(u8, "hello");
    defer allocator.free(s);
}
```

## How to use the allocators to allocate heap memory
These should apparently cover 95% of use cases
```
dupe          ← copy an existing slice
alloc         ← allocate a new slice
create        ← allocate a single struct/value
allocPrint    ← allocate a formatted string
free          ← release alloc/dupe/allocPrint memory
destroy       ← release create memory
```

so, the thing to remember is
```
alloc/dupe/realloc/allocPrint  => deallocate with allocator.free()  
create                         => deallocate with allocator.destroy()  
```

### allocator.alloc(T, n)
**allocate a slice of n items of type T**
```zig
const buf = try allocator.alloc(u8, 1024);  // []u8 of 1024 bytes
defer allocator.free(buf);
```

### allocator.create(T)
**allocate a single item, returns a pointer**
```zig
const node = try allocator.create(MyStruct);  // *MyStruct
defer allocator.destroy(node);  // note: destroy, not free!
node.* = .{ .value = 42 };
```

### allocator.realloc(slice, new_len)
**resize an existing allocation**
```zig
var buf = try allocator.alloc(u8, 64);
buf = try allocator.realloc(buf, 128);  // grow it
defer allocator.free(buf);
```

### 
