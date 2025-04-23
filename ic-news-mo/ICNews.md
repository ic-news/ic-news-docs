# ICNews API Documentation

## Overview

This document provides an overview of the API endpoints and functionalities of the `IcNews` canister, which manages news, categories, tags, providers, and archives on the Internet Computer.

## Types

- **News**: Represents a news item with properties like `index`, `provider`, `category`, `tags`, `hash`, `created_at`, etc.
- **Category**: Represents a news category with a `name`.
- **Tag**: Represents a tag with a `name`.
- **ArchiveData**: Contains information about archived news including the canister reference, number of stored news, and index range.
- **NewsRequest**: Structure for querying news with `start` and `length` parameters.
- **NewsResponse**: Response structure for news queries including total length, first index, current news, and archived news information.
- **Page<T>**: Generic pagination structure with `content`, `limit`, `offset`, and `totalElements`.
- **WebSocketValue**: Variant type to handle different types of WebSocket messages.

## API Endpoints

### Task Status

- **get_task_status**: Query the current status of background tasks and retrieve the last error message if any.
  - Returns: `Result<(Bool, Text), Error>` - A tuple of task status (true if running) and last error message.

### Archives

- **get_archives**: Retrieve a list of all archive data.
  - Returns: `Result<[ArchiveData], Error>` - List of archive data.

### Providers

- **get_providers**: Retrieve a list of all registered providers.
  - Returns: `Result<[(Principal, Text)], Error>` - List of provider principal and alias pairs.
- **add_provider**: Add a new provider (controller only).
  - Parameters: `pid: Principal`, `alias: Text`
  - Returns: `Result<Bool, Error>` - Success status.

### Categories

- **get_categories**: Retrieve a list of all categories.
  - Returns: `Result<[Category], Error>` - List of categories.
- **add_categories**: Add new categories (provider only).
  - Parameters: `categories: AddCategoryArgs`
  - Returns: `Result<Bool, Error>` - Success status.

### Tags

- **get_tags**: Retrieve a list of all tags.
  - Returns: `Result<[Tag], Error>` - List of tags.
- **add_tags**: Add new tags (provider only).
  - Parameters: `tags: AddTagArgs`
  - Returns: `Result<Bool, Error>` - Success status.

### News

- **add_news**: Add new news items (provider only).
  - Parameters: `news: AddNewsArgs`
  - Returns: `Result<Bool, Error>` - Success status.
- **query_news**: Composite query to retrieve news based on start index and length.
  - Parameters: `req: NewsRequest`
  - Returns: `NewsResponse` - Includes news in canister and references to archived news.
- **total_news**: Query the total number of news items including archived ones.
  - Returns: `Result<Nat, Error>` - Total count of news.
- **query_latest_news**: Retrieve the latest news items.
  - Parameters: `size: Nat`
  - Returns: `Result<[News], Error>` - Array of latest news.
- **query_news_by_category**: Retrieve paginated news by category.
  - Parameters: `category: Text`, `offset: Nat`, `limit: Nat`
  - Returns: `Result<Page<News>, Error>` - Paginated news results.
- **query_news_by_tag**: Retrieve paginated news by tag.
  - Parameters: `tag: Text`, `offset: Nat`, `limit: Nat`
  - Returns: `Result<Page<News>, Error>` - Paginated news results.
- **get_news_by_hash**: Retrieve a specific news item by its hash.
  - Parameters: `hash: Text`
  - Returns: `Result<News, Error>` - Specific news item.
- **get_news_by_index**: Composite query to retrieve a specific news item by index, including from archives.
  - Parameters: `index: Nat`
  - Returns: `Result<News, Error>` - Specific news item.
- **get_news_by_time**: Retrieve news items created after a specific time.
  - Parameters: `begin_time: Nat`, `size: Nat`
  - Returns: `Result<[News], Error>` - Array of news items.

### WebSocket

- **ws_open**: Initiates a WebSocket connection.
  - Parameters: `args: CanisterWsOpenArguments`
  - Returns: `CanisterWsOpenResult`
- **ws_close**: Closes a WebSocket connection.
  - Parameters: `args: CanisterWsCloseArguments`
  - Returns: `CanisterWsCloseResult`
