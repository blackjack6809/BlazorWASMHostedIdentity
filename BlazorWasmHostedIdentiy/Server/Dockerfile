#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["BlazorWasmHostedIdentiy/Server/BlazorWasmHostedIdentiy.Server.csproj", "BlazorWasmHostedIdentiy/Server/"]
COPY ["BlazorWasmHostedIdentiy/Shared/BlazorWasmHostedIdentiy.Shared.csproj", "BlazorWasmHostedIdentiy/Shared/"]
COPY ["BlazorWasmHostedIdentiy/Client/BlazorWasmHostedIdentiy.Client.csproj", "BlazorWasmHostedIdentiy/Client/"]
RUN dotnet restore "BlazorWasmHostedIdentiy/Server/BlazorWasmHostedIdentiy.Server.csproj"
COPY . .
WORKDIR "/src/BlazorWasmHostedIdentiy/Server"
RUN dotnet build "BlazorWasmHostedIdentiy.Server.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "BlazorWasmHostedIdentiy.Server.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "BlazorWasmHostedIdentiy.Server.dll"]