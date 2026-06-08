---
description: "Nostr Example with Dual Database Architecture"
og_image:    "https://iambrainstorming.github.io/coding_blog/images/nostr-dual-database.png"
og_type:     "article"
twitter_card: "Nostr Example with Dual Database Architecture"
---

# Recipe Upload Example


## Relay Server

cargo.toml
```toml
[package]
name = "relay-server"
version = "0.1.0"
edition = "2024"

[dependencies]
nostr-sdk = { git = "https://github.com/rust-nostr/nostr", rev = "02a5926" }
nostr-relay-builder = { git = "https://github.com/rust-nostr/nostr", rev = "02a5926" }
nostr-lmdb = { git = "https://github.com/rust-nostr/nostr", rev = "02a5926" }
libsql-rusqlite = "0.33.0" # Drop-in rusqlite replacement with libsql features
tokio = { version = "1", features = ["full"] }
tracing-subscriber = "0.3"
serde_json = "1"
serde = { version = "1", features = ["derive"] }
serde_yaml = "0.9"
```


main.rs
```rust
use nostr_lmdb::NostrLmdb;
use nostr_relay_builder::prelude::*;
use rusqlite::{Connection, params};
use serde::Deserialize;
use std::net::SocketAddr;
use std::sync::{Arc, Mutex};

// Define a custom kind for Recipes (Parameterized Replaceable)
const RECIPE_KIND: u16 = 31000;

#[derive(Debug, Clone)]
struct DbStorage {
    conn: Arc<Mutex<Connection>>,
}

// Minimal structs to parse the YAML and extract the recipe name
#[derive(Deserialize, Debug)]
struct RecipeYaml {
    #[serde(rename = "recipeUpload")]
    recipe_upload: Option<RecipeUpload>,
}

#[derive(Deserialize, Debug)]
struct RecipeUpload {
    recipe: Option<Recipe>,
}

#[derive(Deserialize, Debug)]
struct Recipe {
    name: Option<String>,
}

// ==========================================
// 1. WRITE POLICY (Intercepts incoming posts)
// ==========================================
impl WritePolicy for DbStorage {
    fn admit_event<'a>(
        &'a self,
        event: &'a Event,
        _addr: &'a SocketAddr,
    ) -> BoxedFuture<'a, WritePolicyResult> {
        let conn = self.conn.clone();
        let event = event.clone();

        Box::pin(async move {
            if event.kind == Kind::from(RECIPE_KIND) {
                match serde_yaml::from_str::<RecipeYaml>(&event.content) {
                    Ok(parsed_yaml) => {
                        let recipe_name = parsed_yaml
                            .recipe_upload
                            .and_then(|r| r.recipe)
                            .and_then(|r| r.name)
                            .unwrap_or_else(|| "Unnamed Recipe".to_string());

                        // FIX: Use as_slice() to safely check for the "d" tag
                        // without relying on internal TagKind enums that change between versions.
                        let d_tag = event
                            .tags
                            .iter()
                            .find_map(|t| {
                                let tag_slice = t.as_slice();
                                // Check if the first element of the tag array is "d"
                                if tag_slice.first().map(|s| s.as_str()) == Some("d") {
                                    // If it is, return the second element (the identifier)
                                    tag_slice.get(1).cloned()
                                } else {
                                    None
                                }
                            })
                            .unwrap_or_default();

                        let db_conn = conn.lock().unwrap();
                        match db_conn.execute(
                            "INSERT OR REPLACE INTO recipes (event_id, pubkey, d_tag, recipe_name, yaml_content, created_at)
                             VALUES (?1, ?2, ?3, ?4, ?5, ?6)",
                            params![
                                event.id.to_hex(),
                                event.pubkey.to_hex(),
                                d_tag,
                                recipe_name,
                                event.content,
                                event.created_at.as_secs() as i64,
                            ],
                        ) {
                            Ok(_) => println!("💾 Saved recipe '{}' to database", recipe_name),
                            Err(e) => eprintln!("❌ Database error saving recipe: {}", e),
                        }
                    }
                    Err(e) => {
                        eprintln!(
                            "❌ Invalid YAML for recipe event {}: {}",
                            event.id.to_hex(),
                            e
                        );
                        return WritePolicyResult::reject(
                            MachineReadablePrefix::Error,
                            "Invalid YAML format for recipe",
                        );
                    }
                }
            }
            WritePolicyResult::Accept
        })
    }
}

// ==========================================
// 2. QUERY POLICY (Intercepts incoming read requests)
// ==========================================
#[derive(Debug)]
struct RecipeQueryPolicy;

impl QueryPolicy for RecipeQueryPolicy {
    fn admit_query<'a>(
        &'a self,
        query: &'a mut Filter,
        _addr: &'a SocketAddr,
    ) -> BoxedFuture<'a, QueryPolicyResult> {
        Box::pin(async move {
            // Rule A: Require the client to specify 'kinds' (prevents "give me everything" queries)
            let kinds = match &query.kinds {
                Some(k) => k,
                None => {
                    return QueryPolicyResult::reject(
                        MachineReadablePrefix::Error,
                        "Filter must specify 'kinds'",
                    );
                }
            };

            // Rule B: Only allow querying Recipes (31000) and Metadata (0)
            let allowed_kinds = kinds
                .iter()
                .all(|k| *k == Kind::from(RECIPE_KIND) || *k == Kind::Metadata);

            if !allowed_kinds {
                return QueryPolicyResult::reject(
                    MachineReadablePrefix::Error,
                    "This relay only serves Recipe (31000) and Metadata (0) events",
                );
            }

            // Rule C: Limit the number of authors to prevent expensive DB scans
            if query.authors.as_ref().map(|a| a.len()).unwrap_or(0) > 10 {
                return QueryPolicyResult::reject(
                    MachineReadablePrefix::Error,
                    "Too many authors in query (max 10)",
                );
            }

            // If all rules pass, allow the database query
            QueryPolicyResult::Accept
        })
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    tracing_subscriber::fmt::init();
    println!("🚀 Starting Nostr Relay with Recipe YAML storage...");
    let db = NostrLmdb::open("./db/nostr-lmdb").await?;

    let conn = Connection::open("nostr_relay.db")?;
    conn.execute_batch(
        "CREATE TABLE IF NOT EXISTS recipes (
            event_id TEXT PRIMARY KEY,
            pubkey TEXT NOT NULL,
            d_tag TEXT NOT NULL,
            recipe_name TEXT,
            yaml_content TEXT NOT NULL,
            created_at INTEGER NOT NULL
        );",
    )?;
    println!("✅ Database initialized at ./nostr_relay.db");

    // Build the relay with BOTH policies attached
    let relay = LocalRelay::builder()
        .port(7777)
        .database(db)
        .write_policy(DbStorage {
            conn: Arc::new(Mutex::new(conn)),
        })
        .query_policy(RecipeQueryPolicy) // <--- QUERY POLICY ATTACHED
        .build();

    println!("🔌 Relay is starting...");
    relay.run().await?;

    let url = relay.url().await;
    println!("✅ Relay is live at: {}", url);
    println!("Press Ctrl+C to stop...");

    tokio::signal::ctrl_c().await?;
    println!("\n🛑 Shutting down...");
    std::process::exit(0);
}
```

