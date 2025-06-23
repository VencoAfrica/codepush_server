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
   cd /home/ubuntu/codepush_server
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

## Handle Local Endpoint Challenges
### Issue Summary

When pushing release changes via the CodePush CLI, the files upload successfully as expected. However, when attempting to download these files from mobile applications, the CodePush API returns a URL in the following format:

`http://localhost:10000/<blob-storage-path>`
Since mobile applications cannot access the localhost, they fail to download the necessary files.

Root Cause
The issue arises because CodePush, when running in a local development environment with Azurite (a local Azure Storage emulator), automatically generates storage endpoints using the Azure SDK with UseDevelopmentStorage=true. This setting causes the CodePush API to return URLs pointing to localhost:10000, which are inaccessible to mobile clients.

### Solution
In this case, One way to solve is to go with the approach of replacing the localhost URL from the mobile application code.

The following React Native code snippet demonstrates how to handle this issue by intercepting the downloadUrl from CodePush and replacing http://127.0.0.1:10000 with a valid external URL before downloading the update:

```javascript
const checkForCodePushUpdate = async () => {
    try {
      const update: RemotePackage | null = await codePush.checkForUpdate();

      if (update) {
        // Modify the URL if it points to the local Azurite storage
        if (update.downloadUrl.includes('http://127.0.0.1:10000')) {
          update.downloadUrl = update.downloadUrl.replace(
            'http://127.0.0.1:10000',
            'https://codepush.example.com/blob',
          );
        }
        downloadAndInstallUpdate(update);
      }
    } catch (error) {
      console.error('Error in CodePush sync:', error);
    }
};

const downloadAndInstallUpdate = async (update: RemotePackage) => {
    try {
      if (update.failedInstall) {
        codePush.clearUpdates();
      }
      const downloadedPackage = await update.download();

      await downloadedPackage.install(codePush.InstallMode.IMMEDIATE);
      reportCodePushInstall(downloadedPackage);
    } catch (error) {
      console.error('Error installing the update:', error);
    }
};
```
### Outcome

With this fix in place, the CodePush updates will be downloaded successfully, ensuring that mobile applications can retrieve and install the latest changes without encountering inaccessible localhost URLs.

## Using the cli to connect to the server

To connect to the CodePush server using the CLI, you need to set up the cli on your local machine. Follow these steps:

Clone this repository to your local machine:

change directory to the cli folder in the cloned repository:

run npm install to install the necessary dependencies:

```bash
npm install .
```

run npm build to build the CLI:

```bash
npm run build .
```

Optionally, after building you can install the CLI globally to use it from anywhere:

```bash
npm install -g .
```

Now you can use the CodePush CLI to connect to your self-hosted server. 
To connect to the server, use the following command:
```bash
npm run codepush -- register <server: url>
```

See the [CodePush CLI documentation](./cli/README.md) for more details on how to use the CLI commands.


## Conclusion

This setup ensures a seamless experience for hosting a self-managed CodePush service. With proper containerization and reverse proxy configurations, updates are efficiently managed and delivered to mobile applications.