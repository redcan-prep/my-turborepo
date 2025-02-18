FROM node:20-slim as base
COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.9.0 /lambda-adapter /opt/extensions/lambda-adapter

ENV PNPM_HOME="/var/lib/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
ENV AWS_LWA_PORT=5005

RUN npm install -g corepack@latest
RUN corepack enable

RUN pnpm add -g turbo

FROM base AS builder
WORKDIR /app

COPY . .
RUN turbo prune api --docker

FROM base AS installer
WORKDIR /app

COPY --from=builder /app/out/json/ .
RUN pnpm install

COPY --from=builder /app/out/full/ .
RUN pnpm run build

FROM base AS runner
WORKDIR /app

COPY --from=installer /app ${LAMBDA_TASK_ROOT}

CMD ["node",  "apps/api/dist/src/main.js"]