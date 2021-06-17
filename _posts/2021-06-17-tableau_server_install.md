# 태블로 서버 설치 & 재설치시 TIP
- 태블로 완전삭제
  1. 아래 script를 tableau-server-obliterate.cmd 이름으로 저장
  2. tableau-server-obliterate.cmd -a -y -y -y 명령어로 실행
<pre>
<code>
@ECHO OFF
@SETLOCAL enableextensions enabledelayedexpansion

:refresh_environment_variables
CALL "%~dp0refresh-environment-variables.cmd"

REM script global variables
SET script_dir=%~dp0
IF %script_dir:~-1%==\ SET script_dir=%script_dir:~0,-1%
SET script_full_path=%0
SET script_filename=%~n0%~x0
SET tsm_services=tabadminagent_0 tabadmincontroller_0 tabsvc_0 appzookeeper_0 appzookeeper_1 licenseservice_0 clientfileservice_0
SET tsm_env_vars=TABLEAU_SERVER_CONFIG_NAME TABLEAU_SERVER_DATA_DIR TABLEAU_SERVER_DATA_DIR_VERSION TABLEAU_SERVER_INSTALL_DIR TSM_CLEAN_INSTALL_FAILURE
SET version_string=20182.18.0627.2230
SET license_removal_requested=0
SET yes=0
SET very_silent=0
SET bypass_script_dir_check=0
SET licensing_url="https://licensing.tableau.com/flexnet/services/ActivationService?wsdl"
SET serveractutil_exe=%TABLEAU_SERVER_INSTALL_DIR%\packages\bin.%version_string%\serveractutil.exe

:check_tab_root
IF "%TAB_ROOT%" NEQ "" (
  ECHO Invalid command prompt. Cancelling.
  EXIT /B 1
)

:check_admin
NET SESSION >NUL 2>&1
if %ERRORLEVEL% NEQ 0 (
  ECHO This script must be run as Administrator. Cancelling.
  EXIT /B 1
)

:parse_command_line_params
IF "%1"=="" GOTO end_parse
IF "%1"=="-l" SET license_removal_requested=1
IF "%1"=="/l" SET license_removal_requested=1
IF "%1"=="-h" GOTO show_help
IF "%1"=="/h" GOTO show_help
IF "%1"=="-y" SET /A yes=%yes%+1
IF "%1"=="/y" SET /A yes=%yes%+1
IF "%1"=="-q" SET very_silent=1
IF "%1"=="/q" SET very_silent=1
IF "%1"=="-b" SET bypass_script_dir_check=1
IF "%1"=="/b" SET bypass_script_dir_check=1

SHIFT
GOTO parse_command_line_params
:end_parse


:confirm
IF %yes% LSS 3 (
    ECHO You must specify the '-y' flag three times to confirm running the script is desired
    ECHO.
    GOTO show_help
)

:check_script_dir
IF %bypass_script_dir_check% EQU 0 (
  IF /I "%script_dir%" NEQ "%TEMP%" (
    COPY %script_full_path% %TEMP%\%script_filename% /Y >NUL 2>&1
    START cmd /c %TEMP%\%script_filename% %*
    EXIT /B 0
  )
)

:deactivate_licenses
IF %license_removal_requested% EQU 1 (
  IF EXIST "%serveractutil_exe%" (
    FOR /F "tokens=1,2 delims=:" %%F IN ('"%serveractutil_exe%" -view') DO (
	    ECHO.%%F | findstr /C:"Fulfillment ID">nul && (
		    ECHO Deactivating license fulfillment key: %%G
            "%serveractutil_exe%" -return %%G -comm soap -reason 1 -commServer "%licensing_url%"
		)
    )
  ) ELSE (
    ECHO Unable to find serveractutil.exe, skipping license deactivation.
  )
)

:remove_tsm_services
FOR %%s IN (%tsm_services%) DO (
  SC QUERY %%s >NUL 2>&1
  IF !ERRORLEVEL! EQU 0 (
    FOR /F "tokens=3" %%P IN ('SC QUERYEX %%s ^| FINDSTR PID') DO (SET pid=%%P)
	IF "!pid!" NEQ "0" (
		ECHO Service %%s is running with process ID: !pid!
		ECHO Stopping service %%s
		NET STOP %%s >NUL 2>&1
		TASKKILL /F /PID !pid! >NUL 2>&1
	)

    ECHO Deleting service %%s
    SC DELETE %%s >NUL 2>&1
  ) ELSE (
    ECHO Service %%s not found. Skipping.
  )
)

:uninstall_packages
FOR /F "tokens=*" %%K IN ('reg query HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall /f TableauServer /k 2^>NUL ^| FINDSTR /R /C:"TableauServer"') DO (
  FOR /F "tokens=2*" %%A IN ('REG Query "%%K" /F DisplayName /V /E ^| FIND /I " DisplayName "') DO ECHO Uninstalling %%B
  IF %very_silent% EQU 1 (
    FOR /F "tokens=2*" %%A IN ('REG Query "%%K" /F UninstallString /V /E ^| FIND /I " UninstallString "') DO START "" /B %%B /VERYSILENT /SUPPRESSMSGBOXES
  ) ELSE (
    FOR /F "tokens=2*" %%A IN ('REG Query "%%K" /F UninstallString /V /E ^| FIND /I " UninstallString "') DO START "" /B %%B /SILENT /SUPPRESSMSGBOXES
  )
)

:delete_data_dir
ECHO Deleting data directory %TABLEAU_SERVER_DATA_DIR%
RD /S /Q "%TABLEAU_SERVER_DATA_DIR%" 2>NUL

:remove_certs
certutil -delstore root TableauServerManagerCA

:remove_env_vars
ECHO Deleting Tableau Server environment variables
FOR %%v IN (%tsm_env_vars%) DO (
  reg delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v %%v /f >NUL 2>&1
)

ECHO Tableau Server obliterated
EXIT /B 0

:show_help
ECHO Usage: tableau-server-obliterate [-h] [-l] -y -y -y
ECHO Completely remove Tableau Server from this machine
ECHO.
ECHO This script will stop/kill any running Tableau Server proceses and remove all Tableau Server
ECHO related files. No data or configuration is retained, except files related to licensing.
ECHO This script should only be used to clean a machine because it is destructive to all data.
ECHO.
ECHO This script must be run as the Administrator.
ECHO.
ECHO   -y                       Yes, perform this action. Must be specified three times to
ECHO                            confirm the action is desired.
ECHO   -q                       Quiet mode. Do not display progress UI when uninstalling
ECHO                            Tableau Server packages.
ECHO   -l                       Also delete licensing files and data. This command will
ECHO                            attempt to deactivate licenses before deleting licensing
ECHO                            data, but cannot handle offline deactivation of licenses.
ECHO                            If in doubt, please use 'tsm licenses deactivate' before
ECHO                            running this script.


</pre>
</code>

- 삭제 후 재설치, 토폴로지 설정 시간이 오래걸린다. 꾹 참고 기다리자...
