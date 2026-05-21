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
\`\`\`dockerfile
<paste your final Dockerfile here>
FROM python:3.11-slim



WORKDIR /app
COPY app.py .


ENV PORT=8000
EXPOSE 8000

CMD ["python", "app.py"]

\`\`\`

### .dockerignore
\`\`\`
<paste your .dockerignore here>
\`\`\`

## 2. Answers to the 8 questions

**Q1 — what `.dockerignore` affects:** ...
**Q2 — what is the image ID a hash of:** ...
**Q3 — largest layer and why:** ...
**Q4 — `--memory 64m` shows up as what value:** ...
**Q5 — PID of my app inside the container:** ...
**Q6 — `stop` vs `kill`, and which for a database:** ...
**Q7 — what same-IMAGE-ID-across-tags proves:** ...
**Q8 — tag vs digest mutability:** ...

## 3. Evidence

Paste the **command + output** for each of these. Use fenced code blocks. Trim long output to the relevant lines.

- `docker image history cohort-greet:0.1.0`
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
