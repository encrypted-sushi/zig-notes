# Tagged Unions

## Unions
A regular Union can hold one of the several values that are defined.
```zig
const MyUnion = union {
    int: i64,    // OR
    float: f64,  // one of these, never both
};
// size = 8 bytes (just the largest one)
```

It does NOT hold both an int and a float in the above example.  
That is what Structs are for.
```zig
const MyStruct = struct {
    int: i64,    // exists
    float: f64,  // also exists, at the same time
};
// size = 16 bytes (8 + 8)
```


## Tagged Unions
A tagged union is a union, that secretly carries an enum.
```zig
const MyUnion = union(enum) {
    int: i64,
    float: f64,
};
```

The (enum) tells Zig this: automatically create an enum with labels matching my field names, and store it alongside the data.
This becomes as if the below were done.
```zig
// what zig is secretly doing behind the scenes:
const Tag = enum { int, float };  // auto-generated

const MyUnion = struct {
    tag: Tag,   // which one is active?
    data: ...   // the actual value
};
```

## Switch on Tagged Unions
Because the various types supported by the union are "tagged" with an enum, switching on the tag names can be done.
```zig
const val = MyUnion{ .int = 42 };

switch (val) {
    .int   => |n| std.debug.print("it's an int: {d}\n", .{n}),
    .float => |f| std.debug.print("it's a float: {f}\n", .{f}),
```

