FROM node:18-bookworm-slim

WORKDIR /app

ENV NODE_ENV=production

COPY --chown=node:node package*.json ./
RUN if [ -f package-lock.json ]; then npm ci --omit=dev; else npm install --omit=dev; fi \
	&& npm cache clean --force

COPY --chown=node:node . .

USER node

EXPOSE 3001

CMD ["npm", "start"]
