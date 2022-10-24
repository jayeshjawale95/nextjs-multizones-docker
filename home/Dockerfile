# Install dependencies only when needed
FROM node:17-alpine AS deps

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat curl

ARG BLOG_URL
ENV BLOG_URL ${BLOG_URL}

# Set working directory
WORKDIR /opt/app

# Need to change to correct app folder name for each flow [ deps, builder, runner ]
# set your {APP_NAME}
ENV {APP_NAME} home

# Create apps folder
RUN mkdir -p apps

# Copy related packages and apps
COPY ./home/pages ./apps/${APP_NAME}/pages
COPY ./home/public ./apps/${APP_NAME}/public
COPY ./home/components ./apps/${APP_NAME}/components
COPY ./home/next.config.js ./apps/${APP_NAME}/next.config.js

# Add lockfile and package.json's of isolated subworkspace
COPY ./home/package.json ./apps/${APP_NAME}/package.json
# COPY ./home/package-lock.json ./apps/${APP_NAME}/package-lock.json

# ----------------------------------------------------
# 2. Rebuild the source code only when needed
FROM node:17-alpine AS builder
RUN apk add --no-cache libc6-compat curl
ARG BLOG_URL
ENV BLOG_URL ${BLOG_URL}

WORKDIR /opt/app

# Build the project and its dependencies
ENV NEXT_TELEMETRY_DISABLED 1

# set your {APP_NAME}
ENV {APP_NAME} home

COPY --from=deps /opt/app/ .
WORKDIR /opt/app/apps/${APP_NAME}

RUN npm install
RUN npm run build

# ------------------------------------
# 3. Production image, copy all the files and run next
FROM node:17-alpine AS runner
RUN apk add --no-cache libc6-compat curl
ARG BLOG_URL
ENV BLOG_URL ${BLOG_URL}

WORKDIR /opt/app

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry.
ENV NEXT_TELEMETRY_DISABLED 1

ENV NODE_ENV=production

# set your {APP_NAME}
ENV {APP_NAME} home

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# You only need to copy next.config.js if you are NOT using the default configuration
COPY --from=builder /opt/app/apps/${APP_NAME}/next.config.js ./
COPY --from=builder /opt/app/apps/${APP_NAME}/public ./public
COPY --from=builder /opt/app/apps/${APP_NAME}/components ./components
COPY --from=builder --chown=nextjs:nodejs /opt/app/apps/${APP_NAME}/.next ./.next
COPY --from=builder /opt/app/apps/${APP_NAME}/node_modules ./node_modules
COPY --from=builder /opt/app/apps/${APP_NAME}/package.json ./package.json

USER nextjs

EXPOSE 4001

ENV PORT 4001

CMD ["npm", "run", "start", "--", "-p", "4001"]
