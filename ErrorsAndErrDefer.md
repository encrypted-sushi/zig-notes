# Errors

## What is an Error
An errors in Zig is just just a name.  It does not contain any messages nor data.
The simplest form of an "error" looks like this: error.Name
```zig
fn doSomething() !void {
    return error.SomethingWentWrong;
}
```

Because errors are simply values, it
- can be stored in variables
- can be passed to functions
- can be switched on exhaustively
- is tracked by the compiler, about which errors are returned by a given function
- will cause error handling code to fail if a new error is added to a error set, until all error handlers are updated to handle the additional error


## Error Sets
A set of errors can be defined.  
This is usefull if we want the handle various errors sepratately, because switch will require all errors to be handled.
```zig
const MyError = error{
    SomethingWentWrong,
};
```

## Handling Errors
Errors are handled in 3 different ways in Zig

A. Handling errors with "try"
This "bubbles up" the error received. 
```zig
try doSomething();
```

B. Handling all errors with "catch"
This is a catch-all that handles all errors singlehandedly
```zig
doSomething() catch |err| {
    std.debug.print("got error: {}\n", .{err});
};
```

C. Handling errors with "catch", and using "switch"
This is the pattern to handle various errors differently
```zig
doSomething() catch |err| switch (err) {
    error.SomethingWentWrong => std.debug.print("oh no!\n", .{}),
    else => return err, // propagate anything unexpected
};
```

NOTE: "try" is basically this
```zig
doSomething() catch |err| return err
```


## errdefer — defer only on error

`errdefer` is like `defer`, but only runs if the function returns an error.
If the function succeeds, it is ignored.

```zig
const thing = try allocate_something();  // acquire
errdefer free_something(thing);          // declare cleanup RIGHT HERE

// if anything below errors: errdefer fires, thing is freed
// if everything succeeds:   errdefer ignored, caller owns thing
```

The pattern: **acquire, then immediately declare the error cleanup.**
Co-locate the allocation with its error-path cleanup — never separate them.

```zig
// real example -- multiple allocations, each protected:
const a = try allocator.dupe(u8, str_a);
errdefer allocator.free(a);

const b = try allocator.dupe(u8, str_b);
errdefer allocator.free(b);  // if this fails, `a` is also freed by its errdefer

// for optional allocations:
errdefer if (maybe) |val| allocator.free(val);
```

Without `errdefer` you would need boolean flags to track what was allocated.
`errdefer` replaces all of that — no flags, no manual tracking, no mistakes.