## Nostr client

Cargo.toml

```toml
[package]
name = "nostr-client"
version = "0.1.0"
edition = "2024"

[dependencies]
# Use the latest master branch to match your server
nostr-sdk = { git = "https://github.com/rust-nostr/nostr", rev = "02a5926" }
tokio = { version = "1", features = ["full"] }
futures = "0.3" # Required to use .next() on the notifications stream
```

main.rs

```rust
use nostr_sdk::prelude::*;
use std::env;
use std::time::Duration; // Import Duration for the timeout

const RECIPE_KIND: u16 = 31000;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args: Vec<String> = env::args().collect();
    let relay_url = if args.len() > 1 {
        args[1].clone()
    } else {
        "ws://127.0.0.1:7777".to_string()
    };

    println!("🔑 Generating new Nostr identity...");
    let keys = Keys::generate();
    println!("Public Key: {}", keys.public_key().to_hex());

    let authenticator = SignerAuthenticator::new(keys.clone());
    let client = Client::builder().authenticator(authenticator).build();

    println!("🔌 Connecting to relay: {}", relay_url);
    client.add_relay(&relay_url).await?;
    client.connect().await;

    // Wait for connection to establish
    tokio::time::sleep(Duration::from_secs(1)).await;

    // 1. Define the Recipe in YAML format
    let yaml_content = r#"recipeUpload:
  recipe:
    name: Veg Fried Rice
    description: A flavorful Indo-China-style vegetable fried rice.
    servings: 3
    totalTime: 1600
  ingredients:
    - name: Rice
      quantity: 1 cup uncooked
    - name: Oil
      quantity: 2 tbsp
  instructions:
      title: Prepare Rice
      description: Cook and cool rice for frying.
      details:
        - text: Wash rice thoroughly and soak for 10-15 minutes.
"#;

    println!("📝 Creating a Recipe Event (Kind {})...", RECIPE_KIND);

    // 2. Build and sign the event
    let event = EventBuilder::new(Kind::from(RECIPE_KIND), yaml_content)
        .tag(Tag::identifier("veg-fried-rice"))
        .finalize(&keys)?;

    println!("📡 Publishing recipe to relay...");
    let output = client.send_event(&event).await?;
    println!(
        "✅ Recipe published successfully! Event ID: {}",
        output.id().to_hex()
    );

    // Wait for relay to process and store the event
    tokio::time::sleep(Duration::from_secs(2)).await;

    // 3. Query the recipe back from the relay!
    println!("🔍 Querying the recipe back from the relay...");

    let filter = Filter::new()
        .kind(Kind::from(RECIPE_KIND))
        .identifier("veg-fried-rice");

    // Use fetch_events to automatically handle the REQ, wait for EOSE, and collect events.
    // This is much more reliable than manually parsing the notification stream.
    let events = client.fetch_events(filter).await?;

    if events.is_empty() {
        println!("❌ No recipes found from the relay.");
    } else {
        for event in events.into_iter() {
            if event.kind == Kind::from(RECIPE_KIND) {
                println!("\n🎉 RECEIVED RECIPE FROM RELAY:");
                println!("-----------------------------------");
                println!("{}", event.content);
                println!("-----------------------------------");
            }
        }
    }

    println!("👋 Disconnecting...");
    client.disconnect().await;

    Ok(())
}
```

