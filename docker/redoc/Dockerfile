# Windows イメージを使用する
FROM mcr.microsoft.com/dotnet/framework/sdk:4.8

# Chocolatey のインストール（パッケージ管理ツール）
RUN powershell -Command \
    Set-ExecutionPolicy Bypass -Scope Process -Force; \
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; \
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# Node.js のインストール
RUN choco install -y nodejs
RUN npm install -g http-server
RUN npm install -g npx
# 必要に応じて、その他の依存パッケージをインストールすることもできます

EXPOSE 8080

ENTRYPOINT ["powershell.exe"]
# CMD ["http-server"]