# ---------- builder ----------
FROM oven/bun AS builder
WORKDIR /app

ARG NEXT_PUBLIC_BACKEND_URL
ARG NEXT_PUBLIC_FRONTEND_URL

ENV NEXT_PUBLIC_BACKEND_URL=${NEXT_PUBLIC_BACKEND_URL}
ENV NEXT_PUBLIC_FRONTEND_URL=${NEXT_PUBLIC_FRONTEND_URL}
ENV NEXT_TELEMETRY_DISABLED=1
ENV NODE_ENV=production

COPY . .
RUN bun install

# Build only when targeting production
WORKDIR /app/apps/web
RUN bun run build
RUN bash scripts/handle-nextjs-standalone-build.sh


# ---------- runner (for production) ----------
FROM oven/bun AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000
ENV NEXT_PUBLIC_BACKEND_URL=${NEXT_PUBLIC_BACKEND_URL}
ENV NEXT_PUBLIC_FRONTEND_URL=${NEXT_PUBLIC_FRONTEND_URL}

RUN groupadd -r nodejs && useradd -r -g nodejs nodejs

COPY --from=builder --chown=nodejs:nodejs /app/apps/web/.next ./
COPY --from=builder --chown=nodejs:nodejs /app/apps/web/public ./public
COPY --from=builder --chown=nodejs:nodejs /app/packages/db ./packages/db

USER nodejs
EXPOSE ${PORT}
CMD ["bun", "run", "./build-start-folder/apps/web/server.js"]


# ---------- dev (for local development) ----------
FROM oven/bun AS dev
WORKDIR /app

ENV NODE_ENV=development
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000

COPY . .
RUN bun install
WORKDIR /app/apps/web

EXPOSE 3000
CMD ["bun", "run", "dev"]
