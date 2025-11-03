# =========================
# Stage 1: Build the SPA (ClientApp) with Node (no apt needed)
# =========================
FROM node:18-alpine AS build-node
WORKDIR /clientapp

# Install only what package-lock.json specifies for reproducibility
COPY ClientApp/package*.json ./
RUN npm ci

# Copy the rest and build
COPY ClientApp/ .
RUN npm run build

# =========================
# Stage 2: Restore/Publish the ASP.NET Core app (.NET 5)
# (No apt usage here)
# =========================
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-net
WORKDIR /src

# Copy csproj first to leverage Docker layer caching
COPY Tailwind.Traders.Web.csproj ./
RUN dotnet restore

# Copy the rest and publish
COPY . .
RUN dotnet publish -c Release -o /out

# =========================
# Stage 3: Runtime (ASP.NET Core .NET 5)
# =========================
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
WORKDIR /app

# Copy published app
COPY --from=build-net /out ./

# Copy the built SPA static assets
# (Adjust the destination to where your app serves static files from, if different)
COPY --from=build-node /clientapp/build ./ClientApp/build

# Expose and run
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
ENTRYPOINT ["dotnet", "Tailwind.Traders.Web.dll"]
