#### powershell获取环境变量

```powershell
[Environment]::GetEnvironmentVariable('ENV_NAME', 'Target')
```

- ENV_NAME，环境变量名称;
- Target，全局变量还是用户变量，从'Machine'和'User'二选一。

#### powershell永久增加和修改环境变量

```powershell
[Environment]::SetEnvironmentVariable('ENV_NAME', 'ENV_VALUE', 'Target')
```

- ENV_NAME, 环境变量名称;
- ENV_VALUE，环境变量值；
- Target，全局变量还是用户变量，从'Machine'和'User'二选一。

修改环境变量相同。

#### powershellc永久删除环境变量

```powershell
[Environment]::SetEnvironmentVariable('ENV_NAME', $null, 'Target')
```

- ENV_NAME, 环境变量名称;
- Target，全局变量还是用户变量，从'Machine'和'User'二选一。

