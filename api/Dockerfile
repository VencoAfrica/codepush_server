FROM public.ecr.aws/docker/library/node:22-alpine3.21 AS builder

WORKDIR /usr/src/app

# Copy dependency definitions
COPY package.json package-lock.json ./

# Install dependencies
RUN npm ci

# Copy the source code
COPY . .

# Build the application
RUN npm run build

# Production image
FROM public.ecr.aws/docker/library/node:22-alpine3.21

WORKDIR /usr/src/app

# Copy built application from builder stage
COPY --from=builder /usr/src/app /usr/src/app

# Expose the port
EXPOSE 3000

RUN chown -R node:node .

# Use non-root user
USER node

# Start the application
CMD [ "npm", "run", "start:env" ]