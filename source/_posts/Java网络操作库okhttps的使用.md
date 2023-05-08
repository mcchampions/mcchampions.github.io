---
title: Java网络操作库okhttps的使用
date: 2023-05-08 19:42:44
tags:
---
网络请求:
```java
package a.b;

import okhttp3.*;
import org.apache.commons.lang3.StringUtils;
import org.json.JSONObject;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.*;
import java.util.*;

/**
 * 一些关于 网络请求 的相关方法
 * @author qscbm187531
 */
public class NetUtil {
    static OkHttpClient client = new OkHttpClient();

    /**
     * 发送普通POST请求
     *
     * @param url 链接地址
     */
    public static String sendPostJsonRequest(String url) throws IOException {
        return sendPostJsonRequest(url, new HashMap<>(), "");
    }

    /**
     * 发送普通POST请求（带Header）
     *
     * @param url 链接地址
     */
    public static String sendPostJsonRequest(String url, HashMap<String, String> Header) throws IOException {
        return sendPostJsonRequest(url, Header, "");
    }

    /**
     * 发送普通POST请求（带参数）
     *
     * @param url 链接地址
     */
    public static String sendPostJsonRequest(String url, String param) throws IOException {
        return sendPostJsonRequest(url, null, param);
    }

    /**
     * 发送普通POST请求（带Header和参数）
     *
     * @param url 链接地址
     * @param Header Header
     * @param param 参数
     */
    public static String sendPostJsonRequest(String url, HashMap<String, String> Header, String param) throws IOException {
        Request.Builder builder = new Request.Builder();
        builder.url(url).post(RequestBody.create(MediaType.parse("application/json"), param));
        JSONObject json = new JSONObject(Header);
        for (int i = 0; i < json.keySet().size(); i++) {
            builder.addHeader(Header.keySet().stream().toList().get(i), Header.values().stream().toList().get(i));
        }
        Response response = client.newCall(builder.build()).execute();
        String a = Objects.requireNonNull(response.body()).string();
        response.close();
        return a;
    }

    /**
     * 发送普通的Get请求（带Header）
     * @param url 链接地址
     */
    public static String sendGetRequest(String url, HashMap<String, String> Header) throws IOException {
        Request.Builder builder = new Request.Builder()
                .url(url)
                .get();
        for (int i = 0; i < MapUtil.ergodicHashMaps(Header).size(); i++) {
            builder.addHeader(MapUtil.ergodicHashMaps(Header).get(i).get(0).toString(), MapUtil.ergodicHashMaps(Header).get(i).get(1).toString());
        }

        Response response = client.newCall(builder.build()).execute();

        String a = Objects.requireNonNull(response.body()).string();
        response.close();
        return a;
    }

    /**
     * 上传资源图片
     *
     * @param path 文件路径
     * @param url 上传链接
     * @param Authorization Authorization
     */
    public static String uploadFile(String Authorization, String path, String url) throws IOException {
        File File = new File(path);
        OkHttpClient client = new OkHttpClient().newBuilder()
                .build();
        RequestBody body = new MultipartBody.Builder().setType(MultipartBody.FORM)
                .addFormDataPart("file", File.getName(),
                        RequestBody.create(MediaType.parse("application/octet-stream"),
                                new File(path)))
                .build();
        Request request = new Request.Builder()
                .url(url)
                .addHeader("Authorization", Authorization)
                .addHeader("Content-Type", "multipart/form-data")
                .method("POST", body)
                .build();
        try (Response response = client.newCall(request).execute()) {
            return Objects.requireNonNull(response.body()).string();
        }
    }

    /**
     * 模拟浏览器发送请求
     * @param url 链接地址
     */
    public static String simulationBrowserRequest(String url) throws IOException {
        HashMap<String, String> Header = new HashMap<>();
        Header.put("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36");
        return sendGetRequest(url, Header);
    }

    /**
     * 发送普通的Get请求
     * @param url 链接地址
     */
    public static String sendGetRequest(String url) throws IOException {
        return sendGetRequest(url, new HashMap<>());
    }

    /**
     * 从网络Url中下载文件
     *
     * @param url   路径
     * @param saveDir  保存路径
     * @param fileName 文件名称
     * @throws IOException IOException异常
     */
    public static void downloadFile(String url, File saveDir, String fileName) throws IOException {
        Request request = new Request.Builder()
                .url(url)
                .get()
                .addHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36")
                .build();
        Response response = client.newCall(request).execute();

        byte[] bytes = Objects.requireNonNull(response.body()).bytes();

        if (saveDir.exists()) {
            saveDir.mkdirs();
        }
        String folder = saveDir + File.separator + fileName;
        File file = new File(folder);
        FileOutputStream fos = new FileOutputStream(file);
        fos.write(bytes);
        fos.close();
    }

    /**
     * 从网络Url中下载文件（使用文件原本名字）
     *
     * @param url   路径
     * @param saveDir  保存路径
     * @throws IOException IOException异常
     */
    public static void downloadFile(String url, File saveDir) throws IOException {
        Request request = new Request.Builder()
                .url(url)
                .get()
                .addHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36")
                .build();
        Response response = client.newCall(request).execute();

        byte[] bytes = Objects.requireNonNull(response.body()).bytes();

        if (saveDir.exists()) {
            saveDir.mkdirs();
        }
        String folder = saveDir + File.separator + StringUtils.substringAfterLast(url, "/");
        File file = new File(folder);
        FileOutputStream fos = new FileOutputStream(file);
        fos.write(bytes);
        fos.close();
    }
}
```
WebScoket:
```java
package a.b;

import okhttp3.*;
import okio.ByteString;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;
import org.json.JSONObject;
import org.w3c.dom.events.EventException;

import java.io.IOException;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

/**
 * 事件触发
 * @author qscbm187531
 */
public class EventTrigger {
    public static EventTrigger p;
    public static String wssLo="";
    public static OkHttpClient okHttpClient = new OkHttpClient();
    public static OkHttpClient wss=new OkHttpClient.Builder()
            .pingInterval(30, TimeUnit.SECONDS) //保活心跳
            .build();
    public static  WebSocket mWebSocket;

    public static void main(@NotNull String Authorization) {
        wssLo = "wss:***";//地址
        Request request = new Request.Builder()
                .url(wssLo).build();//构建请求体
        mWebSocket = wss.newWebSocket(request, new WsListenerC(p));
    }

    public static class WsListenerC extends WebSocketListener {
        EventTrigger p;
        public WsListenerC1(EventTrigger p) {
            this.p=p;
        }
        @Override
        public void onMessage(@NotNull WebSocket webSocket, @NotNull ByteString bytes) {
            String text = bytes.utf8;
            //事件处理
        }

        @Override
        public void onFailure(@NotNull WebSocket webSocket, @NotNull Throwable t, @Nullable Response response) {
            t.printStackTrace();
        }

        @Override
        public void onClosed(@NotNull WebSocket webSocket, int code, @NotNull String reason) {
            mWebSocket.close(1000,"正常关闭");
        }
    }
}
```