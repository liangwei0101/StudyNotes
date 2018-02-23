# 批量编译工具的编写
## 目的：因为开发和集成的需要，在不打开ide的情况下，批量编译相关的解决方案。
@echo off <br/>
if exist errorlog.txt del errorlog.txt<br/>
if exist successlog.txt del successlog.txt<br/>
path %SystemRoot%/system32;%SystemRoot%;<br/>
rem 是否有VS2015的相关编译链<br/>
if exist "%ProgramFiles(x86)%\MSBuild\14.0\Bin\MSBuild.exe" (<br/>
	rem has VS2015<br/>
	echo Use VS2015>>successlog.txt<br/>
	for /r %DIR% %%s in (*.sln) do (<br/>
		echo %%s>>temp.txt<br/>
		findstr "Subsys" temp.txt >nul &&( echo %%s is building) && ("%programfiles(x86)%\MSBuild\14.0\Bin\MSBuild.exe" %%s /t:Rebuild /m:3 /consoleloggerparameters:NoItemAndPropertyList;ErrorsOnly /flp1:logfile=errorlog.txt;errorsonly  /verbosity:m ) && ( if %errorlevel%==0 (echo %%s build success>>successlog.txt) else ( echo %%s build fail>>errorlog.txt))<br/>
		echo. >temp.txt<br/>
	) <br/>
) else (<br/>
	if exist "%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" (<br/>
		rem has VS2017 not VS2015<br/>
		echo Use VS2017>>successlog.txt<br/>
		for /r %DIR% %%s in (*.sln) do (<br/>
			echo %%s>>temp.txt<br/>
			findstr "Subsys" temp.txt >nul &&( echo %%s is building) && ("%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" %%s /t:Rebuild /m:3 /consoleloggerparameters:NoItemAndPropertyList;ErrorsOnly /flp1:logfile=errorlog.txt;errorsonly  /verbosity:m ) && ( if %errorlevel%==0 (echo %%s build success>>successlog.txt) else ( echo %%s build fail>>errorlog.txt))<br/>
			echo. >temp.txt<br/>
		)<br/>
	) else (<br/>
		rem not VS2017 not VS2017<br/>
		echo No VS2015 and No VS2017<br/>
	)<br/>
  )<br/>
)<br/>
echo -----------------------------------------successlog-----------------------------------------------------------<br/>
for /f "delims=," %%i in (successlog.txt) do (<br/>
	echo %%i<br/>
)<br/>
echo -----------------------------------------successlog-----------------------------------------------------------<br/>
echo -------------------------------------------errorlog-----------------------------------------------------------<br/>
for /f "delims=," %%i in (errorlog.txt) do (<br/>
	echo %%i<br/>
)<br/>
echo -------------------------------------------errorlog-----------------------------------------------------------<br/>
del temp.txt<br/>
pause<br/>