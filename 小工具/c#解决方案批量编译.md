# 批量编译工具的编写
## 目的：因为开发和集成的需要，在不打开ide的情况下，批量编译相关的解决方案。
@echo off
if exist errorlog.txt del errorlog.txt
if exist successlog.txt del successlog.txt
path %SystemRoot%/system32;%SystemRoot%;
rem 是否有VS2015的相关编译链
if exist "%ProgramFiles(x86)%\MSBuild\14.0\Bin\MSBuild.exe" (
	rem has VS2015
	echo Use VS2015>>successlog.txt
	for /r %DIR% %%s in (*.sln) do (
		echo %%s>>temp.txt
		findstr "Subsys" temp.txt >nul &&( echo %%s is building) && ("%programfiles(x86)%\MSBuild\14.0\Bin\MSBuild.exe" %%s /t:Rebuild /m:3 /consoleloggerparameters:NoItemAndPropertyList;ErrorsOnly /flp1:logfile=errorlog.txt;errorsonly  /verbosity:m ) && ( if %errorlevel%==0 (echo %%s build success>>successlog.txt) else ( echo %%s build fail>>errorlog.txt))
		echo. >temp.txt
	) 
) else (
	if exist "%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" (
		rem has VS2017 not VS2015
		echo Use VS2017>>successlog.txt
		for /r %DIR% %%s in (*.sln) do (
			echo %%s>>temp.txt
			findstr "Subsys" temp.txt >nul &&( echo %%s is building) && ("%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" %%s /t:Rebuild /m:3 /consoleloggerparameters:NoItemAndPropertyList;ErrorsOnly /flp1:logfile=errorlog.txt;errorsonly  /verbosity:m ) && ( if %errorlevel%==0 (echo %%s build success>>successlog.txt) else ( echo %%s build fail>>errorlog.txt))
			echo. >temp.txt
		)
	) else (
		rem not VS2017 not VS2017
		echo No VS2015 and No VS2017
	)
  )
)
echo -----------------------------------------successlog-----------------------------------------------------------
for /f "delims=," %%i in (successlog.txt) do (
	echo %%i
)
echo -----------------------------------------successlog-----------------------------------------------------------
echo -------------------------------------------errorlog-----------------------------------------------------------
for /f "delims=," %%i in (errorlog.txt) do (
	echo %%i
)
echo -------------------------------------------errorlog-----------------------------------------------------------
del temp.txt
pause