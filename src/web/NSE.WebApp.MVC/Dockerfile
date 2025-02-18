# Acesse https://aka.ms/customizecontainer para saber como personalizar seu contêiner de depuração e como o Visual Studio usa este Dockerfile para criar suas imagens para uma depuração mais rápida.

#essas fases são o chamado Multi Stage Build, 4 imagens, cada uma para um Stage

# Esta fase é usada durante a execução no VS no modo rápido (Padrão para a configuração de Depuração) - Base
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base #De qual imagem a imagem que estou construindo vai derivar (imagem baseada em Linux que tem o runtime do asp.net8.0)
USER $APP_UID
WORKDIR /app #diretório de onde vc vai trabalhar - pasta app dentro da parte de escrita da imagem
EXPOSE 8080 #portas que minha máquina consegue "conversar" com o container
EXPOSE 8081


# Esta fase é usada para compilar o projeto de serviço - Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build #(imagem linux que contem o sdk8.0)
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["src/web/NSE.WebApp.MVC/NSE.WebApp.MVC.csproj", "src/web/NSE.WebApp.MVC/"]#copiando o csproj da máquina local pra pasta da imagem
COPY ["src/building blocks/NSE.Core/NSE.Core.csproj", "src/building blocks/NSE.Core/"]
COPY ["src/building blocks/NSE.WebAPI.Core/NSE.WebAPI.Core.csproj", "src/building blocks/NSE.WebAPI.Core/"]
RUN dotnet restore "./src/web/NSE.WebApp.MVC/NSE.WebApp.MVC.csproj" # restore de todos os pacotes necessário pra poder compilar a app
COPY . . #copiar tudo de tudo por workdir
WORKDIR "/src/src/web/NSE.WebApp.MVC" #setando o workdir pra dentro da pasta do projeto
RUN dotnet build "./NSE.WebApp.MVC.csproj" -c $BUILD_CONFIGURATION -o /app/build #vai rodar o comando dotnet build, o parâmetro BUILD_CONFIGURATION ele vai falar que é uma versão como se fosse pra rodar em produção e o -o de output vai apontar pra /app/build(ou criar) onde vai jogar somente os binários da compilação

# Esta fase é usada para publicar o projeto de serviço a ser copiado para a fase final
FROM build AS publish #(imagem com base na imagem build)
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./NSE.WebApp.MVC.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false #vai rodar o comando dotnet publish, com o parâmetro BUILD_CONFIGURATION e -o apontando para /app/publish

# Esta fase é usada na produção ou quando executada no VS no modo normal (padrão quando não está usando a configuração de Depuração)
FROM base AS final #(imagem com base na imagem base)
WORKDIR /app
COPY --from=publish /app/publish . #copiar tudo do container publish para minha pasta do workdir app/publish
ENTRYPOINT ["dotnet", "NSE.WebApp.MVC.dll"] #chamar o comando dotnet e falar pra ele rodar essa dll