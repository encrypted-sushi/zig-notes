# Writers
Zig 0.16.0 did a major overhaul of the std.Io system.  
One of the interesting thing is the std.Io.Writer.

Writing a Writer is still VERY hazy to me, but the below allows me to cargo-cult.
Below is an example of a std.Io.Writer interface I attempted to create as a passthrough to another Writer.

Several things to jot down:
1. .drain() is the only thing that needs to be defined
2. .init() not required, but created due to personal preference

```zig
The "Writer" functions provided by std.Io.Writer uses the defined .drain() function to provide everything it supports.  
The function signature, and the first line can also be considered boilderplate.

const PassthroughWriter = struct {
    downstream: *Io.Writer,
    interface: Io.Writer,

    pub fn init(downstream: *Io.Writer, buffer: []u8) PassthroughWriter {
        return .{
            .downstream = downstream,
            .interface = .{
                .vtable = &.{ .drain = drain },
                .buffer = buffer,
            },
        };
    }

    fn drain(io_w: *Io.Writer, data: []const []const u8, splat: usize) Io.Writer.Error!usize {
        const self: *PassthroughWriter = @alignCast(@fieldParentPtr("interface", io_w));
        return self.downstream.vtable.drain(self.downstream, data, splat);
    }
};
```
