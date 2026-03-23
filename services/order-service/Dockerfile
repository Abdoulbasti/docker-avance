FROM node:25-alpine3.22

WORKDIR /app

ENV NODE_ENV=production

COPY --chown=node:node package*.json ./
RUN if [ -f package-lock.json ]; then npm ci --omit=dev; else npm install --omit=dev; fi \
	&& npm cache clean --force

COPY --chown=node:node . .

USER node

EXPOSE 3002

CMD ["npm", "start"]
