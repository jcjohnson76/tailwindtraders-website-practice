# syntax=docker/dockerfile:1.6

# ─────────────────────────────
# Stage 1: Build the ClientApp
# ─────────────────────────────
FROM node:18-alpine AS build-node
WORKDIR /ClientApp

# Install build toolchain + deps for npm modules that need native builds
COPY ClientApp/package*.json ./
RUN apk add --no-cache python3 make g++ \
  && npm ci

# Build the front-end
COPY ClientApp/ .
RUN npm run build

# ─────────────────────────────
# Stage 2: Build & publish .NET
# ─────────────────────────────
ARG SDK_TAG=8.0
FROM mcr.microsoft.com/dotnet/sdk:${SDK_TAG} AS build-net
ENV BuildingDocker=true

WORKDIR /app
# The Dockerfile is in the same folder as the .csproj
COPY *.csproj ./
RUN dotnet restore

# Copy the rest and publish
COPY . .
RUN dotnet publish -c Release -o /twtweb

# ─────────────────────────────
# Stage 3: Runtime image
# ─────────────────────────────
ARG RUNTIME_TAG=8.0
FROM mcr.microsoft.com/dotnet/aspnet:${RUNTIME_TAG} AS runtime

# Make Kestrel listen on port 80 so your run command (-p 8080:80) works
ENV ASPNETCORE_URLS=http://+:80
WORKDIR /web
EXPOSE 80

# App + static frontend assets
COPY --from=build-net  /twtweb                 .
COPY --from=build-node /ClientApp/build        ./ClientApp/build

ENTRYPOINT ["dotnet", "Tailwind.Traders.Web.dll"]
