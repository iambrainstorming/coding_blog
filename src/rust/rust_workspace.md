
# Rust workspace 

Sure! Let's create a simple example to demonstrate how to build a specific package in a Cargo workspace. 

1. **Set up a Cargo workspace**:
    - Create a new directory for your workspace.
    - Inside this directory, create a `Cargo.toml` file for the workspace.
    - Create subdirectories for each package.

Here’s an example structure:

```
my_workspace/
│
├── Cargo.toml
├── package_a/
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
└── package_b/
    ├── Cargo.toml
    └── src/
        └── main.rs
```

2. **Workspace `Cargo.toml`**:

In the root of your workspace (`my_workspace/Cargo.toml`):

```toml
[workspace]
members = [
    "package_a",
    "package_b",
]
```

3. **Package `Cargo.toml` files**:

For `package_a` (`my_workspace/package_a/Cargo.toml`):

```toml
[package]
name = "package_a"
version = "0.1.0"
edition = "2021"

[dependencies]
```

For `package_b` (`my_workspace/package_b/Cargo.toml`):

```toml
[package]
name = "package_b"
version = "0.1.0"
edition = "2021"

[dependencies]
```

4. **Main.rs files**:

For `package_a` (`my_workspace/package_a/src/main.rs`):

```rust
fn main() {
    println!("Hello from package_a!");
}
```

For `package_b` (`my_workspace/package_b/src/main.rs`):

```rust
fn main() {
    println!("Hello from package_b!");
}
```

5. **Building a specific package**:

To build only `package_a`, navigate to the workspace root (`my_workspace`) and run:

```sh
cargo build -p package_a --release
```

This command will compile only `package_a` in release mode, ignoring `package_b`. You should see output indicating that `package_a` is being built and optimized.

Similarly, to build only `package_b`, use:

```sh
cargo build -p package_b --release
```

This setup shows how to organize a Cargo workspace and build specific packages within it using the `-p` flag.
