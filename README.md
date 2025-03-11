# Visual Studio App Center CodePush Standalone Version

## Overview

This repository provides a self-hosted version of CodePush, a service for managing updates to React Native applications. It includes both the CodePush API and Azurite (a local Azure Storage emulator) in a single containerized setup.

### Official Repository

For reference, visit the official CodePush Server repository: [Microsoft CodePush Server](https://github.com/microsoft/code-push-server).

---

## Getting Started

## Containerization

A `Dockerfile` is provided in the root directory of the repository, which installs both the CodePush API and Azurite in a single container.

### Starting the Container

To launch the `codepush_api_plus_azurite` container, follow these steps:

1. **Connect to the Server:** Use AWS Session Manager or SSH:

   ```sh
   ssh ubuntu@<EC2_PUBLIC_IP>
   ```

2. **Navigate to the Project Directory:**

   ```sh
   cd /home/ubuntu/cz_mw_code_push_server
   ```

3. **Pull the Latest Code (If Changes Exist):**

   ```sh
   git pull
   ```

4. **Start the Container:**

   ```sh
   docker-compose -f docker-compose.yaml up --no-deps --build -d
   ```

---

## Monitoring & Logs

### Viewing Container Logs

- **Live Logs:**

  ```sh
  docker logs -f codepush_api_plus_azurite
  ```

- **Recent Logs (e.g., past 1 minute):**

  ```sh
  docker logs -f codepush_api_plus_azurite --since 1m
  ```

---

## Deployment Information

The CodePush service is hosted on an AWS EC2 instance with the following details:

- **Private IP:** `XX.XX.XX.XX`
- **Public IP:** `XX.XX.XX.XX`

### Services & Endpoints

All API and storage services are reverse-proxied through Nginx and publicly accessible via the following domains:

| Service       | URL                                                                  |
| ------------- | -------------------------------------------------------------------- |
| CodePush API  | [https://codepush.example.com](https://codepush.example.com)             |
| Blob Storage  | [https://codepush.example.com/blob](https://codepush.example.com/blob)   |
| Queue Storage | [https://codepush.example.com/queue](https://codepush.example.com/queue) |
| Table Storage | [https://codepush.example.com/table](https://codepush.example.com/table) |

---

## Conclusion

This setup ensures a seamless experience for hosting a self-managed CodePush service. With proper containerization and reverse proxy configurations, updates are efficiently managed and delivered to mobile applications.

---

**Maintainer:** Sajjad Talib -- DevOps Engineer -- sajjadtalib29@gmail.com
