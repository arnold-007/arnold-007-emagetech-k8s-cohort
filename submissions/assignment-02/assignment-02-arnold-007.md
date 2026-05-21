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
**Q4 — `--memory 64m` shows up as what value:** ...
**Q5 — PID of my app inside the container:** ...
**Q6 — `stop` vs `kill`, and which for a database:** ...
**Q7 — what same-IMAGE-ID-across-tags proves:** ...
**Q8 — tag vs digest mutability:** ...

## 3. Evidence

Paste the **command + output** for each of these. Use fenced code blocks. Trim long output to the relevant lines.

- Command: `docker image history cohort-greet:0.1.0` Output: `IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
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
- `docker container run` (Part 2.2 — the detached run with all flags)
- `docker container logs greet` after 2 curl requests
- `docker container stats --no-stream greet`
- `docker container inspect -f '{{.HostConfig.RestartPolicy.Name}} {{.HostConfig.Memory}}' greet`
- `docker image ls cohort-greet` (showing the three tags from Part 3.1)
- (Optional) URL of your pushed image on Docker Hub / GHCR

## 4. One thing that surprised me

(2–4 sentences. Be specific — "the image was bigger than I expected because the Python base image alone is 130MB and my app is ~2KB" is a good answer; "Docker is cool" is not.)

## 5. One thing I'm still unsure about

(One sentence. This is what your TA will follow up on in office hours.)

---
## Checklist
- [ ] My report lives at `submissions/assignment-02/assignment-02-arnold-007.md`
- [ ] My PR changes exactly one file
- [ ] I answered all 8 questions
- [ ] I included the requested command outputs as evidence
- [ ] I cleaned up containers and images on my machine
- [ ] I read CONTRIBUTING.md and followed the branch + commit conventions

---
## Time spent
~<N> hours
