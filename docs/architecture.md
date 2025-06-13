# A2A Architecture Overview

A2A (Agent2Agent) is an open protocol that lets independent agents collaborate over standard HTTP(S) and JSON-RPC 2.0. It provides a common language for discovering agents, exchanging messages, and coordinating long running tasks.

![A2A architecture diagram](assets/a2a-main.png){width="60%" style="margin:20px auto;display:block;"}

## Protocol Goals

- **Interoperability:** allow agents built with different frameworks to work together.
- **Task management:** provide a shared model for initiating tasks, tracking state, and exchanging artifacts.
- **Flexible transports:** support simple request/response, real‑time streaming, and asynchronous push notifications.
- **Security:** rely on existing web standards for HTTPS and authentication while keeping agents opaque.

## Core Concepts

- **A2A Client** and **A2A Server** form the basic communication pair. The client sends JSON‑RPC requests to the server's HTTP endpoint.
- **Agent Card:** a metadata document describing an agent’s endpoint, capabilities, supported authentication schemes, and available skills.
- **Task:** the stateful unit of work tracked by the server. A task progresses through lifecycle states such as `submitted`, `working`, `input-required`, and `completed`.
- **Artifact:** files or structured data that an agent produces as part of a task.

## Typical Task Flow

1. The client retrieves the server’s Agent Card from a well‑known URL and learns its capabilities and required authentication.
2. The client authenticates and sends a `message/send` or `message/stream` request to start a task.
3. The server returns a `Task` object containing a unique `taskId` and current status.
4. As work progresses the client receives updates either through Server‑Sent Events, push notifications, or by polling `tasks/get`.
5. When the task reaches a terminal state (`completed`, `failed`, or `canceled`), the final artifacts and status are returned.

## Streaming and Asynchronous Interaction

- **Server-Sent Events (SSE):** When the server has `capabilities.streaming: true`, clients can call `message/stream` to keep an HTTP connection open and receive incremental `TaskStatusUpdateEvent` and `TaskArtifactUpdateEvent` objects. The specification describes this in the [Streaming Transport](./specification.md#33-streaming-transport-server-sent-events) section.
- **Push notifications:** Servers that support `capabilities.pushNotifications` can send updates to a client’s webhook using the `tasks/pushNotificationConfig/*` methods.
- **Resubscribe:** If a connection drops, the client may resume streaming updates using `tasks/resubscribe`.

## Authentication

A2A uses standard HTTP authentication headers, discovered via the server’s Agent Card. Clients obtain credentials out‑of‑band and include them with every request (e.g., `Authorization: Bearer <token>`). Servers must authenticate each request and MAY challenge with `401 Unauthorized` or `403 Forbidden` as noted in the [Authentication and Authorization](./specification.md#4-authentication-and-authorization) section.
