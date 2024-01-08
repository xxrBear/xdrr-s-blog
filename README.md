## 快速启动

### 1.MacOS安装Hugo
+ `brew install hugo`

### 2.下载blog源码

```git
# 下载blog源码只保留最近的一条提交
git clone --depth=1 https://github.com/xxrBear/blog.git
# 进入blog目录
cd blog
# 下载主题
git submodule update --init
```

### 3.启动Hugo项目
```shell
hugo
hugo server
```

打开 localhost:1313 就可以看到界面了
