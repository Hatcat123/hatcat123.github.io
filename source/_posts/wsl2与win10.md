
### 关于使用WSL2出现“参考的对象类型不支持尝试的操作”的解决方法。

产生原因和解决方法分析：

代理软件和wsl2的sock端口冲突，使用netsh winsock reset重置修复。
Proxifer开发人员解释如下：
如果Winsock LSP DLL被加载到其进程中，则wsl.exe将显示此错误。最简单的解决方案是对wsl.exe使用WSCSetApplicationCategory WinAPI调用来防止这种情况。在后台，该调用在HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters\AppId_Catalog中为wsl.exe创建一个条目。
这将告诉Windows不要将LSP DLL加载到wsl.exe进程中
```
C:\WINDOWS\system32>NoLsp.exe C:\windows\system32\wsl.exe
Success!
```

https://github.com/microsoft/WSL/issues/4177



### Windows 局域网内其他机器访问 WSL2
[[WSL 2] NIC Bridge mode 🖧 (Has Workaround🔨) #4150](https://github.com/microsoft/WSL/issues/4150)
上面提到过的这个 issue 里其实就是解决的局域网访问的问题，需要的端口通过 Windows 代理转发到 WSL 中。

原理也类似于上面的两幅图。

如果你用了主机访问 WSL2 里的那个脚本，就可以跳过这一节了，因为那个脚本包括了这一节的内容了。

关键代码如下：

不懂的请看注释。
```
# 获取 Windows 和 WSL2 的 ip
$winip = bash.exe -c "ip route | grep default | awk '{print \`$3}'"
$wslip = bash.exe -c "hostname -I | awk '{print \`$1}'"
$found1 = $winip -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';
$found2 = $wslip -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( !($found1 -and $found2) ){
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

# 你需要映射到局域网中端口
$ports=@(80,443,10000,3000,5000,27701,8080);

# 监听的 ip，这么写是可以来自局域网
$addr='0.0.0.0';
# 监听的端口，就是谁来访问自己
$ports_a = $ports -join ",";

# 移除旧的防火墙规则
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' " | Out-Null

# 允许防火墙规则通过这些端口
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP"  | Out-Null
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP"  | Out-Null

# 使用 portproxy 让 Windows 转发端口
for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr"  | Out-Null
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$wslip"  | Out-Null
}
```
powershell 中 @() 就是声明数组的意思，这个脚本遍历你设置的想暴露到局域网的端口的数组，先关闭相应的防火墙策略，然后设置 portproxy 反代 Windows 的端口到 WSL 中。


 
### wsl中得docker问题
redis映射的端口连不上

```
PS C:\Users\YQ5> netstat -ano|findstr "6379"
  TCP    0.0.0.0:6379           0.0.0.0:0              LISTENING       11060
  TCP    0.0.0.0:6379           0.0.0.0:0              LISTENING       1136
  TCP    [::]:6379              [::]:0                 LISTENING       1136
  TCP    [::1]:6379             [::]:0                 LISTENING       15236
```
有两个程序都在绑定6379的端口

其中1136的进程名称叫`com.docker.backend.exe`，与docker有关
而11060的进程名称叫`svchost.exe`是一个windwos的进程，尝试杀掉这个进程redis就能链接上了


### 参考链接

https://lengthmin.me/posts/wsl2-network-tricks/