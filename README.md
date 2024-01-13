## 快速启动

### 1.安装Hugo(MacOS)
+ `brew install hugo`

### 2.下载blog源码

```git
git clone --depth=1 https://github.com/xxrBear/blog.git  # 下载blog源码只保留最近的一条提交

cd blog  # 进入blog目录

git submodule update --init  # 下载主题
```

### 3.本地启动Hugo项目
```shell
hugo
hugo server
```

打开 `localhost:1313` 就可以看到界面了
