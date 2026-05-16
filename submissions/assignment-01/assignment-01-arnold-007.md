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

- A closer analogy would be ephemeral and static IP addresses in Google Cloud. One is lost after a restart/reset whilst the other one stays unchanged.

---



## Part 3 — Put It Together

### Step 3 — `docker container ls` after starting `practice-web`

```
➜  ~ docker container run -d --name practice-web -p 8081:80 -e STUDENT_NAME="Arnold Muoneke" nginx:1.25
7be56509aeb643b5e917417c316cadba06cf7b563fcd7b981126372b0cec9917
➜  ~ docker container ls -a
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS                  PORTS                                     NAMES
7be56509aeb6   nginx:1.25   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds            0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   practice-web
69dd75373e06   alpine       "echo hi"                7 days ago      Exited (0) 7 days ago                                             zen_blackwell

```

Please disregard the other alpine container.

---

### Step 5 — Log streaming (`docker container logs -f`)

Had some intial difficulty with logging but found out I missed a parameter after already refreshing my browser like a mad man:

```
➜  ~ docker container logs -f practice-web
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/05/16 13:03:27 [notice] 1#1: using the "epoll" event method
2026/05/16 13:03:27 [notice] 1#1: nginx/1.25.5
2026/05/16 13:03:27 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2026/05/16 13:03:27 [notice] 1#1: OS: Linux 6.6.114.1-microsoft-standard-WSL2
2026/05/16 13:03:27 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/05/16 13:03:27 [notice] 1#1: start worker processes
2026/05/16 13:03:27 [notice] 1#1: start worker process 29
2026/05/16 13:03:27 [notice] 1#1: start worker process 30
2026/05/16 13:03:27 [notice] 1#1: start worker process 31
2026/05/16 13:03:27 [notice] 1#1: start worker process 32
2026/05/16 13:03:27 [notice] 1#1: start worker process 33
2026/05/16 13:03:27 [notice] 1#1: start worker process 34
2026/05/16 13:03:27 [notice] 1#1: start worker process 35
2026/05/16 13:03:27 [notice] 1#1: start worker process 36
172.17.0.1 - - [16/May/2026:13:05:45 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
2026/05/16 13:05:46 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8081", referrer: "http://localhost:8081/"
172.17.0.1 - - [16/May/2026:13:05:46 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8081/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
172.17.0.1 - - [16/May/2026:13:06:42 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
172.17.0.1 - - [16/May/2026:13:06:46 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
172.17.0.1 - - [16/May/2026:13:06:47 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
172.17.0.1 - - [16/May/2026:13:07:44 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
^Ccontext canceled

```

Please excuse the long terminal output since I already spammed the refresh button

---

### Step 6 — Shell inside container, verify `STUDENT_NAME`

```
➜  ~ docker container exec -it practice-web sh
# env | grep STUDENT_NAME
STUDENT_NAME=Arnold Muoneke

```

✅ The environment variable set at `docker run` time is visible inside the container.

---

### Step 7 & 8 — Custom index.html, verified in browser
Used the wrong echo command (>> append instead of >replace)
```
# echo "Hello from Arnold" >> /usr/share/nginx/html/index.html
# cat /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
Hello from Arnold
# exit

```

Browser output after refreshing `http://localhost:8081`:

```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.

Hello from Arnold

```



### Step 9 — Container IP address via `docker inspect`

```
➜  ~ docker container inspect -f '{{.NetworkSettings.IPAddress}}' practice-web
172.17.0.2

```



---

### Step 10, 11 & 12 — Stop and remove in one command; Disk usage and image removal

```
➜  ~ docker container rm -f practice-web
practice-web
➜  ~ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          6         1         308.1MB   299.6MB (97%)
Containers      1         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
➜  ~ docker image rm nginx:1.25
Untagged: nginx:1.25
Untagged: nginx@sha256:a484819eb60211f5299034ac80f6a681b06f89e65866ce91f356ed7c72af059c
Deleted: sha256:e784f4560448b14a66f55c26e1b4dad2c2877cc73d001b7cd0b18e24a700a070
Deleted: sha256:38ecd4de01b55beb32c596678b061db27f8ecb586096f031e4553869e52a1dc2
Deleted: sha256:168e1116c229abdf6cd9c0f07f82d40d53e358b1da4e1242a15101ece28ccb28
Deleted: sha256:0b7dc712f354a2312c1f1bb6380d8e23919baf0cc09f32a5a4595afb6c3a440b
Deleted: sha256:3ecb32a94af1f9a46cc259bf6cc677acd1fef2023f746e973862d1a8c9e31fe3
Deleted: sha256:b4c8fa0ae3e9f4f7d0ee49a9ac13d8aebd6f2d4a53e204e75f74e4bcb143132f
Deleted: sha256:cd5b03306052e1a435b98a34cd2579dc48f8f0ca280a147f19f066ddb6a0b81d
Deleted: sha256:5d4427064ecc46e3c2add169e9b5eafc7ed2be7861081ec925938ab628ac0e25
➜  ~ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         1         120.4MB   112MB (92%)
Containers      1         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B

```



## One thing that surprised me

I was surprised how unusable the container's shell is. Maybe it is the terminal emulator I'm using. 

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
