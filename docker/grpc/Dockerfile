# Windows イメージを使用する
FROM mcr.microsoft.com/windows/servercore:1809

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# https://github.com/docker-library/python/pull/557
ENV PYTHONIOENCODING UTF-8

ENV PYTHON_VERSION 3.12.3

RUN $url = ('https://www.python.org/ftp/python/{0}/python-{1}-amd64.exe' -f ($env:PYTHON_VERSION -replace '[a-z]+[0-9]*$', ''), $env:PYTHON_VERSION); \
    Write-Host ('Downloading {0} ...' -f $url); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $url -OutFile 'python.exe'; \
    \
    Write-Host 'Installing ...'; \
    # https://docs.python.org/3/using/windows.html#installing-without-ui
    $exitCode = (Start-Process python.exe -Wait -NoNewWindow -PassThru \
    -ArgumentList @( \
    '/quiet', \
    'InstallAllUsers=1', \
    'TargetDir=C:\Python', \
    'PrependPath=1', \
    'Shortcuts=0', \
    'Include_doc=0', \
    'Include_pip=0', \
    'Include_test=0' \
    ) \
    ).ExitCode; \
    if ($exitCode -ne 0) { \
    Write-Host ('Running python installer failed with exit code: {0}' -f $exitCode); \
    Get-ChildItem $env:TEMP | Sort-Object -Descending -Property LastWriteTime | Select-Object -First 1 | Get-Content; \
    exit $exitCode; \
    } \
    \
    # the installer updated PATH, so we should refresh our local value
    $env:PATH = [Environment]::GetEnvironmentVariable('PATH', [EnvironmentVariableTarget]::Machine); \
    \
    Write-Host 'Verifying install ...'; \
    Write-Host '  python --version'; python --version; \
    \
    Write-Host 'Removing ...'; \
    Remove-Item python.exe -Force; \
    Remove-Item $env:TEMP/Python*.log -Force; \
    \
    Write-Host 'Complete.'

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 24.0
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/dbf0c85f76fb6e1ab42aa672ffca6f0a675d9ee4/public/get-pip.py
ENV PYTHON_GET_PIP_SHA256 dfe9fd5c28dc98b5ac17979a953ea550cec37ae1b47a5116007395bfacff2ab9

RUN Write-Host ('Downloading get-pip.py ({0}) ...' -f $env:PYTHON_GET_PIP_URL); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $env:PYTHON_GET_PIP_URL -OutFile 'get-pip.py'; \
    Write-Host ('Verifying sha256 ({0}) ...' -f $env:PYTHON_GET_PIP_SHA256); \
    if ((Get-FileHash 'get-pip.py' -Algorithm sha256).Hash -ne $env:PYTHON_GET_PIP_SHA256) { \
    Write-Host 'FAILED!'; \
    exit 1; \
    }; \
    \
    $env:PYTHONDONTWRITEBYTECODE = '1'; \
    \
    Write-Host ('Installing pip=={0} ...' -f $env:PYTHON_PIP_VERSION); \
    python get-pip.py \
    --disable-pip-version-check \
    --no-cache-dir \
    --no-compile \
    ('pip=={0}' -f $env:PYTHON_PIP_VERSION) \
    ; \
    Remove-Item get-pip.py -Force; \
    \
    Write-Host 'Verifying pip install ...'; \
    pip --version; \
    \
    Write-Host 'Complete.'

WORKDIR /protoc
RUN Set-ExecutionPolicy Bypass -Scope Process -Force; \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest https://github.com/protocolbuffers/protobuf/releases/download/v27.0-rc1/protoc-27.0-rc-1-win64.zip -OutFile ./out.zip

RUN Expand-Archive ./out.zip .
#RUN $path = C:\protoc\bin
#RUN $Env:PATH += ';' + $path

# install MinGit (especially for "go get")
# https://blogs.msdn.microsoft.com/visualstudioalm/2016/09/03/whats-new-in-git-for-windows-2-10/
# "Essentially, it is a Git for Windows that was stripped down as much as possible without sacrificing the functionality in which 3rd-party software may be interested."
# "It currently requires only ~45MB on disk."
ENV GIT_VERSION 2.23.0
ENV GIT_TAG v${GIT_VERSION}.windows.1
ENV GIT_DOWNLOAD_URL https://github.com/git-for-windows/git/releases/download/${GIT_TAG}/MinGit-${GIT_VERSION}-64-bit.zip
ENV GIT_DOWNLOAD_SHA256 8f65208f92c0b4c3ae4c0cf02d4b5f6791d539cd1a07b2df62b7116467724735
# steps inspired by "chcolateyInstall.ps1" from "git.install" (https://chocolatey.org/packages/git.install)
RUN Write-Host ('Downloading {0} ...' -f $env:GIT_DOWNLOAD_URL); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $env:GIT_DOWNLOAD_URL -OutFile 'git.zip'; \
    \
    Write-Host ('Verifying sha256 ({0}) ...' -f $env:GIT_DOWNLOAD_SHA256); \
    if ((Get-FileHash git.zip -Algorithm sha256).Hash -ne $env:GIT_DOWNLOAD_SHA256) { \
    Write-Host 'FAILED!'; \
    exit 1; \
    }; \
    \
    Write-Host 'Expanding ...'; \
    Expand-Archive -Path git.zip -DestinationPath C:\git\.; \
    \
    Write-Host 'Removing ...'; \
    Remove-Item git.zip -Force; \
    \
    Write-Host 'Updating PATH ...'; \
    $env:PATH = 'C:\git\cmd;C:\git\mingw64\bin;C:\git\usr\bin;' + $env:PATH; \
    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine); \
    \
    Write-Host 'Verifying install ("git version") ...'; \
    git version; \
    \
    Write-Host 'Complete.';

SHELL ["cmd", "/S", "/C"]

ENV GOPATH C:\\go

USER ContainerAdministrator
RUN setx /m PATH "%GOPATH%\bin;C:\Program Files\Go\bin;%PATH%"
USER ContainerUser
# doing this first to share cache across versions more aggressively

ENV GOLANG_VERSION 1.22.2
# Docker's Windows path parsing is absolutely *cursed*; please just trust me on this one -Tianon
COPY --from=golang:1.22.2-windowsservercore-1809 ["C:\\\\Program Files\\\\Go","C:\\\\Program Files\\\\Go"]
RUN go version

WORKDIR $GOPATH



EXPOSE 50051