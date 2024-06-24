# Embedded Test Example

An example for using [embedded-test](https://github.com/probe-rs/embedded-test) with the esp32s3 (xtensa).

## Prerequisites

Install a recent version of the Xtensa Rust ecosystem:

```sh
cargo install espup
espup install --toolchain-version 1.77.0.0
```

(The toolchain version must match the toolchain channel in rust-toolchain.toml.)

Install a recent version of [probe-rs](https://github.com/probe-rs/probe-rs), such as revision `a6dd038`:

```sh
cargo install probe-rs-tools --git https://github.com/probe-rs/probe-rs --rev a6dd038 --force --locked
```

In the terminal window where you're running the tests, source export-esp.sh for your current toolchain:

```sh
. $HOME/.rustup/toolchains/esp-1.77.0.0/share/export-esp.sh
```

(You must do this in each terminal window where you intend to run tests.)

Connect an ESP32-S3 device like a ESP32-S3-DevKitC-1.

Erase the device:

```sh
espflash erase-flash --port …
```

## Running Tests

Invoke `cargo test`:

```sh
cargo test --test example_test --release
```

You should see output like the following (otherwise, see the Troubleshooting section below):

```
> cargo test --test example_test --release
   Compiling example-embedded-test v0.1.0 (/Users/myk/Projects/embedded-test-example)
    Finished release [optimized + debuginfo] target(s) in 1.06s
     Running tests/example_test.rs (/Users/myk/Projects/target/xtensa-esp32s3-espidf/release/deps/example_test-5944b6a0a7fa66bd)
      Erasing ✔ [00:00:00] [################################################################################################] 192.00 KiB/192.00 KiB @ 269.96 KiB/s (eta 0s )
  Programming ✔ [00:00:00] [###################################################################################################] 31.26 KiB/31.26 KiB @ 40.88 KiB/s (eta 0s )    Finished in 1.493s

running 8 tests
test tests::takes_state      ... ok
test tests::log              ... INFO  [example_test::tests] Hello, log!
ok
test tests::it_works_ignored ... ignored
test tests::it_fails1        ... Frame 0: syscall_readonly @ 0x42003a9b inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/syscall/xtensa.rs:29:9
Frame 1: sys_exit_extended @ 0x0000000042003a9b inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/mod.rs:196:9
Frame 2: exit @ 0x0000000042003a8f inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/mod.rs:165:5
Frame 3: exit @ 0x0000000042003a8f
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/process.rs:14:5
Frame 4: <unknown function @ 0x82003abe> @ 0x82003abe
FAILED
test tests::it_fails2        ... Frame 0: syscall_readonly @ 0x42003a9b inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/syscall/xtensa.rs:29:9
Frame 1: sys_exit_extended @ 0x0000000042003a9b inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/mod.rs:196:9
Frame 2: exit @ 0x0000000042003a8f inline
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/sys/arm_compat/mod.rs:165:5
Frame 3: exit @ 0x0000000042003a8f
       /Users/myk/.cargo/registry/src/index.crates.io-6f17d22bba15001f/semihosting-0.1.11/src/process.rs:14:5
Frame 4: <unknown function @ 0x82003abe> @ 0x82003abe
FAILED
test tests::it_passes        ... ok
test tests::it_fails3        ... FAILED
test tests::it_timeouts      ... Frame 0: it_timeouts @ 0x42000eb7 inline
       /Users/myk/Projects/embedded-test-example/tests/example_test.rs:86:9
Frame 1: {closure#7} @ 0x0000000042000eb7 inline
       /Users/myk/Projects/embedded-test-example/tests/example_test.rs:5:1
Frame 2: call_once<example_test::tests::__embedded_test_start::{closure_env#7}, ()> @ 0x0000000042000eb4
       /Users/myk/.rustup/toolchains/esp/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
Frame 3: <unknown function @ 0x82003171> @ 0x82003171
FAILED

failures:

---- tests::it_fails1 ----
Test should Pass but it did Panic

---- tests::it_fails2 ----
Test should Pass but it did Panic

---- tests::it_fails3 ----
Test should Panic but it did Pass

---- tests::it_timeouts ----
Test timed out after 10s


failures:
    tests::it_fails1
    tests::it_fails2
    tests::it_fails3
    tests::it_timeouts

test result: FAILED. 3 passed; 4 failed; 1 ignored; 0 measured; 0 filtered out; finished in 12.07s

Error: Some tests failed
```

## Troubleshooting

### Could not clear all hardware breakpoints: An Xtensa specific error occurred

If probe-rs reports generic Xtensa-specific errors:

```
 WARN probe_rs::session: Could not clear all hardware breakpoints: An Xtensa specific error occurred.

Caused by:
    0: Xtensa debug module error
    1: Error writing register 0x47
    2: Register-specific error
 WARN probe_rs::session: Failed to deconfigure device during shutdown: An Xtensa specific error occurred.

Caused by:
    The result index of a batched command is not available.
Error: Failed to attach to RTT

Caused by:
    Error attempting to attach to RTT: Error communicating with probe: An Xtensa specific error occurred.
```

Try running `cargo test` again. Alternately, try disconnecting/reconnecting the connection between the device and your computer.

### Failed to verify flash algorithm. Data mismatch at address 0x40380400

If probe-rs reports flashing failures:

```
 WARN probe_rs::config::sequences::esp: Unknown flash capacity byte: 55
ERROR probe_rs::flashing::flasher: Failed to verify flash algorithm. Data mismatch at address 0x40380400
ERROR probe_rs::flashing::flasher: Original instruction: 0x40001f74
ERROR probe_rs::flashing::flasher: Readback instruction: 0x80809180
…
Error: The flashing procedure failed for '/Users/myk/Projects/example-embedded-test/target/xtensa-esp32s3-espidf/release/deps/example_test-9c99aaa88de0e7bc'.

Caused by:
    The RAM contents did not match the expected contents after loading the flash algorithm.
```

Try re-erasing the device. Also see [Flashing esp32-s3 with probe-rs causes flash algorithm verification fail](https://github.com/probe-rs/probe-rs/issues/2338).

### Fewer IRs detected than TAPs

If you see an error about fewer IRs being detected than TAPs:

```
ERROR probe_rs::probe::common: Fewer IRs detected than TAPs
Error: Connecting to the chip was unsuccessful.

Caused by:
    0: An error with the usage of the probe occurred
    1: Invalid IR scan chain
    2: Invalid IR scan chain
```

Your device may be connected to your computer via a breakout board and ESP-Prog board but not powered via that connection. Ensure it's powered, for example by connecting it to a source of USB power (other than your computer, or you might see the next error).

Alternately, your device may be connected to your computer via USB as well as a breakout board and ESP-Prog board, which can cause this error even when the ESP-Prog board isn't connected to your computer. Try disconnecting the device from the breakout board.

### 2 probes were found

If you see an error about two probes being found:

```
Error: 2 probes were found:
    1. ESP JTAG (VID: 303a, PID: 1001, Serial: 30:30:F9:51:4C:A8, EspJtag)
    2. ESP JTAG (VID: 303a, PID: 1001, Serial: F4:12:FA:C5:81:40, EspJtag)
```

Or:

```
Error: 2 probes were found:
    1. Dual RS232-HS (VID: 0403, PID: 6010, Serial: 90040079FCFC, FTDI)
    2. ESP JTAG (VID: 303a, PID: 1001, Serial: F4:12:FA:C5:81:40, EspJtag)
```

You have two devices connected, or you have one device connected in two ways (such as via both USB and a breakout board to ESP-Prog board). Disconnect one, as probe-rs doesn't support picking a connection (at least not when running indirectly via `cargo test`).

### Failed to open the debug probe

If you see an error about failing to open the debug probe:

```
     Running tests/example_test.rs (/Users/myk/Projects/target/xtensa-esp32s3-espidf/debug/deps/example_test-d7a62cd75bb33918)
Error: Failed to open the debug probe.

Caused by:
    0: The debug probe could not be created.
    1: A USB error occurred.
    2: Could not be opened for exclusive access
```

You may have another probe-rs process actively connected. Try disconnecting it.

## References

* https://github.com/probe-rs/probe-rs
* https://github.com/probe-rs/embedded-test
* https://github.com/probe-rs/embedded-test-example
* https://github.com/esp-rs/esp-hal/tree/main/hil-test
