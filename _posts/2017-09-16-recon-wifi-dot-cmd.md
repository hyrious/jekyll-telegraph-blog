---
title: recon_wifi.cmd
---

学校换了个不稳定的 WIFI，每隔 20~30 分钟主动回收一次 IP（然后网就掉了）。

于是我写了个小批处理来自动重连。

局限性：仅能重连某个固定 wifi，且连接不需要密码的。

用法：

1. `netsh wlan show profiles` 查看你要重连的 WIFI 的名字，填入 `PROFILE`
2. `TESTURL` 可改为 baidu.com 等，用 ping 的方式判断是否有网
3. `TIMEOUT` 每次 ping 后等待的时间（秒）

*recon_wifi.cmd*

```cmd
@echo off
setlocal

set TESTURL=a.suda.edu.cn
set PROFILE=SUDA_WIFI
set TIMEOUT=10

echo Reconnect wifi when it was down.
echo.
echo [Options]
echo   TESTURL^(^=%TESTURL%^): ping it to check the connection
echo   PROFILE^(^=%PROFILE%^): netsh wlan show profiles
echo   TIMEOUT^(^=%TIMEOUT%^): sleep such time after every ping
echo.
echo Control-C,Y to stop.
echo.

:mainloop
>nul ping %TESTURL% -n 1 -w 1000
if errorlevel 1 goto reconnect
>nul timeout /t %TIMEOUT%
goto mainloop
:reconnect
echo Reconnecting at %date% %time%
netsh wlan disconnect
netsh wlan connect name=%PROFILE%
>nul timeout /t %TIMEOUT%
goto mainloop
```
