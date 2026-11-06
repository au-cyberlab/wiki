# Overview

This is a very simple pin board looking form that lets users make posts and lets others reply to that post. There is no auth and you just need to add in a display username.

## Configuration

There is almost no configuration here. It is intended to be simple and only serve the purpose of sharing posts without authentication.

To deploy this system, include `forum` in a lab configuration or the `deployment_config.yml`, then deploy the branch.

## Usage

The forum server is exposed on port 8080.

You can either connect to it directly, or by using the command line like the examples below.

```bash
# See all forum posts
curl http://forum_server.forum.lab:8080/api/posts

# See replies to a specific thread
curl http://forum_server.forum.lab:8080/api/posts/1

# Make a new post
curl -X POST http://forum_server.forum.lab:8080/api/posts \
     -H 'Content-Type: application/json' \
     -d '{"username": "alice", "content": "Grid exercise moved to 2pm"}'

# Reply to a post
curl -X POST http://forum_server.forum.lab:8080/api/posts/1/replies \
     -H 'Content-Type: application/json' \
     -d '{"username": "bob", "content": "Thanks!"}'
```
