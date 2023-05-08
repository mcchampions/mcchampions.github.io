---
title: Java监听文件变化
date: 2023-05-08 19:55:35
tag:
- 文件监听
- commons-io
category:
- Java
---
首先引入相应依赖：
```xml
<dependency>
	<groupId>commons-io</groupId>
	<artifactId>commons-io</artifactId>
	<version>2.7</version>
</dependency>
```
注意，不同的版本需要不同的JDK支持，2.7需要Java 8及以上版本。

commons-io对实现文件监听的实现位于org.apache.commons.io.monitor包下，基本使用流程如下：

自定义文件监听类并继承 FileAlterationListenerAdaptor 实现对文件与目录的创建、修改、删除事件的处理；
自定义文件监控类，通过指定目录创建一个观察者 FileAlterationObserver；
向监视器添加文件系统观察器，并添加文件监听器；
调用并执行。

第一步：创建文件监听器。根据需要在不同的方法内实现对应的业务逻辑处理。
```java
public class FileListener extends FileAlterationListenerAdaptor {

    @Override
    public void onStart(FileAlterationObserver observer) {
        super.onStart(observer);
        System.out.println("onStart");
    }

    @Override
    public void onDirectoryCreate(File directory) {
        System.out.println("新建：" + directory.getAbsolutePath());
    }

    @Override
    public void onDirectoryChange(File directory) {
        System.out.println("修改：" + directory.getAbsolutePath());
    }

    @Override
    public void onDirectoryDelete(File directory) {
        System.out.println("删除：" + directory.getAbsolutePath());
    }

    @Override
    public void onFileCreate(File file) {
        String compressedPath = file.getAbsolutePath();
        System.out.println("新建：" + compressedPath);
        if (file.canRead()) {
            // TODO 读取或重新加载文件内容
            System.out.println("文件变更，进行处理");
        }
    }

    @Override
    public void onFileChange(File file) {
        String compressedPath = file.getAbsolutePath();
        System.out.println("修改：" + compressedPath);
    }

    @Override
    public void onFileDelete(File file) {
        System.out.println("删除：" + file.getAbsolutePath());
    }

    @Override
    public void onStop(FileAlterationObserver observer) {
        super.onStop(observer);
        System.out.println("onStop");
    }
}
```
第二步：封装一个文件监控的工具类，核心就是创建一个观察者FileAlterationObserver，将文件路径Path和监听器FileAlterationListener进行封装，然后交给FileAlterationMonitor
```java
public class FileMonitor {

	private FileAlterationMonitor monitor;

	public FileMonitor(long interval) {
		monitor = new FileAlterationMonitor(interval);
	}

	/**
	 * 给文件添加监听
	 *
	 * @param path     文件路径
	 * @param listener 文件监听器
	 */
	public void monitor(String path, FileAlterationListener listener) {
		FileAlterationObserver observer = new FileAlterationObserver(new File(path));
		monitor.addObserver(observer);
		observer.addListener(listener);
	}

	public void stop() throws Exception {
		monitor.stop();
	}

	public void start() throws Exception {
		monitor.start();

	}
}
```
第三步：运行
```java
public class FileRunner {
	public static void main(String[] args) throws Exception {
		FileMonitor fileMonitor = new FileMonitor(1000);
		fileMonitor.monitor("/Users/zzs/temp/example.yml", new FileListener());
		fileMonitor.start();
        FileMonitor file = new FileMonitor(1000);
        file.monitor("/Users/zzs/temp/", new FileListener());
        file.start();
	}
}
```