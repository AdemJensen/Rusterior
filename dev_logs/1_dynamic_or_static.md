# Dev log 1: Dynamic linking or static linking

This stage was instructed by video https://www.bilibili.com/video/BV1DG4y1Z7q2/?p=4&vd_source=9db47c759b7a33fb827409095dd0b1cc

When creating project structure, I have considered both dynamic linking and static 
linking. But as I dig further, I realized that the Rust does not provide very good 
support on dynamic linking.

## How to build engine into dll?
That's easy, just set `crate-type = dylib` and it will be compiled into a dll.

## How to link the dll into our sandbox program?
We have 2 ways to do that:

Option 1: Using `rustc` command to compile it manually:

```bash
# this command creates dynamic library lib.dll
# rustc engine\src\lib.rs --crate-type dylib  # this command triggers failures
rustc --crate-name engine engine\src\lib.rs --crate-type dylib -C prefer-dynamic

# this command creates static library libtest_static.rlib
rustc --crate-name test_static test_static\src\lib.rs --crate-type rlib

# this command creates main.exe, with one statically linked rlib
# and one dynamically linked dylib
rustc sandbox\src\main.rs --crate-type bin --extern engine='engine.dll' --extern test_static='libtest_static.rlib'
```

Option 2: Using `cargo build` with env `RUSTFLAGS` set to `-C prefer-dynamic`:

```bash
# On windows:
$env:RUSTFLAGS='-C prefer-dynamic'
cargo build

# On linux or OSX:
RUSTFLAGS='-C prefer-dynamic' cargo build
```

Btw, if the `RUSTFLAGS` variable was not set, it will return:

```
error: cannot satisfy dependencies so `std` only shows up once                                                                                                                                                                         
  |
  = help: having upstream crates all available in one format will likely make this go away
```

This problem seems to be a known issue, and if the dynamic linking is on, 
the game must also ship `libstd.dll`, like the Bevy does:

> //! # Warning<br/>
> //!<br/>
> //! Do not enable this feature for release builds because this would require you to ship<br/>
> //! `libstd.so` and `libbevy_dylib.so` with your game.

So I finally decide to make the engine as a static crate, and change to dynamic
linking when the Rust lang got its dynamic linking strategy improved.
