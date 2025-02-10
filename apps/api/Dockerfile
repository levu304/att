FROM oven/bun:slim AS base
# Install moon binary
RUN curl -fsSL https://moonrepo.dev/install/moon.sh | bash
RUN export PATH="$HOME/.moon/bin:$PATH"

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

# Cache packages
# COPY package.json package.json
# COPY bun.lockb bun.lockb

# COPY /apps/server/package.json ./apps/server/package.json
# COPY /packages/config/package.json ./packages/config/package.json

# RUN bun install

# Copy source files
COPY --from=skeleton /app/.moon/docker/sources .
# Build something (optional)
RUN moon run api:install
RUN moon run api:build
# Prune workspace
RUN moon docker prune

# COPY /apps/server ./apps/server
# COPY /packages/config ./packages/config

# ENV NODE_ENV=production

# RUN bun build \
# 	--compile \
# 	--minify-whitespace \
# 	--minify-syntax \
# 	--target bun \
# 	--outfile server \
# 	./src/index.ts

FROM gcr.io/distroless/base

WORKDIR /app

COPY --from=build /app/server server

ENV NODE_ENV=production

CMD ["./server"]

EXPOSE 3000