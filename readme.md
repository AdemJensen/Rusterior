# Rusterior Game Engine
This is a working-in-progress 3D game engine, ECS based, 
written with Rust lang.

# Dev log for now
We haven't found a decent way to link dynamic libraries in cargo. 
So far, we have tried: 
- running cargo with env variable RUSTFLAGS='-C prefer-dynamic'
- adding build.rs and set env variables
- adding build.rs and set -C flags directly

None of the above works... But we do have a workable rustc command set:

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

Heads up, you need std-e33da769ef0fc5d8.dll, which is the build output
for crate 'prefer-dynamic' in crates.io, to run main.exe.

We are trying some new statements here:

```bash
cargo rustc -- -C prefer-dynamic
```

This statement won't work.
