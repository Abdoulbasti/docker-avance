FROM node:25-alpine3.22 AS node-builded

WORKDIR /app

COPY package*.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:25-alpine3.22 AS runtime

ENV NODE_ENV=production
WORKDIR /app

COPY package*.json package-lock.json ./
RUN npm ci --omit=dev && npm cache clean --force

COPY --from=node-builded /app/dist ./dist
COPY --from=node-builded /app/server.cjs ./server.cjs

RUN chown -R node:node /app
USER node

EXPOSE 8080

CMD ["node", "server.cjs"]