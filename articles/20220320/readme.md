# Windows 下 fnm 的安装与配置

1. 从 [github](https://github.com/Schniz/fnm/releases) 下载最新的 exe 文件，并放置在文件夹中（例：D:\fnm）

2. 配置环境变量：计算机 - 系统属性  -高级系统设置 - 环境变量 - Path - 添加 - D:\fnm
   
3. 更改 PowerShell 执行策略：

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
   ```

4. 在 PowerShell 的 `$PROFILE` 中添加:

   ```powershell
   fnm env --use-on-cd | Out-String | Invoke-Expression
   ```

## 与 husky 配合使用

需要添加 `.huskyrc` 以便 husky 能够使用 fnm 管理的 node。

C:\Users\\{user}\\.huskyrc

```bash
eval "$(fnm env --use-on-cd)"

export PATH=$(cygpath $FNM_MULTISHELL_PATH):$PATH

if [[ -f .node-version || -f .nvmrc ]]; then
   fnm use
fi
```
