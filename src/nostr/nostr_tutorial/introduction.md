# Nostr Tutorial

Date: 07-06-2026

```bash
mkdir nostr-db-demo
cargo new relay-server
cargo new nostr-client
```

## Relay Server

main.rs

```rust
use nostr_relay_builder::prelude::*;
use rusqlite::{Connection, params};
use std::net::SocketAddr;
use std::sync::{Arc, Mutex};

// 1. Define a custom state holder for our database connection
#[derive(Debug, Clone)]
struct DbStorage {
    conn: Arc<Mutex<Connection>>,
}

// 2. Implement WritePolicy to intercept and save events to libsql-rusqlite
impl WritePolicy for DbStorage {
    fn admit_event<'a>(
        &'a self,
        event: &'a Event,
        _addr: &'a SocketAddr,
    ) -> BoxedFuture<'a, WritePolicyResult> {
        let conn = self.conn.clone();
        let event = event.clone(); // Clone to own the event inside the async block

        Box::pin(async move {
            // Serialize tags to JSON. Using to_vec() to convert Tags to Vec<Tag>
            let tags_json = serde_json::to_string(&event.tags.to_vec()).unwrap_or_default();

            // Lock the database connection
            let db_conn = conn.lock().unwrap();

            // Insert the event into the database
            match db_conn.execute(
                "INSERT OR REPLACE INTO nostr_events (event_id, pubkey, kind, content, created_at, tags)
                 VALUES (?1, ?2, ?3, ?4, ?5, ?6)",
                params![
                    event.id.to_hex(),
                    event.pubkey.to_hex(),
                    u16::from(event.kind) as i64,
                    event.content,
                    event.created_at.as_secs() as i64, // Fixed: use as_secs() instead of deprecated as_u64()
                    tags_json
                ],
            ) {
                Ok(_) => {
                    println!("💾 Saved event {} to database", event.id.to_hex());
                    WritePolicyResult::Accept
                }
                Err(e) => {
                    eprintln!("❌ Database error saving event: {}", e);
                    // Fixed: Use WritePolicyResult::reject with prefix and message
                    WritePolicyResult::reject(MachineReadablePrefix::Error, "Database error")
                }
            }
        })
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    tracing_subscriber::fmt::init();
    println!("🚀 Starting Nostr Relay with libsql-rusqlite storage...");

    // 1. Initialize the database
    let conn = Connection::open("nostr_relay.db")?;
    conn.execute_batch(
        "CREATE TABLE IF NOT EXISTS nostr_events (
            event_id TEXT PRIMARY KEY,
            pubkey TEXT NOT NULL,
            kind INTEGER NOT NULL,
            content TEXT,
            created_at INTEGER NOT NULL,
            tags TEXT
        );",
    )?;
    println!("✅ Database initialized at ./nostr_relay.db");

    // 2. Build the relay
    let relay = LocalRelay::builder()
        .write_policy(DbStorage {
            conn: Arc::new(Mutex::new(conn)),
        })
        .build();

    println!("🔌 Relay is starting...");

    // 3. Start the relay in the background (this returns immediately!)
    relay.run().await?;

    // 4. Get and print the URL
    let url = relay.url().await;
    println!("✅ Relay is live at: {}", url);
    println!("Press Ctrl+C to stop...");

    // 5. Keep the program alive until the user presses Ctrl+C
    tokio::signal::ctrl_c().await?;

    println!("\n🛑 Shutting down...");

    // 6. Force exit immediately to prevent Tokio from hanging on background WebSocket tasks
    std::process::exit(0);
}

```

cargo.toml

```toml
[package]
name = "relay-server"
version = "0.1.0"
edition = "2024"

[dependencies]
nostr-relay-builder = { git = "https://github.com/rust-nostr/nostr" }
nostr-sdk = { git = "https://github.com/rust-nostr/nostr" }
libsql-rusqlite = "0.33.0" # Drop-in rusqlite replacement with libsql features
tokio = { version = "1", features = ["full"] }
tracing-subscriber = "0.3"
serde_json = "1"
```

## Nostr client

```rust
use nostr_sdk::prelude::*;
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Get the relay URL from command line arguments
    let args: Vec<String> = env::args().collect();
    let relay_url = if args.len() > 1 {
        args[1].clone()
    } else {
        "ws://127.0.0.1:7070".to_string()
    };

    println!("🔑 Generating new Nostr identity...");
    let keys = Keys::generate();
    println!("Public Key: {}", keys.public_key().to_hex());

    // 2. Initialize the Nostr client using the builder pattern
    // Client::new() now takes 0 arguments. We must attach a SignerAuthenticator.
    let authenticator = SignerAuthenticator::new(keys.clone());
    let client = Client::builder().authenticator(authenticator).build();

    // 3. Add and connect to the relay
    println!("🔌 Connecting to relay: {}", relay_url);
    client.add_relay(&relay_url).await?;

    // connect() returns a builder that implements IntoFuture, so we can .await it
    client.connect().await;

    // Give the WebSocket a moment to fully establish the connection
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;

    // 4. Create a Text Note (Kind 1)
    println!("📝 Creating a text note...");
    let content = format!(
        "Hello Nostr! This is a test post from my Rust client. Timestamp: {}",
        Timestamp::now().as_secs()
    );

    // 5. Build and sign the event with our keys
    // send_event_builder was removed. We must finalize the EventBuilder into an Event.
    let event = EventBuilder::text_note(content).finalize(&keys)?;

    // 6. Publish the event to the relay
    println!("📡 Publishing event to relay...");
    let output = client.send_event(&event).await?;

    println!("✅ Event published successfully!");
    println!("🆔 Event ID: {}", output.id().to_hex());

    // Wait a moment to ensure the relay processes the event
    tokio::time::sleep(std::time::Duration::from_secs(2)).await;

    // 7. Disconnect cleanly
    // disconnect() returns (), so we do NOT use the '?' operator here.
    println!("👋 Disconnecting...");
    client.disconnect().await;

    Ok(())
}
```

Cargo.toml

```toml
[package]
name = "nostr-client"
version = "0.1.0"
edition = "2024"

[dependencies]
# Use the latest master branch to match your server
nostr-sdk = { git = "https://github.com/rust-nostr/nostr" }
tokio = { version = "1", features = ["full"] }
```


cargo run ws://127.0.0.1:port

e.g.
cargo run ws://127.0.0.1:64991
