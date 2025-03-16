---
title: "Alembic 实战指南：快速入门到FastAPI 集成"
date: 2025-03-15T20:08:49+08:00
lastmod: 2025-03-15T20:08:49+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "在FastAPI项目中，快速上手 alembic 工具"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" # 文章的图片
---

## 一、快速开始

### 1.1 简介

Alembic 是一个基于 SQLAlchemy 的数据库迁移工具，主要用于管理数据库模式（Schema）的变更，例如新增表、修改字段、删除索引等，确保数据库结构与应用程序的 ORM 模型保持一致。

Alembic 通过版本控制来管理数据库模式的变化，每次迁移都会生成一个唯一的版本文件，类似于 Git 的提交（commit）。

- 在线模式：直接连接数据库，执行迁移。
- 离线模式：生成 SQL 语句，后续再执行。

### 1.2 安装

```shell
uv add alembic
```

### 1.3 初始化

```shell
uv run alembic init alembic

# 最后的alembic是项目的名字，可以修改，类似虚拟环境
```

### 1.4 alembic 目录结构

```shell
project/
│── alembic/             # Alembic 迁移目录
│   ├── versions/        # 迁移脚本存放目录
│   ├── env.py           # 迁移脚本配置（数据库连接、ORM 绑定等）
│   ├── script.py.mako   # 迁移脚本的模板
│── alembic.ini          # Alembic 配置文件
```

## 二、实战

### 2.1 实战简介

使用 FastAPI + SQLModel + PostgreSQL + Alembic 测试 Alembic 数据库迁移功能

### 2.2 项目初始化

- 创建测试文件夹

```shell
mkdir test-alembic
```

- 设置 uv 工程

```shell
cd test-alembic && uv init
```

如果你不知道 uv 是什么？推荐你看看博主的这篇文章哟~

[uv 入门指南](https://www.xxrbear.cn/posts/tech/tools/uv/)

- 增加项目依赖项

```shell
uv add fastapi sqlmodel psycopg2 alembic pydantic_core pydantic_settings
```

- 初始化 FastAPI 项目

```python
from fastapi import FastAPI

app = FastAPI()
```

- 创建项目 settings.py 模块

```python
from pydantic import ConfigDict, computed_field, PostgresDsn
from pydantic_core import MultiHostUrl
from pydantic_settings import BaseSettings


class Settings(BaseSettings):

    POSTGRES_SERVER: str
    POSTGRES_PORT: int = 5432
    POSTGRES_USER: str
    POSTGRES_PASSWORD: str = ""
    POSTGRES_DB: str = ""

    model_config = ConfigDict(
        env_file=".env"
    )  # 使用 model_config 替代 class ConfigDict

    @computed_field  # type: ignore[prop-decorator]
    @property
    def SQLALCHEMY_DATABASE_URI(self) -> PostgresDsn:
        return MultiHostUrl.build(
            scheme="postgresql+psycopg2",
            username=self.POSTGRES_USER,
            password=self.POSTGRES_PASSWORD,
            host=self.POSTGRES_SERVER,
            port=self.POSTGRES_PORT,
            path=self.POSTGRES_DB,
        )


settings = Settings()  # type: ignore
```

### 2.3 创建 models.py 模块

```python
import uuid
from datetime import datetime

from sqlmodel import Field, SQLModel


class User(SQLModel, table=True):
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    user_account: str = Field(..., max_length=32, description="账号")

    username: str = Field(
        default="", max_length=32, description="用户昵称", nullable=True
    )
    user_avatar: str = Field(
        default="", max_length=64, description="用户头像", nullable=True
    )
    user_profile: str = Field(
        default="", max_length=512, description="用户简介", nullable=True
    )
    user_role: str = Field(
        default="user", max_length=32, description="用户角色：user/admin/ban"
    )
    create_time: datetime = Field(default_factory=datetime.now, description="创建时间")
    update_time: datetime = Field(
        default_factory=datetime.now,
        sa_column_kwargs={"onupdate": datetime.now},
        description="更新时间",
    )
    is_delete: bool = Field(default=False, description="是否删除", nullable=True)

```

### 2.4 初始化 Alembic

```shell
uv run alembic init alembic
```

### 2.5 连接 PG 数据库

- 创建 .env 文件

```plain
POSTGRES_SERVER=localhost
POSTGRES_PORT=5432
POSTGRES_USER=admin
POSTGRES_PASSWORD=123456
POSTGRES_DB=test
```

假设你的 PG 数据库设置是上面这些

- 修改 env.py 文件 如下

```python
from logging.config import fileConfig

from alembic import context
from sqlalchemy import engine_from_config, pool

# 这是 Alembic 配置（Config）对象，它提供对当前使用的 .ini 配置文件中参数的访问
config = context.config

# 解释 Python 日志配置文件
# 这行代码的作用基本上是设置日志记录器 loggers
fileConfig(config.config_file_name)

# 在此添加你的模型的 MetaData 对象 以支持 自动生成（autogenerate） 迁移脚本
# User 必须存在，虽然看起来没有使用到，但是这样才会被SQLModel识别到！！！
from models import SQLModel, User
from settings import settings

target_metadata = SQLModel.metadata


# env.py 需要的其他配置值可以通过 config 获取, 例如
# my_important_option = config.get_main_option("my_important_option")


def get_url():
    return str(settings.SQLALCHEMY_DATABASE_URI)


def run_migrations_offline():
    """以离线模式运行迁移。
    在这种情况下，上下文（Context）仅使用数据库 URL 进行配置，而不会创建引擎（Engine），尽管这里也可以使用引擎
    通过跳过引擎的创建，我们甚至不需要数据库驱动（DBAPI）可用
    在此模式下，对 context.execute() 的调用会将指定的 SQL 语句直接输出到迁移脚本中
    """
    url = get_url()
    context.configure(
        url=url, target_metadata=target_metadata, literal_binds=True, compare_type=True
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online():
    """以在线模式运行迁移。

    在这种情况下，我们需要创建一个引擎（Engine）并将连接（Connection）与上下文（Context）关联
    """
    configuration = config.get_section(config.config_ini_section)
    configuration["sqlalchemy.url"] = get_url()
    connectable = engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, target_metadata=target_metadata, compare_type=True
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

- 修改 script.py.mako 文件如下

```plain
"""${message}

Revision ID: ${up_revision}
Revises: ${down_revision | comma,n}
Create Date: ${create_date}

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa

# 就是加了这一行
import sqlmodel.sql.sqltypes

${imports if imports else ""}

# revision identifiers, used by Alembic.
revision: str = ${repr(up_revision)}
down_revision: Union[str, None] = ${repr(down_revision)}
branch_labels: Union[str, Sequence[str], None] = ${repr(branch_labels)}
depends_on: Union[str, Sequence[str], None] = ${repr(depends_on)}


def upgrade() -> None:
    """Upgrade schema."""
    ${upgrades if upgrades else "pass"}


def downgrade() -> None:
    """Downgrade schema."""
    ${downgrades if downgrades else "pass"}

```

**注意：这个很重要，不然生成的迁移脚本有问题！！！**

### 2.6 实战指令

- 创建迁移文件

```python
uv run alembic revision --autogenerate -m "Initial migration"
```

- 执行迁移

```python
uv run alembic upgrade head
```

- 回滚迁移

```python
uv run alembic downgrade -1
```

- 查看当前版本

```python
uv run alembic current
```

- 查看历史版本

```python
uv run alembic history
```
