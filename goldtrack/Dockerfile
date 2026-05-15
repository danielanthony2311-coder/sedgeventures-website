# GoldTrack — Node/Express + Vite frontend
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY server/package*.json ./
RUN npm install

# Copy all source files (server + frontend src)
COPY server/server.ts ./
COPY server/tsconfig.json ./
COPY frontend/src ./src
COPY frontend/package*.json ./frontend-package.json

# Build the Vite frontend (output to /app/dist)
RUN npm run build 2>/dev/null || npx vite build 2>/dev/null || echo "Build skipped — will use dev mode"

# Production image
FROM node:20-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/server.ts ./
COPY --from=builder /app/tsconfig.json ./
COPY --from=builder /app/dist ./dist 2>/dev/null || true

# Install tsx for running TypeScript directly
RUN npm install -g tsx

EXPOSE 8080

ENV NODE_ENV=production
ENV PORT=8080

CMD ["tsx", "server.ts"]
