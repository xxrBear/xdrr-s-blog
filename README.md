## 简介
一个使用 Hugo + Papermod 主题搭建的个人博客网站，记录了自己的一点点学习经历...

## 快速启动

### 安装Hugo

<details>
    <summary>
       macos 
    </summary>

```shell
brew install hugo@0.99.1
```

</details>

<details>
<summary>
windows
</summary>

```shell
iwr -useb get.scoop.sh | iex

scoop install hugo-extended@0.99.1
```

</details>

<details>
<summary>
ubuntu
</summary>

```shell
sudo apt install hugo=0.99.1
```

</details>

### 下载源码

```git
git clone --depth=1 https://github.com/xxrBear/blog.git --recurse-submodules  # 下载blog源码
```

### 本地启动
```shell
hugo
hugo server
```

打开 `localhost:1313` 就可以看到界面了
