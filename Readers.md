# Readers
Zig 0.16.0 did a major overhaul of the std.Io system.  
One of the interesting thing is the std.Io.Reader.

If I create a Reader interface, I can make all functions, that take std.Io.Reader as an argument, use it seemlessly.

Writing a Reader is still VERY hazy to me, but the below allows me to cargo-cult.
Below is an example of a std.Io.Reader interface I attempted to create as a Base64 Url Encoded No Padding Reader.

Several things to jot down:
1. This is a file, which by default, is treated as a struct in Zig
2. "reader" field can do some trickery, apparently, but the below can be used as boilerplate
3. Constructor not required but added for personal convenience
4. .stream() is the only thing that needs to be defined

The "Reader" functions provided by std.Io.Reader uses the defined .stream() function to provide everything it supports.  
The function signature, and the first line can also be considered boilderplate.
```zig
const Base64UrlNoPadding = @This();

const std = @import("std");

src: *std.Io.Reader,
reader: std.Io.Reader = .{ // This part is boilerplate for implementing std.Io.Reader
    .vtable = &.{ .stream = stream }, // This is the VTable thing.  if .stream() is called, maps to stream in the struct.
    .buffer = &.{},
    .seek = 0,
    .end = 0,
},

pub fn init(src: *std.Io.Reader) Base64UrlNoPadding {
    return Base64UrlNoPadding{ .src = src };
}

fn stream(r: *std.Io.Reader, w: *std.Io.Writer, limit: std.Io.Limit) std.Io.Reader.StreamError!usize {
    const self: *Base64UrlNoPadding = @fieldParentPtr("reader", r);
    // actual implementation of how to provide the data

    // Determine how many bytes are retquested to the Reader
    const n = limit.minInt(4); // number of bytes requested or 4, whichever is smaller
    if (n == 0) return 0;

    // Determine how many raw bytes (entropy) would be needed to produce the requested bytes
    const raw_len = (n * 3 + 3) / 4; // ceiling division to determine raw bytes of entropy needed

    // Buffers for the raw_read and encrypted vablues
    var raw_buf: [3]u8 = undefined;
    var enc_buf: [4]u8 = undefined;

    //
    self.src.readSliceAll(raw_buf[0..raw_len]) catch return error.ReadFailed;
    _ = std.base64.url_safe_no_pad.Encoder.encode(&enc_buf, raw_buf[0..raw_len]);
    w.writeAll(enc_buf[0..n]) catch return error.WriteFailed;
    return n;
}

```
