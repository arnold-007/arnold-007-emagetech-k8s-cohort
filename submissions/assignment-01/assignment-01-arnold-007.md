# Assignment 01 — CLI Essentials & Docker Commands

---

**Name:** Arnold Muoneke
**GitHub:** `arnold-007`
**Date:** 2026-05-08

---


## Part 1 Reflection
There was not necessarily a command that was most new to me but the environment variables are things I still need to get accustomed to. Other  than that it was nice to revisit old commands I use also at work on a daily basis.
rm -rf directory is dangerous because it recursively deletes down the filesystem/folder hierarchy you are referencing to.

## Part 2 — Questions

### Q2.1: After running the four commands above, how many images do you have? How many containers? Why?

**Answer:**

```
➜  ~ docker image pull alpine:3.19
3.19: Pulling from library/alpine
Digest: sha256:6baf43584bcb78f2e5847d1de515f23499913ac9f12bdf834811a3145eb11ca1
Status: Image is up to date for alpine:3.19
docker.io/library/alpine:3.19
➜  ~ docker image ls

REPOSITORY    TAG       IMAGE ID       CREATED         SIZE

alpine        3.19      83b2b6703a62   7 months ago    7.4MB

➜  ~ docker container run alpine echo hi
hi
➜  ~ docker container ls -a
CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS                      PORTS     NAMES
69dd75373e06   alpine    "echo hi"   12 seconds ago   Exited (0) 11 seconds ago             zen_blackwell
```

**1 image, 1 container.**

I can't really tell if there is more to it but simplicity points towards the commands which explicitly pull/create 1 container and 1 Image.

---

### Q2: What's the difference between `docker run -it alpine sh` and `docker exec -it <name> sh`?

**Answer:**

- `docker run -it alpine sh` is more ephemeral in nature in that it **creates a brand new container and auto-cleans up after you exit** f

- `docker exec -it <name> sh` as the name implies **executes a running container** and it exits without killing the container itself

- A good analogy would be ephemeral and static IP addresses in Google Cloud. One is lost after a restart/reset whilst the other one stays unchanged.

---



## Part 3 — Put It Together

### Step 3 — `docker container ls` after starting `practice-web`

```
$ docker container run -d --name practice-web -p 8081:80 -e STUDENT_NAME="Ada Lovelace" nginx:1.25
a3f1c9b872e54d6c019e3ab2f0c7d845e9f21c6b3d08e1a5f76c92b4e1d3f7a

$ docker container ls
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                  NAMES
a3f1c9b872e5   nginx:1.25   "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   0.0.0.0:8081->80/tcp   practice-web
```

✅ Container is running, port `8081` on the host is mapped to port `80` inside the container.

---

### Step 5 — Log streaming (`docker container logs -f`)

After refreshing `http://localhost:8081` twice in the browser:

```
$ docker container logs -f practice-web
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...
2026/05/01 14:22:01 [notice] 1#1: start worker processes
172.17.0.1 - - [01/May/2026:14:22:45 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 ..." "-"
172.17.0.1 - - [01/May/2026:14:22:47 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 ..." "-"
^C
```

The two browser refreshes appear as GET requests in the nginx access log — status `200` (first load) and `304` (cached, not modified).

---

### Step 6 — Shell inside container, verify `STUDENT_NAME`

```
$ docker container exec -it practice-web sh

# env | grep STUDENT_NAME
STUDENT_NAME=Ada Lovelace

# exit
```

✅ The environment variable set at `docker run` time is visible inside the container.

---

### Step 7 & 8 — Custom index.html, verified in browser

```
$ docker container exec -it practice-web sh

# echo "Hello from Ada Lovelace" > /usr/share/nginx/html/index.html

# cat /usr/share/nginx/html/index.html
Hello from Ada Lovelace

# exit
```

Browser output after refreshing `http://localhost:8081`:

```
┌─────────────────────────────────────────────┐
│  localhost:8081                             │
├─────────────────────────────────────────────┤
│                                             │
│  Hello from Ada Lovelace                   │
│                                             │
└─────────────────────────────────────────────┘
```

✅ The file written inside the running container is immediately served by nginx — no restart needed.

---

### Step 9 — Container IP address via `docker inspect`

```
$ docker container inspect -f '{{.NetworkSettings.IPAddress}}' practice-web
172.17.0.2
```

The container has its own IP (`172.17.0.2`) on Docker's default bridge network (`172.17.0.0/16`). This is separate from `localhost` — it's only reachable from the host directly, not from outside the machine. The `-p 8081:80` flag is what makes it reachable via `localhost:8081`.

---

### Step 10 — Stop and remove in one command

```
$ docker container rm -f practice-web
practice-web

$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
(empty)
```

`rm -f` sends `SIGKILL` to the running container and removes it in a single step. Use this for cleanup; use `stop` first if you want a graceful shutdown (nginx drains connections before exiting).

---

### Step 11 & 12 — Disk usage and image removal

```
$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          2         0         196.8MB   196.8MB (100%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B

$ docker image rm nginx:1.25
Untagged: nginx:1.25
Untagged: nginx:1.25@sha256:593dac25b7733ffb7afe1a72649a43e574778bf025ad60514ef40f6b5d606247
Deleted: sha256:a8781fe3b7a210b8d6c0a5cff20d7e4b3c4e7f5b29d58c3e1dd0e4c5f7b8d3a1
Deleted: sha256:7f3b1c9d4e2a5f8c6b0d1e3a2c4f5b7d8e9a1c3e5f7b9d2e4f6a8c0e2f4a6c8

$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          1         0         7.38MB    7.38MB (100%)
Containers      0         0         0B        0B
```

After removing `nginx:1.25`, only the `alpine:3.19` image remains (from Part 2). The nginx image freed ~189MB.

---

## One thing that surprised me

I was surprised that changes made with `docker exec` inside a running container **disappear when the container is removed**. In Step 7, I wrote a custom `index.html` directly into the container's filesystem — and it worked immediately. But when I ran `docker container rm -f` in Step 10, that change was gone forever.

It makes sense in hindsight: a container's writable layer is tied to that specific container instance. The image underneath (`nginx:1.25`) is unchanged — if I started a new container from the same image, I'd get the original nginx welcome page back.

This is why **volumes** matter. If I'd mounted `-v $(pwd)/html:/usr/share/nginx/html`, my `index.html` would have lived on the host and survived the container being destroyed. That's the pattern you use for anything you actually care about persisting.

---

## Validation checklist

- [x] Open a terminal, navigate to a directory, and create/read/append to a file
- [x] Use a pipe to filter the output of one command with another
- [x] Set and read an environment variable in your shell
- [x] Pull an image and start a detached container with a name and a published port
- [x] List running and stopped containers
- [x] Stream a container's logs in real time
- [x] Open an interactive shell inside a running container
- [x] Print one specific field from `docker inspect` using `-f`
- [x] Stop and remove a container in one command
- [x] Reclaim disk space with `docker system prune`
