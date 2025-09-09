# ---- Base image for all stages ----
FROM node:22-alpine AS base
WORKDIR /app
ENV NODE_ENV=production

# Install libc6-compat for some native deps
RUN apk add --no-cache libc6-compat \
  && corepack enable \
  && corepack prepare pnpm@latest --activate

# ---- Dependencies (install all deps, including dev) ----
FROM base AS deps
RUN apk add --no-cache curl
RUN apk add --no-cache bash
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .

# ---- Build ----
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm run build
RUN pnpm prune --prod

# ---- Runtime ----
FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup -S nodejs && adduser -S nodejs -G nodejs \
  && corepack enable \
  && corepack prepare pnpm@latest --activate

COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist

ENV PORT=3000
EXPOSE 30010

USER nodejs

# 👇 Run only the "users" app inside monorepo
CMD ["node", "dist/index.js"]
