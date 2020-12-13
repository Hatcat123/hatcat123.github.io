从网络上下载文件，尤其是非常大的文件怎么确保文件准确无误呢？

通常网站提供文件时会同时提供该文件的校验值，如MD5，SHA1，SHA256等，

当文件下载完成后，计算它的校验值，如果和网站提供的一致，就可以放心使用了。

Windows 使用命令行计算校验值

在命令行下，可以使用Windows自带的certutil命令来计算一个文件的校验值：

```
certutil.exe -hashfile
要求至少 1 个参数，但收到了 0 个
CertUtil: 找不到参数

用法:
  CertUtil [选项] -hashfile InFile [HashAlgorithm]
  通过文件生成并显示加密哈希

选项:
  -Unicode          -- 以 Unicode 编写重定向输出
  -gmt              -- 将时间显示为 GMT
  -seconds          -- 用秒和毫秒显示时间
  -v                -- 详细操作
  -privatekey       -- 显示密码和私钥数据
  -pin PIN                  -- 智能卡 PIN
  -sid WELL_KNOWN_SID_TYPE  -- 数字 SID
            22 -- 本地系统
            23 -- 本地服务
            24 -- 网络服务

哈希算法: MD2 MD4 MD5 SHA1 SHA256 SHA384 SHA512

CertUtil -?              -- 显示动词列表(命名列表)
CertUtil -hashfile -?    -- 显示 "hashfile" 动词的帮助文本
CertUtil -v -?           -- 显示所有动词的所有帮助文本
```
certutil的使用方法非常简单，只需要执行“certutil -hashfile 文件名 校验值类型”，即可计算出对应文件的校验值。例如：计算D:\works\hello.txt这个文件的MD5，可以执行命令：
```
certutil -hashfile D:\works\Hello.txt MD5
```


Linux 使用命令行计算校验值
Linux下可以直接使用md5sum/sha1sum/sha256sum等命令直接计算文件的对应校验值。
```
md5sum /works/Hello.txt 
sha1sum /works/Hello.txt 
sha256sum /works/Hello.txt 
```

关于校验值

校验值是一组16进制数，不区分大小写，校验值本身只与文件内容有关，只要文件内容不改变校验值就不变；如复制/剪切/粘贴，修改文件创建时间/访问时间，修改文件读/写/执行属性等操作都不会导致校验值发生改变。

当掌握快速计算校验值方法后，以后发送文件时就可以附带上该文件的校验值以防止文件中途损坏或被他人无意间修改。





