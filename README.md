<h1 align="center">
    tower-sessions
</h1>

<p align="center">
    🥠 Sessions as a `tower` and `axum` middleware.
</p>

<div align="center">
    <a href="https://crates.io/crates/tower-sessions">
        <img src="https://img.shields.io/crates/v/tower-sessions.svg" />
    </a>
    <a href="https://docs.rs/tower-sessions">
        <img src="https://docs.rs/tower-sessions/badge.svg" />
    </a>
    <a href="https://github.com/maxcountryman/tower-sessions/actions/workflows/rust.yml">
        <img src="https://github.com/maxcountryman/tower-sessions/actions/workflows/rust.yml/badge.svg" />
    </a>
    <a href="https://codecov.io/gh/maxcountryman/tower-sessions" > 
        <img src="https://codecov.io/gh/maxcountryman/tower-sessions/graph/badge.svg?token=74POF0TJDN"/> 
    </a>
</div>

## 🎨 Overview

This crate provides sessions, key-value pairs associated with a site
visitor, as a `tower` middleware.

It offers:

- **Pluggable Storage Backends:** Bring your own backend simply by
  implementing the `SessionStore` trait, fully decoupling sessions from their
  storage.
- **An `axum` Extractor for `Session`:** Applications built with `axum`
  can use `Session` as an extractor directly in their handlers. This makes
  using sessions as easy as including `Session` in your handler.
- **Common Backends Out-of-the-Box:** `RedisStore`, SQLx
  (`SqliteStore`, `PostgresStore`, `MySqlStore`), and `MongoDBStore` stores
  are available via their respective feature flags.
- **Layered Caching:** With `CachingSessionStore`, applications can leverage a
  cache such as `MokaStore` to reduce roundtrips to the store when loading
  sessions.
- **Simple Key-Value Interface:** Sessions offer a key-value interface that
  supports native Rust types. So long as these types are `Serialize` and can
  be converted to JSON, it's straightforward to insert, get, and remove any
  value.
- **Strongly-Typed Sessions:** Strong typing guarantees are easy to layer on
  top of this foundational key-value interface.

This crate's session implementation is inspired by the [Django sessions middleware](https://docs.djangoproject.com/en/4.2/topics/http/sessions) and it provides a transliteration of those semantics.

### User authentication

For managing user authentication and authorization, please see [`axum-login`](https://github.com/maxcountryman/axum-login).

## 📦 Install

To use the crate in your project, add the following to your `Cargo.toml` file:

```toml
[dependencies]
tower-sessions = "0.5.1"
```

## 🤸 Usage

### `axum` Example

```rust
use std::net::SocketAddr;

use axum::{
    error_handling::HandleErrorLayer, response::IntoResponse, routing::get, BoxError, Router,
};
use http::StatusCode;
use serde::{Deserialize, Serialize};
use time::Duration;
use tower::ServiceBuilder;
use tower_sessions::{Expiry, MemoryStore, Session, SessionManagerLayer};

const COUNTER_KEY: &str = "counter";

#[derive(Default, Deserialize, Serialize)]
struct Counter(usize);

#[tokio::main]
async fn main() {
    let session_store = MemoryStore::default();
    let session_service = ServiceBuilder::new()
        .layer(HandleErrorLayer::new(|_: BoxError| async {
            StatusCode::BAD_REQUEST
        }))
        .layer(
            SessionManagerLayer::new(session_store)
                .with_secure(false)
                .with_expiry(Expiry::OnInactivity(Duration::seconds(10))),
        );

    let app = Router::new()
        .route("/", get(handler))
        .layer(session_service);

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn handler(session: Session) -> impl IntoResponse {
    let counter: Counter = session
        .get(COUNTER_KEY)
        .expect("Could not deserialize.")
        .unwrap_or_default();

    session
        .insert(COUNTER_KEY, counter.0 + 1)
        .expect("Could not serialize.");

    format!("Current count: {}", counter.0)
}
```

You can find this [example][counter-example] as well as other example projects in the [example directory][examples].

> [!NOTE]
> See the [crate documentation][docs] for more usage information.

## 🦺 Safety

This crate uses `#![forbid(unsafe_code)]` to ensure everything is implemented in 100% safe Rust.

## 🛟 Getting Help

We've put together a number of [examples][examples] to help get you started. You're also welcome to [open a discussion](https://github.com/maxcountryman/tower-sessions/discussions/new?category=q-a) and ask additional questions you might have.

## 👯 Contributing

We appreciate all kinds of contributions, thank you!

[counter-example]: https://github.com/maxcountryman/tower-sessions/tree/main/examples/counter.rs
[examples]: https://github.com/maxcountryman/tower-sessions/tree/main/examples
[docs]: https://docs.rs/tower-sessions
