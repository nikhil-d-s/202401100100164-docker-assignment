# Docker & Container Security — Practical Assessment

## Student Details

- **Registration Number:** `202401100100164`
- **Name:** `Nikhil Dubey`
- **Port:** `8071`
- **Docker Hub Username:** `nikhildevs`
- **Docker Hub Image:** `nikhildevs/fest-backend`
- **Repository:** `202401100100164-docker-assessment`

## Project Overview

This project implements a containerized three-tier registration application using Docker, Docker Compose, Flask, PostgreSQL, and NGINX.

The application consists of:

- **Frontend:** NGINX serves the registration page.
- **Backend:** Python Flask API handles health checks and registration requests.
- **Database:** PostgreSQL stores registration records persistently using a named Docker volume.

The services are separated using Docker networks so the frontend cannot directly access the database.

## Technology Stack

- Docker and Docker Compose
- Python and Flask
- PostgreSQL
- NGINX
- Trivy for vulnerability scanning
- Cosign for container image signing and verification
- Git and GitHub

## Application Configuration

| Setting | Value |
|---|---|
| Registration number | `202401100100164` |
| Application port | `8071` |
| Backend container port | `5000` |
| Database name | `festpass` |
| Database user | `festpass` |
| Frontend URL | `http://localhost:8071` |
| Health endpoint | `/api/health` |
| Registrations endpoint | `/api/registrations` |

## Run the Application

Create a local `.env` file using `.env.example` as a template and set a suitable database password.

Start the application:

```bash
docker compose up -d --build
```

Check service status:

```bash
docker compose ps
```

Check backend health:

```bash
curl http://localhost:8071/api/health
```

Open the frontend at `http://localhost:8071`.

Stop the application:

```bash
docker compose down
```

The PostgreSQL named volume is retained when running `docker compose down` without the `-v` option. This allows registration data to persist across container recreation.

## Docker Image Hardening

### Original Image — `fest-backend:0.1`

The initial image used the `python:3.9` base image and the original application dependencies.

The initial image was scanned using Trivy, and the results were saved to:

`reports/trivy-before.txt`

The container's runtime identity was checked using:

```bash
docker run --rm --entrypoint id fest-backend:0.2
```

The container runs as `appuser` with UID `999`, rather than root.

The after-scan results are saved to:

`reports/trivy-after.txt`

## Vulnerability and Image Size Comparison

Fill in the CRITICAL and HIGH counts using the actual results from the saved Trivy reports. Do not substitute total vulnerability counts for severity-specific counts.

| Metric | `fest-backend:0.1` | `fest-backend:0.2` |
|---|---:|---:|
| CRITICAL vulnerabilities | 234 | 0 |
| HIGH vulnerabilities | 2243 | 44 |
| Image content size | 412 MB | 54.2 MB |

The hardened image reduced the reported image content size from 412 MB to 54.2 MB, a reduction of approximately 86.8%.

### Required CRITICAL Vulnerability Gate

The following command was executed against the hardened image:

```bash
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

**Observed exit status:** `0`

This indicates that the scan passed the specified CRITICAL vulnerability gate with unfixed vulnerabilities excluded. It does not imply that the image has no HIGH, MEDIUM, or LOW vulnerabilities.

## Evidence and Reports

- `reports/trivy-before.txt` — Trivy report for `fest-backend:0.1`.
- `reports/trivy-after.txt` — Trivy report for `fest-backend:0.2`.
- `screenshots/` — Assessment screenshots, using the required evidence filenames.

## Docker Hub Release

- **Docker Hub repository:** https://hub.docker.com/r/nikhildevs/fest-backend
- **Release version:** `1.0.0`
- **Published tags:**
  - `nikhildevs/fest-backend:1.0.1`
- **Image digest:** `sha256:15358b6b63a01b174ceb0efc56d2563dc4f7391b7867c52970f72fcafee73104`

Both tags were pushed successfully and returned the same digest.

## Cosign Image Signing and Verification

### Signed release — `1.0.0`

The Docker image `nikhildevs/fest-backend:1.0.1` was signed using Cosign with a password-protected private key. The signature was created and verified against the image digest.

- **Image digest:** `sha256:15358b6b63a01b174ceb0efc56d2563dc4f7391b7867c52970f72fcafee73104`
- **Verification key:** `cosign.pub`
- **Result:** Signature verification succeeded.

### Unsigned release — `1.0.1`

The image `nikhildevs/fest-backend:1.0.1` was built and pushed without signing it.

The following verification command was executed:

```bash
cosign verify --key cosign.pub nikhildevs/fest-backend:1.0.1
```

- **Observed result:** `Error: no signatures found`
- **Exit code:** `10`
- **Expected outcome:** Verification failed because the image was unsigned.

The private signing key (`cosign.key`) is excluded from version control. Only the public verification key (`cosign.pub`) is intended to be committed.

## AI Tools Used

- **ChatGPT (OpenAI):** Used for assistance understanding the assessment, Docker configuration, image hardening, vulnerability scanning, and documentation.

## Completion Status

- [x] Initial backend image built and tested.
- [x] Three-tier application configured with Docker Compose.
- [x] Database persistence and network isolation tested.
- [x] Original image vulnerability scan saved.
- [x] Hardened image built and verified to run as a non-root user.
- [x] Hardened image CRITICAL vulnerability gate passed.
- [x] Actual before-and-after CRITICAL and HIGH counts entered in the comparison table.
- [x] Remaining assessment stages, including image publication and Cosign signing, completed and documented.

## Notes

Only `.env.example` is committed to the repository. The local `.env` file and private signing keys must not be committed.

Update this README as the remaining assessment stages are completed.
