# clay-ntp

A brick for [Clay](https://github.com/mseminatore/clay) that asks an NTP server for the time and
tells you how far your clock is off.

```clay
import("ntp")

r = Ntp query("pool.ntp.org", 123, 3000)
("your clock is off by " + r get("offset") toString + " ms") println
```

It **measures** the clock; it never changes it (setting the system clock needs operating-system
privileges, and `chrony`, `ntpd` or Windows Time already do that well). To use corrected time in a
program, add the offset yourself:

```clay
offset = (Ntp query("pool.ntp.org", 123, 3000)) get("offset")
trueNow = Date nowMillis + offset       // Unix time in ms, corrected
```

A long-running program can query once at startup, and again from time to time.

Requires Clay 0.3 or newer (the brick uses the bitwise operators).

## Install

```sh
clay bricks add github.com/mseminatore/clay-ntp ntp
```

This also installs the [hex](https://github.com/mseminatore/clay-hex) brick, which `ntp` uses
(`Hex` becomes available to your program too).

## API

| Call | Returns |
|---|---|
| `Ntp query(host, port, timeoutMs)` | asks the server and returns the Dict described below. Port 123 is the standard one. |
| `Ntp request` | the 48-byte request packet (a fresh random nonce each time) |
| `Ntp parse(reply, request, sentMs, receivedMs)` | checks a server's reply to `request` and returns the Dict. `sentMs` and `receivedMs` are your clock's time (`Date nowMillis`) just before sending and just after receiving, as integers |

`query` is `request`, a UDP round trip and `parse`; use the other two to do the network part
yourself.

### The result

| Key | Meaning |
|---|---|
| `time` | the server's time, in milliseconds since 1970 |
| `offset` | how far **behind** the server your clock is, in ms (negative if you are ahead). Add it to `Date nowMillis` to correct |
| `delay` | the round-trip network delay, in ms |
| `stratum` | the server's distance from a reference clock (1 = directly attached; at most 15) |
| `leap` | the leap-second warning: 0 none, 1 a second will be added, 2 a second will be removed |

All times are whole milliseconds. The offset is accurate to about the network delay, so it is
good for knowing that you are 3 seconds off, not for microsecond work. A tiny negative `delay`
(a ms or two) can happen from rounding; a large one means the clock changed during the exchange,
so query again.

### Errors

`query` lets network errors through unchanged: `TimeoutError` when no reply comes within
`timeoutMs`, and `SocketError` when the host cannot be found. `parse` raises a `RuntimeError`
that says what was wrong:

- `ntp parse: reply too small` / `request too small` — fewer than 48 bytes
- `ntp parse: not a server reply`, `incorrect version` — not an NTP version 3 or 4 server packet
- `ntp parse: request/reply mismatch` — the reply does not echo this request's nonce (a stray or
  forged packet)
- `ntp parse: kiss of death RATE` — the server refused you; the code (`RATE`, `DENY`, ...) is
  in the message. Wait before asking again, and do not poll a public server more than once every
  few minutes
- `ntp parse: unsynchronized` — the server's own clock is not set
- `ntp parse: invalid stratum` — stratum above 15
- `ntp parse: expects a String (got Number)`, `sentMs must be an integer number of milliseconds`
  — wrong kind of argument

## Notes

- This is SNTP: one request, one reply. It does not average several servers, so it will not
  match a full NTP client.
- The reply is checked for your request's random nonce, but not for which address it came from.
- Dates in 2036 and later are read correctly (NTP's 32-bit seconds roll over in 2036).

## Testing

```sh
clay tests/test_ntp.clay       # a brick can import itself by name
clay examples/ntp_time.clay    # needs the network: asks a real server
```

The tests build replies by hand, so they need no network. `.github/workflows/test.yml` runs them on
Linux, macOS and Windows for every push and pull request on GitHub (it builds Clay from source;
pin `CLAY_REF` to a release once you depend on one). On another Git host, delete it.

## License

MIT (see `LICENSE`).
