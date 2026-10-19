# ============ 构建阶段：npm 依赖安装 + vite 打包 ============
FROM node:22-alpine AS build

WORKDIR /app

# 先只拷贝清单文件，命中依赖缓存层
COPY package.json package-lock.json ./

RUN --mount=type=cache,target=/root/.npm \
    npm ci

# 再拷贝源码
COPY . .

# npm run build = vue-tsc -b && vite build
# 若类型检查报错导致构建失败，改用 npx vite build 跳过类型检查
RUN npm run build

# ============ 运行阶段：nginx 托管静态产物 ============
FROM nginx:1.27-alpine

# 用容器专用的 nginx 配置覆盖默认配置（反代目标为 compose 服务名 erp-api）
COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
