FROM node:22-alpine AS build
ENV CI=true
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install --global pnpm@9.11.0 && pnpm install --frozen-lockfile
COPY . .
RUN pnpm exec vue-tsc --noEmit && \
    npm_package_version="$(node -p 'require("./package.json").version')" \
    NODE_OPTIONS=--max-old-space-size=1536 pnpm exec vite build

FROM nginx:stable-alpine@sha256:0985e772fb9f729e6fa0980da05fca5d9c468e870eed43071545afa9d2e27d94
ARG SOURCE_REV
LABEL org.opencontainers.image.source="https://github.com/ziran258/it-tools" \
      org.opencontainers.image.revision="${SOURCE_REV}"
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
