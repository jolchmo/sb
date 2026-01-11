### Source Bashrc(sb) ⚡

sb是一个很好用的代码片段。按以下指南完成设置后，当你修改了环境变量，或者切换到不同的 Python 项目目录时，只需敲下 `sb` ，即可瞬间完成环境刷新。

# Ubuntu
## 🐧 SETUP (Linux / Ubuntu)
将以下内容添加到你的 `~/.bashrc` (或 `.zshrc`) 末尾，其后敲下最后一次的source ~/.bashrc
```bash
# --- Copy Paste this into your ~/.bashrc ---

check_python_env() {
    # 优先检测本地是否有 .venv 文件夹
    if [[ -f "./.venv/bin/activate" ]]; then
        echo -e "\033[32m[+] Detect .venv, activating...\033[0m"
        source ./.venv/bin/activate
    # 如果当前路径下有./.venv/bin/activate，那么自动激活python虚拟环境
    elif  [[ -f "./.venv/bin/activate" ]]; then
        source ./.venv/bin/activate
    fi
}

sb() {
    # 1. 重新读取 bash 配置 (刷新环境变量/别名)
    source ~/.bashrc
    
    # 2. 检查并按需自动激活当前目录的 Python 虚拟环境
    check_python_env
}
```

## 🪟 SETUP (Windows PowerShell)
使用POWERSHELL,输入$profile,找到如C:\Users\<USER>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1的路径。
```
# ------------- Path 定义区域: 这是你的“标准已知好”及全部所需路径 -------------
# 设置所需的环境路径,可以先通过 echo $env:path查看
$StdPathLocations = @(
    "C:\Program Files\PowerShell\7",
    "C:\Windows\System32\WindowsPowerShell\v1.0\",
    # ---- 核心系统 (System Core) ----
    "C:\WINDOWS\system32",
    "C:\WINDOWS",
    "C:\WINDOWS\System32\Wbem",
    "C:\WINDOWS\System32\OpenSSH",
    # ---- Build Tools & Compilers (VS, CMake, Qt) ----
    "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.42.34433\bin\Hostx64\x64",
    "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin",
    "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\Ninja",
    "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\",
    "C:\Qt\6.7.3\msvc2022_64\bin",
    # ---- Common Platforms (Node, Py, Dotnet) ----
    "C:\Program Files\nodejs\",
    "C:\Program Files\dotnet\",
    "C:\Users\<USER>\AppData\Local\Programs\Python\Python311",
    "C:\Users\<USER>\AppData\Local\Programs\Python\Python311\Scripts",
    "C:\Users\<USER>\AppData\Local\Programs\Python\Python38",
    "C:\Users\<USER>\AppData\Local\Programs\Python\Python38\Scripts",
    "C:\Program Files\Eclipse Adoptium\jdk-17.0.16.8-hotspot\bin",
    "C:\Program Files\Microsoft\jdk-21.0.8.9-hotspot\bin",
)

$SetUserEnvVars = {
    Write-Host " [Env] Setting global variables..." -ForegroundColor DarkGray
    # --- General ---
    $env:EDITOR = "code"            
    $env:LANG   = "en_US.UTF-8"
    # --- Java Options (Example) ---
    # 下方路径务必确保和 $StdPathLocations 里的 JDK 版本对应
    $env:JAVA_HOME = "C:\Program Files\Microsoft\jdk-21.0.8.9-hotspot\"
    # --- Aliases (常用别名) ---
    if (Get-Alias grep -ErrorAction SilentlyContinue) { Remove-Item alias:\grep }
    Set-Alias grep findstr
    Set-Alias ll ls 
    Set-Alias n vim
    Set-Alias which Get-Command
    Set-Alias time Get-Date

}
# 定义函数别名
function sb {
    Write-Host " [System] Reloading Profile..." -ForegroundColor Cyan

    Write-Host "`n====== REFRESHING ENVIRONMENT ======" -ForegroundColor DarkCyan
    # 1. 执行变量重设 Logic
    # 按需解开下方的注释
    #& $SetUserEnvVars

    Write-Host " [PATH] Environment Variables Cleaned & Reset." -ForegroundColor DarkCyan
    # 2. 强制设定 System PATH (这就相当于硬生生 source 了一遍干净的 path 变量)
    # 按需解开下方的注释
    #$env:Path = $StdPathLocations -join ";"


    # 3. 加载虚拟环境(如有)
    $foundEnv = $false
    # 检测: .venv/bin/activate.ps1 (Unix/Legacy)
    if (Test-Path ".\.venv\bin\Activate.ps1") {
        Write-Host " [Venv] Activating (Bin mode)..." -ForegroundColor Green
        & ".\.venv\bin\Activate.ps1"
        $foundEnv = $true
    }
    if (-not $foundEnv) {
        Write-Warning " [Warn] No virtual environment (.venv) found in current directory."
    }
}
```
