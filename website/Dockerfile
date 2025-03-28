# Stage 1: Install Dependencies
ARG NODE=node:20-alpine 
FROM ${NODE} AS deps

RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY package*.json ./
RUN npm install

# Stage 2: Build
FROM ${NODE} AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

# Stage 3: Production Runner
FROM ${NODE} AS runner

WORKDIR /app

ENV NODE_ENV production
ENV PORT 3000
ENV NEXT_TELEMETRY_DISABLED 1

# Install PM2 globally
RUN npm install -g pm2

# Create a non-root user
RUN addgroup --system --gid 1001 nodegroup
RUN adduser --system --uid 1001 appuser

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder --chown=appuser:nodegroup /app/.next/standalone ./
COPY --from=builder --chown=appuser:nodegroup /app/.next/static ./.next/static

USER appuser

EXPOSE 3000

# Start the application using PM2
CMD ["pm2-runtime", "node", "--", "server.js"]