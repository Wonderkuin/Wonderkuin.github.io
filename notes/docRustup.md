# Rust Windows 环境搭建

## [Rust Style Guidelines](https://doc.rust-lang.org/1.0.0/style/README.html)

---

```
RUSTUP_DIST_SERVER
https://mirrors.ustc.edu.cn/rust-static
RUSTUP_UPDATE_ROOT
https://mirrors.ustc.edu.cn/rust-static/rustup
```

```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
```

```
简单命令
rustup show
rustc --version
cargo --version

cargo new hello-rust
cd ./hello-rust
cargo run
```

```
 __________________________
< Hello fellow Rustaceans! >
 --------------------------
        \
         \
            _~^~^~_
        \) /  o o  \ (/
          '_   -   _'
          / '-----' \
```

```
rustup update
rustup self uninstall

rustc main.rs

cargo build
cargo run
cargo check
cargo build --release
```

```
# 本地lib
[dependencies]
bevy = { path = "../bevy" }
```

---