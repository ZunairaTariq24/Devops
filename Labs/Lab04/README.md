# Lab 04: Simple Python Database Connections

**Student:** Zunaira Tariq  |  **Registration No.:** `2023-BCS-095`  |  **GitHub:** [@ZunairaTariq24](https://github.com/ZunairaTariq24)

## Table of Contents

- [Overview](#overview)
- [Learning Objectives](#learning-objectives)
- [Environment](#environment)
- [Task Progress](#task-progress)
- [Task Details](#task-details)
- [Exam Questions](#exam-questions)
- [Known Issues and Changes](#known-issues-and-changes)
- [Repository Structure](#repository-structure)
- [Key Takeaways](#key-takeaways)

## Overview

This lab connects Python to MySQL and MongoDB running in Docker. The same `db_connect.py` file was edited through four stages:

| Stage | Source of connection settings | Task |
|-------|-------------------------------|------|
| 1 | Values written in code (MySQL) | Task 4 |
| 2 | Values written in code (MongoDB) | Task 5 |
| 3 | Environment variables via `os.getenv()` | Task 6 |
| 4 | INI file via `configparser` | Task 7 |

Each program only connects, sends a request, and prints which database answered.

## Learning Objectives

- Explain the username, password, endpoint and database name used for a connection.
- Connect to MySQL and MongoDB with a short Python program.
- Explain the settings embedded in a MongoDB URI.
- Use `os.getenv()` to run unchanged code against dev, test and simulated prod.
- Use `configparser` to read settings from an INI credentials file.
- Confirm the target by reading the environment value stored in the database.

## Environment

| Component | Version / Detail |
|-----------|------------------|
| Host | Ubuntu (hostname `ubuntu`, user `zunairatariq24`) |
| Kernel | `<run: uname -r>` |
| Docker Engine | `<run: docker --version>` |
| Docker Compose | `<run: docker compose version>` |
| Python | 3.14 (virtual environment `.venv`) |
| MySQL | 8.4 (host port 3307) |
| MongoDB | 8.0 with a kernel workaround (see [Known Issues](#known-issues-and-changes)) |
| Python packages | `pymongo`, `mysql-connector-python` (check with `pip list`) |

## Task Progress

| # | Task | Status |
|---|------|--------|
| 1 | Fork and clone the Databases repository | ✅ |
| 2 | Install Docker, Compose and Python | ✅ |
| 3 | Start the database services | ✅ |
| 4 | MySQL with values in the code | ✅ |
| 5 | MongoDB with values in the code | ⬜ |
| 6 | MongoDB with `os.getenv()` | ⬜ |
| 7 | MongoDB with a credentials file | ⬜ |
| Q1 | Missing environment variable | ⬜ |
| Q2 | Which source the file version reads | ⬜ |
| Q3 | Wrong password in the file | ⬜ |

> Change ⬜ to ✅ as each task is finished.

## Task Details

### Task 1: Fork and Clone the Databases Repository

Forked the course repository to `ZunairaTariq24/Databases` and cloned it to the Ubuntu host.

### Task 2: Install Docker, Docker Compose and Python

Installed Docker Engine, the Compose plugin and Python, then created the `.venv` virtual environment and installed the required packages.

### Task 3: Start the Database Services

Started the MySQL and three MongoDB (dev, test, prod) services with Docker Compose and confirmed the published host ports.

| Service | Host port |
|---------|-----------|
| MySQL | 3307 |
| mongo-dev | 27017 |
| mongo-test | 27018 |
| mongo-prod | 27019 |

### Task 4: MySQL with Values in the Code

`db_connect.py` connected to MySQL at `127.0.0.1:3307` using values written directly in the code. Output confirmed database `school_mysql`, server `127.0.0.1 3307` and environment `dev`.

### Task 5: MongoDB with Values in the Code

`db_connect.py` was rewritten to build a MongoDB URI from hardcoded username, password, endpoint (`127.0.0.1:27017`) and database (`school_dev`), then ping the server and read the environment name from `environment_info`.

### Task 6: MongoDB with `os.getenv()`

The same file was changed to read `MONGO_USERNAME`, `MONGO_PASSWORD`, `MONGO_ENDPOINT` and `MONGO_DATABASE` from the shell environment. A short `if` check stops the program if any variable is missing. The unchanged program was run against dev (27017), test (27018) and simulated prod (27019).

### Task 7: MongoDB with a Credentials File

The program was changed to read the same four settings from an INI file with `configparser`. Credential files were created for each environment, and their permissions were restricted.

## Exam Questions

### Q1: A Missing Environment Variable

Ran the Task 6 program with one variable unset. The `if` check stopped the program with the message asking to set all four variables.

### Q2: Which Source Does the File Version Read?

Documented which source (environment variables or the INI file) the Task 7 program reads and how it selects it.

### Q3: A Wrong Password in the File

Changed the password in the INI file to a wrong value and recorded the authentication error, showing that a correct endpoint is not enough without matching credentials.

## Known Issues and Changes

**MongoDB 8.0 failed to start on this kernel; fixed with an environment variable**

- **Problem:** On a host running Linux kernel 6.19 or newer, the `mongo:8.0` containers exited immediately with: `MongoDB cannot start: Linux kernel versions 6.19 and newer has a known incompatibility with this version of MongoDB` ([SERVER-121912](https://jira.mongodb.org/browse/SERVER-121912)). The cause is an incompatibility between that kernel and the TCMalloc allocator bundled with MongoDB.
- **Checks:** `docker logs databases-mongo-dev-1` showed the error above. `grep -c avx /proc/cpuinfo` returned 2, so the CPU was not the problem.
- **Change:** Added a `docker-compose.override.yml` that sets `GLIBC_TUNABLES=glibc.pthread.rseq=1` on `mongo-dev`, `mongo-test` and `mongo-prod`. No other compose settings were changed. With `rseq=0` the containers still exited; with `rseq=1` they started and stayed up.
- **Note:** This is a community workaround, not an official fix. MongoDB's documentation says kernel 7.0.14 or later resolves the incompatibility.

```yaml
services:
  mongo-dev:
    environment:
      - GLIBC_TUNABLES=glibc.pthread.rseq=1
  mongo-test:
    environment:
      - GLIBC_TUNABLES=glibc.pthread.rseq=1
  mongo-prod:
    environment:
      - GLIBC_TUNABLES=glibc.pthread.rseq=1
```

**Virtual environment after a restart**

After the server restarted, `python` was not found because the virtual environment was no longer active. Fixed with `source .venv/bin/activate` (the prompt shows `(.venv)` when active).

## Repository Structure

```
DevOps/
└── Labs/
    └── Lab04/
        └── README.md
```

**Fork:** [ZunairaTariq24/Databases](https://github.com/ZunairaTariq24/Databases)

## Key Takeaways

- The connection block stays the same. Only the source of its settings changes.
- `os.getenv()` lets one program run against several targets without edits.
- An INI file keeps settings out of the code.
- A correct endpoint is not enough. Credentials must match too.
- Always confirm the target by reading the stored environment value, not just the connection message.
- When a container exits immediately, `docker logs` shows the real reason.

---

*Lab 04 completed on Ubuntu · Submitted to DevOps/Labs/Lab04*
