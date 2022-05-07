FROM node:alpine AS builder
WORKDIR /app
RUN yarn set version berry
COPY package.json ./
RUN yarn
COPY . .
RUN yarn build

FROM node:alpine
WORKDIR /app
COPY --from=builder /app/.next ./
CMD ["yarn", "start"]