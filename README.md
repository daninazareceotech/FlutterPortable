
@echo off
setlocal EnableExtensions
chcp 65001 >nul
title FlutterPortable - Gerenciador
color 0B

set "ROOT=%~dp0"
set "FLUTTERDIR=%ROOT%flutter"
set "FLUTTER=%FLUTTERDIR%\bin\flutter.bat"
set "DART=%FLUTTERDIR%\bin\dart.bat"
set "PROJETOS=%ROOT%projetos"
set "CACHE=%ROOT%cache"
set "ANDROIDSDK=%ROOT%android-sdk"

if not exist "%PROJETOS%" mkdir "%PROJETOS%"
if not exist "%CACHE%" mkdir "%CACHE%"

if not exist "%FLUTTER%" (
    cls
    echo ERRO: Flutter SDK nao encontrado!
    echo.
    echo Caminho esperado:
    echo "%FLUTTER%"
    echo.
    echo Extraia o SDK do Flutter na pasta:
    echo "%FLUTTERDIR%"
    echo.
    pause
    exit /b 1
)

set "PATH=%FLUTTERDIR%\bin;%PATH%"
set "PUB_CACHE=%CACHE%"

if exist "%ANDROIDSDK%" (
    set "ANDROID_HOME=%ANDROIDSDK%"
    set "ANDROID_SDK_ROOT=%ANDROIDSDK%"
    set "PATH=%ANDROIDSDK%\platform-tools;%PATH%"
)

:MENU
cls
echo ==========================================
echo          FLUTTER PORTABLE
echo ==========================================
echo Pasta: %ROOT%
echo.
echo [1] Verificar Flutter e Dart
echo [2] Executar Flutter Doctor
echo [3] Listar dispositivos
echo [4] Abrir terminal Flutter
echo [5] Listar projetos
echo [6] Criar projeto Flutter
echo [7] Abrir pasta FlutterPortable
echo [8] Executar projeto Flutter
echo [9] Instalar dependencias
echo [10] Verificar projeto
echo [11] Atualizar dependencias
echo [12] Limpar projeto
echo [13] Compilar APK Android
echo [14] Executar no Chrome
echo [15] Executar no Windows
echo [16] Atualizar Flutter
echo [17] Aceitar licencas Android
echo [0] Sair
echo.
set "OP="
set /p "OP=Escolha: "

if "%OP%"=="1" goto VERSAO
if "%OP%"=="2" goto DOCTOR
if "%OP%"=="3" goto DISPOSITIVOS
if "%OP%"=="4" goto TERMINAL
if "%OP%"=="5" goto LISTAR
if "%OP%"=="6" goto CRIAR
if "%OP%"=="7" start "" "%ROOT%" & goto MENU
if "%OP%"=="8" goto EXECUTAR
if "%OP%"=="9" goto DEPENDENCIAS
if "%OP%"=="10" goto VERIFICAR
if "%OP%"=="11" goto ATUALIZAR_DEP
if "%OP%"=="12" goto LIMPAR
if "%OP%"=="13" goto APK
if "%OP%"=="14" goto CHROME
if "%OP%"=="15" goto WINDOWS
if "%OP%"=="16" goto ATUALIZAR_FLUTTER
if "%OP%"=="17" goto LICENCAS
if "%OP%"=="0" exit /b

goto MENU

:VERSAO
cls
echo === FLUTTER ===
call "%FLUTTER%" --version
echo.
echo === DART ===
call "%DART%" --version
echo.
pause
goto MENU

:DOCTOR
cls
call "%FLUTTER%" doctor -v
echo.
pause
goto MENU

:DISPOSITIVOS
cls
call "%FLUTTER%" devices
echo.
pause
goto MENU

:TERMINAL
start "Terminal FlutterPortable" /D "%PROJETOS%" cmd /k "set PATH=%FLUTTERDIR%\bin;%PATH%&& set PUB_CACHE=%CACHE%&& if exist "%ANDROIDSDK%" set ANDROID_HOME=%ANDROIDSDK%"
goto MENU

:LISTAR
cls
echo === PROJETOS FLUTTER ===
echo.
for /d %%D in ("%PROJETOS%\*") do (
    if exist "%%D\pubspec.yaml" echo %%~nxD
)
echo.
pause
goto MENU

:CRIAR
cls
echo === CRIAR PROJETO FLUTTER ===
echo.
set "NOME="
set /p "NOME=Nome do projeto: "

if not defined NOME goto MENU

if exist "%PROJETOS%\%NOME%" (
    echo Essa pasta ja existe!
    pause
    goto MENU
)

pushd "%PROJETOS%"
call "%FLUTTER%" create "%NOME%"
set "RESULTADO=%ERRORLEVEL%"
popd

if not "%RESULTADO%"=="0" (
    echo Falha ao criar o projeto.
    pause
    goto MENU
)

echo.
echo Projeto criado em:
echo %PROJETOS%\%NOME%
pause
goto MENU

:EXECUTAR
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" run
popd
pause
goto MENU

:DEPENDENCIAS
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" pub get
popd
pause
goto MENU

:VERIFICAR
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%DART%" analyze
set "RESULTADO=%ERRORLEVEL%"
popd

echo.
echo Codigo de saida: %RESULTADO%
pause
goto MENU

:ATUALIZAR_DEP
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" pub upgrade
popd
pause
goto MENU

:LIMPAR
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" clean
popd
pause
goto MENU

:APK
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" build apk
set "RESULTADO=%ERRORLEVEL%"
popd

if "%RESULTADO%"=="0" (
    echo.
    echo APK gerado em:
    echo %PROJETOS%\%NOME%\build\app\outputs\flutter-apk\
)
pause
goto MENU

:CHROME
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" run -d chrome
popd
pause
goto MENU

:WINDOWS
call :SELECIONAR
if not defined NOME goto MENU

pushd "%PROJETOS%\%NOME%"
call "%FLUTTER%" run -d windows
popd
pause
goto MENU

:ATUALIZAR_FLUTTER
cls
echo Atualizando o SDK Flutter...
call "%FLUTTER%" upgrade
pause
goto MENU

:LICENCAS
cls
call "%FLUTTER%" doctor --android-licenses
pause
goto MENU

:SELECIONAR
set "NOME="
echo.
echo Projetos disponiveis:
for /d %%D in ("%PROJETOS%\*") do (
    if exist "%%D\pubspec.yaml" echo   %%~nxD
)
echo.
set /p "NOME=Nome da pasta do projeto: "

if not defined NOME exit /b

if not exist "%PROJETOS%\%NOME%\pubspec.yaml" (
    echo Projeto Flutter nao encontrado!
    set "NOME="
    pause
)
exit /b
