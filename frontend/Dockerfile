# Stage 1 — build the app
FROM dhi.io/node:24-alpine-dev AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci --legacy-peer-deps

# Install vite explicitly if not already in package.json
RUN npm list vite > /dev/null 2>&1 || npm install --no-save vite@5.4.11

COPY . .

RUN npm run build

# Create non-root user in build stage
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Stage 2 — distroless runtime (minimal, secure)
FROM dhi.io/node:24-alpine

WORKDIR /app

# Copy built files from builder
COPY --from=build /app/dist ./dist

# Copy all node_modules from builder (includes vite)
COPY --from=build /app/node_modules ./node_modules

COPY vite.preview.config.js ./vite.config.js

# Copy user from build stage
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /etc/group /etc/group

EXPOSE 4173

# Switch to non-root user
USER appuser

# Use exec form for proper signal handling
ENTRYPOINT ["node", "--no-warnings"]
CMD ["./node_modules/.bin/vite", "preview", "--host", "0.0.0.0", "--port", "4173"]
