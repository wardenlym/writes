
下载 https://hyper.is/

打开 菜单——Edit——Preference 中修改

修改启动默认进入wsl
```
 # 编辑shell和shellArgs
 shell: 'C:\\Windows\\System32\\wsl.exe',
 shellArgs: ['~'],
```

安装插件，主要是为了透明
因为默认的windows下透明功能

打开系统管理员powershell
PS C:\Windows\system32> hyper i hyper-opacity
PS C:\Windows\system32> hyper i hyper-search

https://github.com/lucleray/hyper-
配置参考

设置前台后台不同透明度

module.exports = {
  config: {
    // rest of the config
    opacity: {
      focus: 0.8,
      blur: 0.5
    }
  }
  // rest of the file
}