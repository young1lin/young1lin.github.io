+++
title = "Mac 下 Docker 更改国内镜像源"
date = 2020-07-16T00:00:00Z
tags = ["Docker"]
categories = ["tools"]
+++

# 打开 `DockerDesktop`

![docker更改国内镜像源.png](https://i.loli.net/2020/07/16/BM374ek9xKfoOZT.png)



# 如果你有其他配置改了的，直接复制这个，记住前面的逗号不要删

``` 
,"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn","https://hub-mirror.c.163.com"]
```

# 如果你之前什么都没改，复制粘贴下面内容

``` json
{
  "debug": true,
  "experimental": false,
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn","https://hub-mirror.c.163.com"]
}
```

# 如果在上图的界面报错，不能改

打开 Terminal 进入 Users/YourName目录下，复制粘贴进去

``` shell
cd .docker
vi deamon.json
```

重启`docker`，镜像生效

{% note danger %}

如果 `docker` 点 `prefrence`一直在更新状态，检查一下`daemon.json` 文件是否格式正确

{% endnote %}



# 如果不想换镜像源，可以开代理

> 别问我什么是代理，问就是不知道
>
> 我的本地代理端口是10080，你要是服务器的话，`http://[ ip address]:[port]`
>
> 像这样
>
> 二选一，别两个都写，或者打开 `terminal`，
>
> `export ALL_PROXY=socks5://127.0.0.1:10080;`
> `export http_proxy=socks5://127.0.0.1:10080; `
> `export https_proxy=socks5://127.0.0.1:10080;`，

![proxy.png](https://i.loli.net/2020/07/20/2vHdlr7B34L5fGX.png)

# 终极方案

##  Dockerfile

FROM 你的服务器后，加上，Docker Compose 建议下一种方法。

```dockerfile
ENV http_proxy <HTTP_PROXY>
ENV https_proxy <HTTPS_PROXY>
```

## 打开终端后设置

```bash
export http_proxy="<HTTPS_PROXY>"
export https_proxy="<HTTPS_PROXY>"
```



# 引用

> https://www.jianshu.com/p/419eaf4425a6
>
> http://pangguoming.com/blog/architecture/docker-configuration-file-daemon.json