---
title: 美团店铺数据token解析与生成
tags:
  - python爬虫
  - 美团


date: 2019-4-11 18:36:28


categories:
  - python爬虫

---
**介绍：**要想解析token，首先得弄清楚其使用的加密算法，而对于美团的token，加密算法其实比较简单，就是采用了二进制压缩与base64编码。
<!--more-->

要想解析token，首先得弄清楚其使用的加密算法，而对于美团的token，加密算法其实比较简单，就是采用了二进制压缩与base64编码。所以解析token主要分为两个步骤：一是base64解码，二是二进制解压。解析token时主要用到了base64和zlib两个库，下面以实际例子来验证。

首先定义解析token方法：
```
def decode_token(token):
    # base64解码
    token_decode = base64.b64decode(token.encode())
    # 二进制解压
    token_string = zlib.decompress(token_decode)
    return token_string
```
然后获取多个美团的token，调用decode_token方法进行解析：
```
if __name__ == '__main__':
	token = [
        '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'
    ]

    for i in range(0, len(token)):
        token1 = decode_token(token[i])
        print(token1)
```
最后解析后得到了以下的结果：
```
b'{"rId":100900,"ver":"1.0.6","ts":1555228714393,"cts":1555228714429,"brVD":[1010,750],"brR":[[1920,1080],[1920,1040],24,24],"bI":["https://gz.meituan.com/meishi/c11/",""],"mT":[],"kT":[],"aT":[],"tT":[],"aM":"","sign":"eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=="}'
b'{"rId":100900,"ver":"1.0.6","ts":1555230010591,"cts":1555230010659,"brVD":[1010,750],"brR":[[1920,1080],[1920,1040],24,24],"bI":["https://gz.meituan.com/meishi/c11/",""],"mT":[],"kT":[],"aT":[],"tT":[],"aM":"","sign":"eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=="}'
b'{"rId":100900,"ver":"1.0.6","ts":1555230580338,"cts":1555230580399,"brVD":[1010,750],"brR":[[1920,1080],[1920,1040],24,24],"bI":["https://gz.meituan.com/meishi/c11/",""],"mT":[],"kT":[],"aT":[],"tT":[],"aM":"","sign":"eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=="}'
b'{"rId":100900,"ver":"1.0.6","ts":1555230116325,"cts":1555230116367,"brVD":[1010,750],"brR":[[1920,1080],[1920,1040],24,24],"bI":["https://gz.meituan.com/meishi/c11/",""],"mT":[],"kT":[],"aT":[],"tT":[],"aM":"","sign":"eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=="}'
```
三、生成token
在上一步解析token的结果中，可以看到token中包含有以下几个信息，而且除了ts和cts两个属性值外，其余值都是固定的。
```
"rId":100900,
"ver":"1.0.6",
"ts":1555228714393,
"cts":1555228714429,
"brVD":[1010,750],
"brR":[[1920,1080],[1920,1040],24,24],
"bI":["https://gz.meituan.com/meishi/c11/",""],
"mT":[],
"kT":[],
"aT":[],
"tT":[],
"aM":"",
"sign":"eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=="
```
ts和cts两个值其实就是时间戳，也就是token的有效时间，所以我们在生成token时主要修改这两个属性，其余值保持不变即可。但是要注意到这里的ts和cts是我们传统时间戳的1000倍，所以生成的时候千万不要忘记扩大1000倍。废话不多说，直接看代码吧。

### 生成token

```
def encode_token():
     ts = int(datetime.now().timestamp() * 1000)
     token_dict = {
        'rId': 100900,
        'ver': '1.0.6',
        'ts': ts,
        'cts': ts + 100 * 1000,
        'brVD': [1010, 750],
        'brR': [[1920, 1080], [1920, 1040], 24, 24],
        'bI': ['https://gz.meituan.com/meishi/c11/', ''],
        'mT': [],
        'kT': [],
        'aT': [],
        'tT': [],
        'aM': '',
        'sign': 'eJwdjktOwzAQhu/ShXeJ4zYNKpIXqKtKFTsOMLUn6Yj4ofG4UjkM10CsOE3vgWH36df/2gAjnLwdlAPBBsYoR3J/hYD28f3z+PpUnmJEPqYa5UWEm0mlLBRqOSaP1qjEtFB849VeRXJ51nr56AOSVIi9S0E3LlfSzhitMix/mQwsrdWa7aTyCjInDk1mKu9nvOHauCQWq2rB/8laqd3cX+adv0zdzm3nbjTOdzCi69A/HQAHOOyHafMLmEtKXg=='
    }
    # 二进制编码
    encode = str(token_dict).encode()
    # 二进制压缩
    compress = zlib.compress(encode)
    # base64编码
    b_encode = base64.b64encode(compress)
    # 转为字符串
    token = str(b_encode, encoding='utf-8')
    return token
```

生成token其实就是解析token的逆过程，所以要先进行二进制压缩，然后进行base64编码，最后转为字符串，下面就是生成了4个token的结果。
```
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
```