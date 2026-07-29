# Base62 URL Shortener

A containerized URL shortening service built with React, Express, MongoDB, Redis, ZooKeeper, and Nginx.

## Overview

This project provides a simple URL shortener that generates Base62 short codes and stores original URLs in MongoDB. It uses Redis for caching, ZooKeeper for distributed token range management, and Nginx to proxy requests to the Node.js API.

## Architecture

- `client/` - React application for shortening URLs and showing the shortened link.
- `server/` - Express API handling URL creation, redirection, and token range generation.
- `redis-server` - Redis cache used to store recently shortened URLs.
- `zookeeper-server` - ZooKeeper used to manage token ranges for distributed hashing.
- `nginx/` - Nginx reverse proxy forwarding requests to the Node API.

## Features

- Shorten URLs into Base62 hash strings
- Redirect shortened URLs to the original destination
- Redis cache for repeated URL lookups
- MongoDB persistence for URL records and visit tracking
- ZooKeeper-backed token range generation for distributed environments
- Docker Compose setup for local development

## Services and Ports

- `nginx` - `http://localhost:4000`
- `client` - `http://localhost:3000`
- `node-server` - `http://localhost:8081`
- `redis-server` - `localhost:6379`
- `zookeeper-server` - `localhost:2181`

> In this setup, the client application sends requests to `http://localhost:4000`, which is proxied by Nginx to the Node server.

## Installation

1. Clone the repository.
2. Ensure Docker Desktop is installed and running.
3. From the project root, start the services:

```bash
docker compose up --build
```

4. Open the application in the browser:

```bash
http://localhost:4000
```

## Development Notes

- The React app is served from `client/` and uses `axios` to call the API.
- The Express server is in `server/` and loads environment variables with `dotenv`.
- The server sets up MongoDB, Redis, and ZooKeeper connections on startup.
- A background process (`purgeAliases`) is executed when the server starts.

## Environment Variables

The server expects the following environment variables in a `.env` file or environment:

- `REACT_APP_MONGODB_URI` - MongoDB connection string
- `REACT_APP_REDIS_HOST` - Redis hostname (for example `redis-server`)
- `REACT_APP_REDIS_PORT` - Redis port (for example `6379`)
- `PORT` - Optional port for the server (default `8081`)

Example `.env`:

```env
REACT_APP_MONGODB_URI=mongodb://localhost:27017/shortener
REACT_APP_REDIS_HOST=redis-server
REACT_APP_REDIS_PORT=6379
PORT=8081
```

## API Endpoints

- `POST /url`
  - Request body: `{ "OriginalUrl": "https://example.com" }`
  - Response: shortened hash string
- `GET /url/:identifier`
  - Redirects to the original URL associated with the hash
- `GET /del`
  - Removes the ZooKeeper token node (used for cleanup/testing)

## How It Works

1. The client validates the URL and sends it to the API.
2. The server checks Redis for an existing cached hash.
3. If missing, it looks up MongoDB for the original URL.
4. If still missing, the server generates a new Base62 hash using the current ZooKeeper token range and stores the new URL record.
5. The redirect endpoint loads the original URL by hash and increments visit count via a queue.

## Notes

- The current hash generator uses the range managed by ZooKeeper to produce short codes.
- The app uses a local proxy configuration through Nginx for incoming web traffic.
- To scale the API service, you can run:

```bash
docker compose up --build --scale node-server=3
```

## License

This project is provided as-is. See the repository `LICENSE` for license details.
