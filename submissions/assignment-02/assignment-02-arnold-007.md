# Assignment 02 Running Containers & Images Deep Dive - Arnold Muoneke

---

**GitHub username:** arnold-007
**Date completed:** YYYY-MM-DD
**Language chosen:** Python | Node.js

---


## 1. The image I built

- Final image ID: 04b5a8dc7823 `<docker image inspect -f '{{.Id}}' cohort-greet:0.1.0>`
- Image size: 124MB `<from docker image ls>`
- Number of layers: 6 Layers. I used the Dockerfile entries. Then I observed the IDs during the build process and correlated them to the table entries of docker image history `<from docker image history | wc -l minus 1 for header>`

### Dockerfile
```
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
ENV PORT=8000
EXPOSE 8000
CMD ["python", "app.py"]

```

### .dockerignore
```
.git
.gitignore
node_modules
__pycache__
*.pyc
*.log
README.md
```

## 2. Answers to the 8 questions

**Q1 — what `.dockerignore` affects:** `.dockerignore` affects all files/directories named within it and prevent them from being involved in the build stage. In other words, COPY cannot make use of the files. An exception to dockerignore would be Files already in previous layers of the image. I do not understand the reason for adding .git and .gitignore to the dockerignore file but a quick search revealed that this preents redundancy and guards against exposure of sensitive credentials, commits, source code history.

**Q2 — what is the image ID a hash of:**  I love this question because it was my a-ha moment for question 1. The Image ID 04b5a8dc7823 is a hash of Step 6/6 the topmost image layer during the build. This has helped me better visualize the build process as stacking one onion layer/matroshka doll after the other.

**Q3 — largest layer and why:**  After running the history command I noticed these two: 
```
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
...............
<missing>      41 hours ago   RUN /bin/sh -c set -eux;   savedAptMark="$(a…   42MB      buildkit.dockerfile.v0
      .........
<missing>      3 days ago     # debian.sh --arch 'amd64' out/ 'trixie' '@1…   78.6MB    debuerreotype 0.17
```
The largest layer is the one containing the Debian OS Image at 78.6MB. It contains the whole opersting system and is the foundation of the further layers. This makes sense to me.

**Q4 — `--memory 64m` shows up as what value:**
It shows up in bytes via inspect and not Mebibytes(MiB). Initial suspicion was to avoid over-computing (conversion from one number system to another). I however was curious and found out even more reasons why this is the case. Apparently ocker prints memory in bytes because:
Kernels require bytes (cgroups, API uses them), No unit ambiguity (avoids MB vs. MiB confusion), Precise & exact (no rounding errors from fractions)

**Q5 — PID of my app inside the container:** 
ps aux unfortunately didnt work inside the container and I tried installing with `apt-get install procps`. This also did not work! Alternatively, I used `docker top greet`.
`➜  assignment-02 docker top greet
UID                 PID                 PPID                C                   STIME               TTY                 TIME                                               CMD
root                77782               77758               0                   May21               ?                   00:00:                               00            python app.py
` The PID of my app stems from a parent PID 77758. The app process was initiated by a root user. That's all I can say about this.

**Q6 — `stop` vs `kill`, and which for a database:** `docker stop` cleanly terminates a container without use of force. `docker kill` however, is a forceful abrupt termination of a container. An everday analogy would be shutting down a windows laptop. The graceful method would be to click the shut down button and expect all running tasks to shutdown before the ultimate termination. The "kill" in this case, would be impatiently holding down the power button to forcefully shutdown your windows system. It is primitive but necessary in some cases like blue-screen or glitch scenario. An equivalent of that in Docker would require the use of `docker kill`.

For databases, I would definitely recommend docker stop. This would prevent corruption of databases and also enable clean recovery during the next database start

**Q7 — what same-IMAGE-ID-across-tags proves:**  Tagging an image multiple times, in this case 3 times, does not necessarily mean you will have three images. You do only have 3 pointers pointing towards the same image.

**Q8 — tag vs digest mutability:** Unlike the tag, the digest is immutable and therefore will not give you the same image. It will keep pointing at the original image you tied it to.

## 3. Evidence

Paste the **command + output** for each of these. Use fenced code blocks. Trim long output to the relevant lines.


