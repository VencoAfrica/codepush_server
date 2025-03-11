# Builder
FROM public.ecr.aws/docker/library/node:22-alpine3.21 as builder

WORKDIR /opt/azurite

# Install dependencies and build the app
COPY *.json LICENSE NOTICE.txt ./
COPY src ./src
COPY tests ./tests
RUN npm ci --unsafe-perm && npm run build

# Production image
FROM public.ecr.aws/docker/library/node:22-alpine3.21

ENV NODE_ENV=production

WORKDIR /opt/azurite

# Default Workspace Volume
VOLUME [ "/data" ]

COPY package*.json LICENSE NOTICE.txt ./
COPY --from=builder /opt/azurite/dist/ dist/

RUN npm pkg set scripts.prepare="echo no-prepare" && npm ci --unsafe-perm

# Blob Storage Port
EXPOSE 10000
# Queue Storage Port
EXPOSE 10001
# Table Storage Port
EXPOSE 10002

CMD ["azurite", "-l", "/data", "--blobHost", "0.0.0.0","--queueHost", "0.0.0.0", "--tableHost", "0.0.0.0"]