- **ws_message**: Handles incoming WebSocket messages.
  - Parameters: `args: CanisterWsMessageArguments`, `msg_type: ?AppMessage`
  - Returns: `CanisterWsMessageResult`
- **ws_get_messages**: Retrieves messages for WebSocket clients.
  - Parameters: `args: CanisterWsGetMessagesArguments`
  - Returns: `CanisterWsGetMessagesResult`

## Internal Functions

- **\_save_to_archive**: Periodically saves news to archive canisters if the buffer exceeds a threshold.
- **\_deploy_archive_canister**: Deploys a new archive canister if needed.

## System Functions

- **preupgrade**: Saves the news buffer to a stable array before upgrade.
- **postupgrade**: Restores the news buffer from the stable array after upgrade.

## WebSocket Callbacks

- **on_open**: Handles new WebSocket connections.
- **on_message**: Processes incoming WebSocket messages.
- **on_close**: Manages WebSocket connection closures.

## Request Examples with TypeScript

Below are some examples of how to interact with the `IcNews` canister using TypeScript. These examples assume you are using the `@dfinity/agent` library to interact with the Internet Computer.

### Setting up the Agent

```typescript
import { Actor, HttpAgent } from "@dfinity/agent";
import { idlFactory } from "./idl/icnews"; // Replace with the actual path to your IDL file

// Initialize the agent
const agent = new HttpAgent({ host: "https://ic0.app" });

// Create an actor for the IcNews canister
const icNewsActor = Actor.createActor(idlFactory, {
  agent,
  canisterId: "fs66q-viaaa-aaaan-qzwja-cai",
});
```

### Querying Latest News

```typescript
async function getLatestNews(size: number) {
  try {
    const result = await icNewsActor.query_latest_news(size);
    if ("ok" in result) {
      console.log("Latest News:", result.ok);
      return result.ok;
    } else {
      console.error("Error fetching latest news:", result.err);
      throw new Error(result.err);
    }
  } catch (error) {
    console.error("Unexpected error:", error);
    throw error;
  }
}

// Usage
getLatestNews(5)
  .then((news) => {
    // Process the news items
  })
  .catch((err) => {
    // Handle error
  });
```

### Querying News by Category with Pagination

```typescript
async function getNewsByCategory(category: string, offset: number, limit: number) {
  try {
    const result = await icNewsActor.query_news_by_category(category, offset, limit);
    if ("ok" in result) {
      console.log(`News for category ${category}:`, result.ok);
      return result.ok;
    } else {
      console.error("Error fetching news by category:", result.err);
      throw new Error(result.err);
    }
  } catch (error) {
    console.error("Unexpected error:", error);
    throw error;
  }
}

// Usage
getNewsByCategory("Technology", 0, 10)
  .then((page) => {
    // Process the paginated news items
    console.log("Total items:", page.totalElements);
    console.log("News content:", page.content);
  })
  .catch((err) => {
    // Handle error
  });
```

### Establishing WebSocket Connection

```typescript
// Note: WebSocket interactions require additional setup and libraries.
import IcWebSocket, { createWsConfig, generateRandomIdentity } from "ic-websocket-js";

// Configuration for WebSocket
const wsConfig = createWsConfig({
  canisterId: "fs66q-viaaa-aaaan-qzwja-cai",
  canisterActor: icNewsActor,
  networkUrl: "https://icp-api.io",
});

// Create WebSocket instance
const ws = new IcWebSocket("wss://api.ic.news/ws/", undefined, wsConfig);

// Setup event handlers
ws.onopen = () => {
  console.log("WebSocket connection established");
  // Subscribe to latest news updates
  const subscribeMessage = {
    topic: "get_latest_news",
    args: [],
    result: { Common: { Text: "" } },
    timestamp: BigInt(Date.now()),
  };
  ws.send(subscribeMessage);
};

ws.onmessage = (event) => {
  const message = event.data;
  console.log("Received message:", message);
  // Process received news data if needed
};

ws.onerror = (event) => {
  console.error("WebSocket error:", event.error);
};

ws.onclose = () => {
  console.log("WebSocket connection closed");
};
```

These examples cover basic interactions with the `IcNews` canister. Depending on your application's needs, you may need to adjust the error handling, add authentication, or incorporate additional logic for WebSocket communication.

This documentation covers all public and significant internal functionalities of the `IcNews` canister as of the latest code review.
