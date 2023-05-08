---
title: Java获取当前jar包的路径
date: 2023-05-08 19:35:55
tag:
- jar
- 路径
category:
- Java
---
有的时候，我们需要获取当前jar包的路径，但又不知道怎么实现，那就看这一篇播客吧
```java
     /**
     * 获取当前jar包的路径（不包含jar包本身）
     * @return 路径（如：”C:/Test/“)
     */
    public static String getJarPath() {
        String path = java.net.URLDecoder.decode(System.getProperty("java.class.path"));
        int firstIndex = path.lastIndexOf(System.getProperty("path.separator")) + 1;
        int lastIndex = path.lastIndexOf(File.separator) + 1;
        return path.substring(firstIndex, lastIndex);
    }
```