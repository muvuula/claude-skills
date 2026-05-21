---
name: cleanse
description: 一键彻底卸载指定应用，自动查杀进程/服务/目录/快捷方式/AppData残留
args: 应用名（如 Marvis、WeChat、TencentMeeting）
---

# Cleanse

## 注入参数
应用名：`{{args}}`

## 步骤

1. 查杀相关系统服务：
   ```powershell
   Get-Service *{{args}}* -ErrorAction SilentlyContinue | Stop-Service -Force
   ```

2. 查杀所有相关进程：
   ```powershell
   Get-Process *{{args}}* -ErrorAction SilentlyContinue | Stop-Process -Force
   ```

3. 删除 Program Files 安装目录：
   ```powershell
   Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' -Directory -ErrorAction SilentlyContinue | Where-Object { $_.Name -like "*{{args}}*" } | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
   ```

4. 删除桌面和开始菜单快捷方式：
   ```powershell
   Get-ChildItem "$env:USERPROFILE\Desktop", "$env:APPDATA\Microsoft\Windows\Start Menu" -Filter "*{{args}}*.lnk" -ErrorAction SilentlyContinue | Remove-Item -Force
   ```

5. 清理 AppData 残留：
   ```powershell
   Get-ChildItem "$env:LOCALAPPDATA", "$env:APPDATA" -Directory -ErrorAction SilentlyContinue | Where-Object { $_.Name -like "*{{args}}*" } | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
   ```

6. 如遇文件被锁，重启资源管理器后重试卸载：

   > 某些 DLL 被 explorer.exe 加载（如任务栏扩展），需先重启 shell。
   ```powershell
   taskkill /f /im explorer.exe; timeout /t 2
   ```
   重复步骤 3-5，然后：
   ```powershell
   Start-Process explorer.exe
   ```

7. 验证清理结果：
   ```powershell
   Write-Host "=== 检查残留 ==="
   Get-Service *{{args}}* -ErrorAction SilentlyContinue | ForEach-Object { Write-Warning "服务残留: $($_.Name)" }
   Get-Process *{{args}}* -ErrorAction SilentlyContinue | ForEach-Object { Write-Warning "进程残留: $($_.ProcessName)" }
   Get-ChildItem "C:\Program Files", "C:\Program Files (x86)" -Directory -ErrorAction SilentlyContinue | Where-Object { $_.Name -like "*{{args}}*" } | ForEach-Object { Write-Warning "目录残留: $($_.FullName)" }
   Get-ChildItem "$env:USERPROFILE\Desktop" -Filter "*{{args}}*.lnk" -ErrorAction SilentlyContinue | ForEach-Object { Write-Warning "快捷方式残留: $($_.FullName)" }
   Write-Host "=== 检查完成 ==="
   ```
