### Source Bashrc(sb) ⚡

sb是一个很好用的代码片段。按以下指南完成设置后，当你修改了环境变量，或者切换到不同的 Python 项目目录时，只需敲下 `sb` ，即可瞬间完成环境刷新。

# Ubuntu
## 🐧 SETUP (Linux / Ubuntu)
将以下内容添加到你的 `~/.bashrc` (或 `.zshrc`) 末尾，其后敲下最后一次的source ~/.bashrc
```bash
# --- Copy Paste this into your ~/.bashrc ---

check_python_env() {
    local curr_dir="$PWD"
    # 优先检测本地是否有 .venv 文件夹
    if [[ -f "$curr_dir/.venv/bin/activate" ]]; then
        echo -e "\033[32m[+] Detect .venv at $curr_dir, activating...\033[0m"
        source "$curr_dir/.venv/bin/activate"
        return
    # 2. 检测 Conda environment.yml (自动激活同名环境或按需处理)
    elif [[ -f "$curr_dir/environment.yml" ]]; then
        local env_name=$(grep "name:" "$curr_dir/environment.yml" | cut -d' ' -f2)
        if [ ! -z "$env_name" ]; then
            echo -e "\033[36m[+] Detect Conda environment.yml, activating [$env_name]...\033[0m"
            conda activate "$env_name" 2>/dev/null || echo -e "\033[31m[!] Conda activate failed.\033[0m"
        fi
        return
    fi
}

sb() {
# 防止死循环：设置一个临时环境变量标记
    if [[ -z "$SB_RELOADING" ]]; then
        export SB_RELOADING=1
        echo -e "\033[34m[System] Reloading config...\033[0m"
        
        # 重新加载配置
        if [ -f ~/.bashrc ]; then source ~/.bashrc; fi
        if [ -f ~/.zshrc ]; then source ~/.zshrc; fi
        
        # 清除标记
        unset SB_RELOADING
    fi

    # 检查并激活环境
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
    #...
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
    # 优先检测: .venv/Scripts/activate.ps1 (Windows常见)
    $curr = Get-Item .
    # --- 优先检测 .venv (原生虚拟环境) ---
    $venvPath = Join-Path $curr.FullName ".venv\Scripts\Activate.ps1"
    if (Test-Path $venvPath) {
        Write-Host " [Venv] Activating .venv at $($curr.FullName)..." -ForegroundColor Green
        & $venvPath
        $foundEnv = $true
        break
    }

    # --- 其次检测 Conda (environment.yml) ---
    $condaYaml = Join-Path $curr.FullName "environment.yml"
    if (Test-Path $condaYaml) {
        # 从 yaml 文件中解析 name 字段
        $envContent = Get-Content $condaYaml | Select-String "name:\s*(.*)"
        if ($envContent) {
            $envName = $envContent.Matches.Groups[1].Value.Trim()
            Write-Host " [Conda] Activating environment [$envName]..." -ForegroundColor Cyan
            conda activate $envName
            $foundEnv = $true
            break
        }
    }
    if (-not $foundEnv) {
        Write-Warning " [Warn] No virtual environment (.venv) found in current directory."
    }
}
```
