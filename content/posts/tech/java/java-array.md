---
title: "Java 数组"
date: 2025-05-15T12:02:08+08:00
lastmod: 2025-05-15T12:02:08+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 数组"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202505062222488.png" # 文章的图片
---

## 一、数组的基础概念

定义：数组是存储相同数据类型元素的固定长度的线性数据结构

特点：

- 元素类型相同
- 连续内存空间
- 长度固定
- 支持通过索引随机访问

## 二、数组的声明和初始化

### 2.1. 声明

```java
int[] arr; // 推荐
int arr[]; // 兼容C风格，不推荐
```

### 2.2. 初始化

#### 静态初始化（声明 + 赋值）

```java
int[] arr = {1, 2, 3, 4};
String[] names = {"Tom", "Jerry"};
```

#### 动态初始化（指定长度，后续赋值）

```java
int[] arr = new int[5]; // 默认值为 0
```

#### 动态初始化 + 直接赋值

```java
int[] arr = new int[]{1, 2, 3};
```

### 2.3 多维数组

```java
int[][] matrix = new int[3][4]; // 3行4列
int[][] matrix = {{1,2},{3,4},{5,6}};
```

### 2.4 不规则（交错）数组

```java
int[][] arr = new int[3][];
arr[0] = new int[2];
arr[1] = new int[3];
arr[2] = new int[1];
```

## 三、数组的访问

```java
System.out.println(arr[0]); // 访问第一个元素
arr[2] = 10; // 修改元素
```

索引越界异常：

- `ArrayIndexOutOfBoundsException`

## 四、数组的默认值

- 数值型：`0`
- 浮点型：`0.0`
- char：`'\u0000'`
- boolean：`false`
- 引用类型：`null`

## 五、Arrays 工具类

### 5.1 常用方法

| 方法                                     | 作用                 | 使用场景                         | 注意事项                         |
| ---------------------------------------- | -------------------- | -------------------------------- | -------------------------------- |
| `Arrays.sort(array)`                     | 升序排序             | 对基础类型或对象数组排序         | 默认升序，需排序前检查是否已排序 |
| `Arrays.sort(array, comparator)`         | 自定义排序           | 按自定义规则排序对象数组         | 需实现 `Comparator` 接口         |
| `Arrays.binarySearch(array, key)`        | 二分查找             | 查找已排序数组中的元素           | 数组必须已排序                   |
| `Arrays.fill(array, value)`              | 填充整个数组         | 快速初始化数组所有元素           | 全部元素都会被改为相同值         |
| `Arrays.fill(array, from, to, value)`    | 填充部分数组         | 填充指定范围元素                 | `[from, to)` 左闭右开区间        |
| `Arrays.equals(array1, array2)`          | 判断数组是否相等     | 判断两个一维数组内容是否相等     | 只适合一维数组                   |
| `Arrays.deepEquals(array1, array2)`      | 判断多维数组是否相等 | 比较多维数组内容是否相等         | 适用于多维数组                   |
| `Arrays.copyOf(original, newLength)`     | 复制数组             | 扩容、缩容、备份数组             | 超出原长度的部分自动填充默认值   |
| `Arrays.copyOfRange(original, from, to)` | 复制数组部分         | 复制数组指定范围                 | `[from, to)` 左闭右开区间        |
| `Arrays.toString(array)`                 | 转换为字符串         | 便于打印查看一维数组内容         | 只适合一维数组                   |
| `Arrays.deepToString(array)`             | 转换多维数组为字符串 | 便于打印查看多维数组内容         | 适用于多维数组                   |
| `Arrays.asList(array)`                   | 转换为固定大小 List  | 快速转 List（如用于 `for-each`） | 不能添加/删除元素，仅支持修改    |

### 5.2 示例

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

public class ArraysDemo {
    public static void main(String[] args) {
        // 1. sort 排序
        int[] arr1 = {5, 3, 2, 4, 1};
        Arrays.sort(arr1);
        System.out.println("排序后的 arr1: " + Arrays.toString(arr1));

        String[] strs = {"apple", "banana", "pear"};
        Arrays.sort(strs, Comparator.reverseOrder());
        System.out.println("自定义排序后的 strs: " + Arrays.toString(strs));

        // 2. binarySearch 二分查找
        int[] arr2 = {1, 2, 3, 4, 5};
        int index = Arrays.binarySearch(arr2, 3);
        System.out.println("元素 3 的索引: " + index);

        // 3. fill 填充
        int[] arr3 = new int[5];
        Arrays.fill(arr3, 9);
        System.out.println("填充后的 arr3: " + Arrays.toString(arr3));

        Arrays.fill(arr3, 1, 3, 7);
        System.out.println("部分填充后的 arr3: " + Arrays.toString(arr3));

        // 4. equals 比较
        int[] arr4a = {1, 2, 3};
        int[] arr4b = {1, 2, 3};
        System.out.println("arr4a 和 arr4b 是否相等: " + Arrays.equals(arr4a, arr4b));

        // 5. deepEquals 多维数组比较
        int[][] arr5a = {{1, 2}, {3, 4}};
        int[][] arr5b = {{1, 2}, {3, 4}};
        System.out.println("arr5a 和 arr5b 是否深度相等: " + Arrays.deepEquals(arr5a, arr5b));

        // 6. copyOf 复制
        int[] arr6 = {1, 2, 3};
        int[] copy1 = Arrays.copyOf(arr6, 5);
        System.out.println("扩容复制后的 copy1: " + Arrays.toString(copy1));

        int[] partCopy = Arrays.copyOfRange(arr6, 1, 3);
        System.out.println("范围复制后的 partCopy: " + Arrays.toString(partCopy));

        // 7. toString 数组转字符串
        int[] arr7 = {1, 2, 3};
        System.out.println("arr7 转字符串: " + Arrays.toString(arr7));

        // 8. deepToString 多维数组转字符串
        int[][] arr8 = {{1, 2}, {3, 4}};
        System.out.println("arr8 多维数组转字符串: " + Arrays.deepToString(arr8));

        // 9. asList 转 List
        String[] arr9 = {"a", "b", "c"};
        List<String> list = Arrays.asList(arr9);
        System.out.println("数组转 List: " + list);

        // 修改 asList 后的 List 元素
        list.set(0, "z");
        System.out.println("修改后的 List: " + list);
        System.out.println("对应的原数组: " + Arrays.toString(arr9));
    }
}
```

## 六、常见概念陷阱

- 数组是对象，`arr` 本身是引用类型
- `arr.length` 是属性，不是方法
- 多维数组实质上是数组的数组
- 不支持直接扩容（需要新建数组拷贝）
- 不支持删除元素（手动处理或使用集合类）
- 使用 `==` 比较数组时比较的是引用地址