- Command: `docker image history cohort-greet:0.1.0`
  Output:
  `IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
04b5a8dc7823   24 hours ago   /bin/sh -c #(nop)  CMD ["python" "app.py"]      0B
030b991fd8df   24 hours ago   /bin/sh -c #(nop)  EXPOSE 8000                  0B
f8901e6ad755   24 hours ago   /bin/sh -c #(nop)  ENV PORT=8000                0B
69c5dd42caec   24 hours ago   /bin/sh -c #(nop) COPY file:3f4d64ef73b9ac87…   741B
89b3f9420bf7   24 hours ago   /bin/sh -c #(nop) WORKDIR /app                  0B
1455a91ef4da   41 hours ago   CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      41 hours ago   RUN /bin/sh -c set -eux;  for src in idle3 p…   36B       buildkit.dockerfile.v0
<missing>      41 hours ago   RUN /bin/sh -c set -eux;   savedAptMark="$(a…   42MB      buildkit.dockerfile.v0
<missing>      41 hours ago   ENV PYTHON_SHA256=272179ddd9a2e41a0fc8e42e33…   0B        buildkit.dockerfile.v0
<missing>      41 hours ago   ENV PYTHON_VERSION=3.11.15                      0B        buildkit.dockerfile.v0
<missing>      41 hours ago   ENV GPG_KEY=A035C8C19219BA821ECEA86B64E628F8…   0B        buildkit.dockerfile.v0
<missing>      41 hours ago   RUN /bin/sh -c set -eux;  apt-get update;  a…   3.81MB    buildkit.dockerfile.v0
<missing>      41 hours ago   ENV LANG=C.UTF-8                                0B        buildkit.dockerfile.v0
<missing>      41 hours ago   ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      3 days ago     # debian.sh --arch 'amd64' out/ 'trixie' '@1…   78.6MB    debuerreotype 0.17
`
  
- Command: `docker container run` (Part 2.2 — the detached run with all flags)
   Output: after run command was this `4a1bd447cc945833f4ade8ac8b1d95236f27f138a9afe3b6136c231ff75499cc` meanwhile on a separate terminal this was the curl output `hi, Arnold Muoneke — 2026-05-21T21:44:28.073206Z`
      
- Command: `docker container logs greet` after 2 curl requests
  Output: `listening on :8000
[req] 172.17.0.1 "GET / HTTP/1.1" 200 -
[req] 172.17.0.1 "GET / HTTP/1.1" 200 -
[req] 172.17.0.1 "GET / HTTP/1.1" 200 -
`
- Command: `docker container stats --no-stream greet`
  Output: `
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O     PIDS
4a1bd447cc94   greet     0.02%     13.36MiB / 64MiB    20.88%    2.84kB / 1.94kB   0B / 1.95MB   1
`
- Command: `docker container inspect -f '{{.HostConfig.RestartPolicy.Name}} {{.HostConfig.Memory}}' greet`
  Output: `unless-stopped
67108864`
- Command: `docker image ls cohort-greet` (showing the three tags from Part 3.1)
  Output:
  `REPOSITORY     TAG       IMAGE ID       CREATED        SIZE
cohort-greet   0.1       04b5a8dc7823   31 hours ago   124MB
cohort-greet   0.1.0     04b5a8dc7823   31 hours ago   124MB
cohort-greet   latest    04b5a8dc7823   31 hours ago   124MB`

- (Optional) URL of your pushed image on Docker Hub / GHCR

## 4. One thing that surprised me

What has surprised me this time around is how much one underestimates the probability of errors arising from general commands inside or outside the container. The ps aux for example and other commands like the forest, pstree commands etcetera.

## 5. One thing I'm still unsure about

Tracing parent Process IDs with their child Processes' IDs.

---
## Checklist
- [x] My report lives at `submissions/assignment-02/assignment-02-arnold-007.md`
- [x] My PR changes exactly one file
- [x] I answered all 8 questions
- [x] I included the requested command outputs as evidence
- [x] I cleaned up containers and images on my machine
- [x] I read CONTRIBUTING.md and followed the branch + commit conventions

---
## Time spent
~Lost Count honestly.
