FROM oven/bun:1 as base
WORKDIR /app

# Install moon binary and git, moon docker setup needs git
RUN apt-get update 
RUN apt-get install -y curl
RUN curl -LJO https://moonrepo.dev/install/moon.sh 
RUN chmod +x moon.sh
RUN ./moon.sh
ENV PATH="/root/.moon/bin:$PATH"
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*

#### SKELETON
FROM base AS skeleton
COPY . .
RUN moon docker scaffold api

FROM base AS build
WORKDIR /app
# Copy toolchain
COPY --from=skeleton /root/.proto /root/.proto
# Copy workspace skeleton
COPY --from=skeleton /app/.moon/docker/workspace .

# Install toolchain and dependencies
RUN moon docker setup

# Copy source files
COPY --from=skeleton /app/.moon/docker/sources .

# Prune workspace
RUN moon run api:install
RUN moon run api:build

FROM gcr.io/distroless/base
WORKDIR /app
COPY --from=build /app/apps/api/build/server server
ENV NODE_ENV=production
ENTRYPOINT ["./server"]
EXPOSE 3000