## Dual Database Creates Duplicate Storage

**Yes, using the dual database architecture creates duplicate storage.**

Here is exactly what is being duplicated:

| Data Field | Stored in Internal Nostr DB? | Stored in Custom `recipes` Table? | Duplicate? |
| :--- | :--- | :--- | :--- |
| `event_id` | ✅ Yes | ✅ Yes | **Yes** |
| `pubkey` | ✅ Yes | ✅ Yes | **Yes** |
| `kind` | ✅ Yes | ❌ No (Hardcoded as 31000) | No |
| `created_at` | ✅ Yes | ✅ Yes | **Yes** |
| `content` (YAML) | ✅ Yes | ✅ Yes | **Yes** |
| `tags` | ✅ Yes | ❌ No (Only `d` tag extracted) | No |

### Why do developers accept this duplication?

It seems wasteful, but in Nostr application development, this "Dual Database" pattern is often used intentionally to solve a specific problem: **Query Performance vs. Protocol Compliance.**

1.  **The Internal Nostr DB (Raw Storage):**
    *   **Purpose:** It handles the complex Nostr logic automatically (NIP-33 replaceable events, NIP-09 deletions, signature verification).
    *   **Query Limitation:** It is very hard to search inside this database. If you want to find "all recipes that use 'Rice'", the database has to load every single YAML string into memory and parse it one by one. This is slow.

2.  **The Custom `recipes` Table (Indexed Storage):**
    *   **Purpose:** It allows for **fast SQL indexing**.
    *   **Query Advantage:** You can run `SELECT * FROM recipes WHERE recipe_name LIKE '%Fried Rice%'` instantly. You can do complex SQL filtering (like searching by ingredient or cooking time). You can't do that efficiently with the raw Nostr database.

### SQLite File Collision
If you are trying to open the exact same file (`nostr_relay.db`) using two completely different database engines:
1. `NostrSqlite::open("nostr_relay.db")` (The internal Nostr DB engine)
2. `Connection::open("nostr_relay.db")` (Your custom `rusqlite` engine)

SQLite does not handle concurrent writes from two separate connection pools to the same file. If a client publishes a recipe at the exact same time the `LocalRelay` tries to save a raw event, your app will crash with a `SQLITE_BUSY` / `database is locked` panic.
