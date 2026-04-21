# ADR 0001: Use SQLite for Local Persistence

## Status
Accepted

## Context
Marinara Engine is designed to be a "plug-and-play" local-first application. Users should be able to download the app, run it, and immediately start playing without needing to sign up for an account, spin up a PostgreSQL docker container, or provision cloud resources. However, the app still needs a robust relational database to handle Chats, Characters, Messages, Lorebooks, and Settings.

## Decision
We will use **SQLite** as the default and primary database engine via the `better-sqlite3` driver and `Drizzle ORM`. 

## Consequences
**Positive:**
- Zero setup for the end user. The database file (`marinara-engine.db`) is automatically created on disk upon first startup.
- Easy to backup, share, and migrate data (it's just a file).
- Drizzle ORM provides excellent type-safety across our TypeScript monolithic structure.

**Negative:**
- SQLite does not scale well horizontally, but since this is a local app designed for single-user scale, horizontal scaling is not a requirement.
- Full text search across large chat histories is slightly harder to implement than in Postgres, but still feasible via SQLite FTS5 extensions if needed down the road.
