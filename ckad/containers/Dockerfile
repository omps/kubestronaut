# use a minimal base image
FROM node:14-alpine AS build

# set a working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package.json package-lock.json ./
RUN npm install

#copy the rest of the application
COPY . .

#build the app
RUN npm run build

#use a smaller image for runtime
FROM node:14-alpine

#set a non-root user
RUN addgroup app && adduser -S -G app ap
USER app

#set the working dir
WORKDIR /app

#copy only the build artifacts
COPY --from=build /app/build /app

#start the app
CMD ["npm", "start"]
