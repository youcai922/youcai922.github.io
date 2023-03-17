```bat
echo off
echo 上班脚本
echo 第一部分-启动常用软件
Start "" "E:\work\software\ideaUI\bin\idea64.exe"
Start "" "E:\work\software\dbeaver\dbeaver.exe"
Start "" "C:\Program Files\Google\Chrome\Application\chrome.exe"
Start "" "E:\work\software\npp\Notepad++\notepad++.exe"
echo 常用软件启动成功
echo 第二部分-完成上班任务
msg * "检查钉钉打卡"
msg * "每日工作计划邮件发送"
echo 重置IDEA有效日期
start "" "E:\work\software\ideaUI\pojie\永久激活方式(请解压，资料在里面)\方式三（无限重置激活插件）\reset_script\reset_jetbrains_eval_windows.vbs"
pause
```



```
netstat -ano |find /i /c "CLOSE_WAIT"
```

