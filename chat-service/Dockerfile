# Многоэтапная сборка для оптимизации размера образа
FROM node:20-alpine AS builder

# Создаем рабочую директорию
WORKDIR /usr/src/app

# Копируем файлы зависимостей
COPY package*.json ./
COPY prisma ./prisma/

# Устанавливаем зависимости
RUN npm ci --only=production && npm cache clean --force

# Генерируем Prisma Client
RUN npx prisma generate

# Копируем исходный код
COPY . .

# Собираем приложение
RUN npm run build

# Финальный образ
FROM node:20-alpine AS production

# Создаем пользователя для безопасности
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nestjs -u 1001

# Создаем рабочую директорию
WORKDIR /usr/src/app

# Копируем node_modules и собранное приложение
COPY --from=builder --chown=nestjs:nodejs /usr/src/app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /usr/src/app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /usr/src/app/prisma ./prisma

# Переключаемся на пользователя nestjs
USER nestjs

# Открываем порт
EXPOSE 3002

# Команда для запуска приложения
CMD ["node", "dist/main.js"]