---
title: UESTC 校园网自动登录脚本
published: 2025-10-21 11:58:11
description: 实验室服务器无 GUI，所以需要通过脚本登录校园网
tags: [逆向]
category: 疑难杂症
draft: false
---

[电子科技大学校园网登录网站](http://10.253.0.237/srun_portal_pc?ac_id=1&theme=dx)

来看看第一个请求，请求的网址是`http://10.253.0.237/cgi-bin/get_challenge`，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251021042434-1761020674262.png"/>

其中的载荷，
- callback：jQuery 回调函数标识，用于处理异步请求的返回结果，实现前端对响应的回调处理。
- username：用户账号，格式为[学号]@dx-uestc。
- ip：用户的 IP 地址x.x.x.x，用于网络身份识别或定位。
- 最后的-参数：是时间戳（1761019519604），用于保证请求的唯一性，防止重复请求或缓存问题。
 
<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251021042555-1761020755681.png"/>


再来看看第二个请求，请求的网址是`http://10.253.0.237/cgi-bin/srun-portal`，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251021042022-1761020422628.png"/>

其中的载荷，
- callback：jQuery 回调函数标识，用于处理异步请求的返回结果，格式为动态生成的随机序列 + 时间戳。
- action：操作类型，login 表示此次请求是登录操作。
- username：登录用户名，格式为 [学号]@dx-uestc。
- password：密码，通过 MD5 加密（{MD5} 前缀标识加密方式），保障传输安全。
- ac_id、ip：网络接入标识和用户 IP 地址x.x.x.x，用于网络认证场景。
- chksum、info：登录校验的加密参数，用于防止请求被篡改，保障登录安全性。
- n、type、os、name：配置参数，如 os 为 Windows 10 表示操作系统类型，name 为 Windows 是系统名称。
- double_stack：网络协议标识（0 表示未使用双栈协议）。
- 最后的-参数：时间戳类标识，用于请求的唯一性校验。

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251021042709-1761020829280.png"/>

这里的载荷就有点难办了，这里的 password，chksum 和 info 就需要到前端的 js 中逆向获取了，打断点定位到的核心代码如下，

```js
var token = response.challenge,
    i = info({
        username: username,
        password: data.password,
        ip: (data.ip || response.client_ip),
        acid: data.ac_id,
        enc_ver: enc
    }, token),
    hmd5 = pwd(data.password, token);
var chkstr = token + username;
chkstr += token + hmd5;
chkstr += token + data.ac_id;
chkstr += token + (data.ip || response.client_ip);
chkstr += token + n;
chkstr += token + type;
chkstr += token + i;
var os = getOS();
var params = {
    action: "login",
    username: username,
    password: "{MD5}" + hmd5,
    ac_id: data.ac_id,
    ip: data.ip || response.client_ip,
    chksum: chksum(chkstr),
    info: i,
    n: n,
    type: type,
    os: os.device,
    name: os.platform,
    double_stack: data.double_stack
};
```

理性分析一下，也就是说正常我们模拟登录的流程如下，

1. 填写账号密码
2. 准备第一个载荷
3. 向`http://10.253.0.237/cgi-bin/get_challenge`这个网址发送请求获取 challenge 也就是 token 的值
4. 根据 token 值准备第二个载荷
5. 向`http://10.253.0.237/cgi-bin/srun-portal`这个网址发送登录验证请求

具体代码请参考:

::github{repo="b71db892/AutoLoginUESTC"}