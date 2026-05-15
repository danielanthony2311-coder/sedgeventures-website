FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY server/package*.json ./
RUN npm install --production

# Copy server
COPY server/server.ts ./
COPY server/tsconfig.json ./

# Copy pre-built frontend (no build step needed)
COPY dist ./dist

# Install tsx to run TypeScript
RUN npm install -g tsx

EXPOSE 8080
ENV NODE_ENV=production
ENV PORT=8080

CMD ["tsx", "server.ts"]
