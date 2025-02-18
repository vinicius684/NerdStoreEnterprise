# Acesse https://aka.ms/customizecontainer para saber como personalizar seu contêiner de depuração e como o Visual Studio usa este Dockerfile para criar suas imagens para uma depuração mais rápida.

#essas fases são o chamado Multi Stage Build, 4 imagens, cada uma para um Stage

# Esta fase é usada durante a execução no VS no modo rápido (Padrão para a configuração de Depuração) - Base
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base 
USER $APP_UID
WORKDIR /app 
EXPOSE 8080 
EXPOSE 8081


# Esta fase é usada para compilar o projeto de serviço - Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build 
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["./web/NSE.WebApp.MVC/NSE.WebApp.MVC.csproj", "web/NSE.WebApp.MVC/"]
COPY ["./building blocks/NSE.Core/NSE.Core.csproj", "building blocks/NSE.Core/"]
COPY ["./building blocks/NSE.WebAPI.Core/NSE.WebAPI.Core.csproj", "building blocks/NSE.WebAPI.Core/"]
RUN dotnet restore "web/NSE.WebApp.MVC/NSE.WebApp.MVC.csproj" 
COPY . . 
WORKDIR "web/NSE.WebApp.MVC" 
RUN dotnet build "NSE.WebApp.MVC.csproj" -c $BUILD_CONFIGURATION -o /app/build 

# Esta fase é usada para publicar o projeto de serviço a ser copiado para a fase final
FROM build AS publish 
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "NSE.WebApp.MVC.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false 

# Esta fase é usada na produção ou quando executada no VS no modo normal (padrão quando não está usando a configuração de Depuração)
FROM base AS final 
WORKDIR /app
COPY --from=publish /app/publish . 
ENTRYPOINT ["dotnet", "NSE.WebApp.MVC.dll"] 