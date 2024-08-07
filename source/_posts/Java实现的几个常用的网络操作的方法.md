---
title: Java实现的几个常用的网络操作的方法
date: 2024-07-10 21:16:36
tags: 
tag:
  - post
  - get
category: 
  - Java
  - 网络
  
---
```java
import io.github.minecraftchampions.dodoopenjava.DodoOpenJava;
import io.github.minecraftchampions.dodoopenjava.debug.Result;
import lombok.NonNull;
import lombok.extern.slf4j.Slf4j;
import org.json.JSONObject;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.SocketTimeoutException;
import java.net.URL;
import java.net.UnknownHostException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

/**
 * 一些 有关 网络请求 的相关方法
 *
 * @author qscbm187531
 */
@Slf4j
public class NetUtil {
    /**
     * 发送普通POST请求
     *
     * @param url 链接地址
     */
    public static String sendPostRequest(@NonNull String url) throws IOException {
        return sendPostRequest(url, new HashMap<>(0), "");
    }

    /**
     * 发送普通POST请求（带Header）
     *
     * @param url 链接地址
     */
    public static String sendPostRequest(@NonNull String url,
                                         @NonNull HashMap<String, String> header) throws IOException {
        return sendPostRequest(url, header, "");
    }

    /**
     * 发送普通POST请求（带参数）
     *
     * @param url 链接地址
     */
    public static String sendPostRequest(@NonNull String url, @NonNull String param) throws IOException {
        return sendPostRequest(url, new HashMap<>(0), param);
    }

    /**
     * 发送普通POST请求（带Header和参数）
     *
     * @param stringUrl 链接地址
     * @param header    Header
     * @param param     参数
     */
    public static String sendPostRequest(@NonNull String stringUrl,
                                         @NonNull HashMap<String, String> header,
                                         @NonNull String param) throws IOException {
        return sendPostRequest(stringUrl, header, param.getBytes(StandardCharsets.UTF_8));
    }

    /**
     * 发送普通POST请求（带Header和参数）
     *
     * @param stringUrl 链接地址
     * @param header    Header
     * @param param     参数
     */
    public static String sendPostRequest(@NonNull String stringUrl,
                                         @NonNull HashMap<String, String> header,
                                         byte[] param) throws IOException {
        try {
            URL url = new URL(stringUrl);
            HttpURLConnection con = (HttpURLConnection) url.openConnection();
            con.setRequestMethod("POST");
            con.setConnectTimeout(15000);
            con.setDoOutput(true);
            con.setUseCaches(false);
            con.setInstanceFollowRedirects(true);
            header.forEach(con::setRequestProperty);
            con.connect();
            OutputStream out = con.getOutputStream();
            out.write(param);
            out.flush();
            out.close();
            BufferedReader reader = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String line;
            StringBuilder sb = new StringBuilder();
            while ((line = reader.readLine()) != null) {
                sb.append(line).append("\n");
            }
            reader.close();
            return sb.toString();
        } catch (SocketTimeoutException e) {
            log.error("发送请求超时", e);
            return null;
        }
    }

    /**
     * 发送普通的Get请求（带Header）
     *
     * @param stringUrl 链接地址
     */
    public static String sendGetRequest(@NonNull String stringUrl,
                                        @NonNull HashMap<String, String> header) throws IOException {
        URL url = new URL(stringUrl);
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("GET");
        connection.setConnectTimeout(15000);
        connection.setDoOutput(false);
        connection.setInstanceFollowRedirects(true);
        header.forEach(connection::setRequestProperty);
        connection.setUseCaches(false);
        connection.connect();
        BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            sb.append(line).append("\n");
        }
        reader.close();
        return sb.toString();
    }

    /**
     * 上传资源图片
     *
     * @param path   文件路径
     * @param url    上传链接
     * @param header header
     */
    public static String uploadFile(@NonNull HashMap<String, String> header,
                                    @NonNull String path,
                                    @NonNull String url) throws IOException {
        try {
            StringBuilder sb = new StringBuilder();
            File file = new File(path);
            String boundary = "===" + System.currentTimeMillis() + "===";
            header.put("Content-Type", "multipart/form-data; boundary=" + boundary);
            sb.append("--").append(boundary).append("\r\n")
                    .append("Content-Disposition: form-data; name=\"file\"; filename=\"")
                    .append(file.getName()).append("\"")
                    .append("\r\n").append("Content-Type: ")
                    .append(HttpURLConnection.guessContentTypeFromName(file.getName())).append("\r\n")
                    .append("\r\n");
            String start = sb.toString();
            byte[] bytes;
            try (InputStream inputStream = new FileInputStream(file)) {
                bytes = inputStream.readAllBytes();
            }
            String end = "\r\n" + "--" + boundary + "--" + "\r\n";
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            os.write(start.getBytes());
            os.write(bytes);
            os.write(end.getBytes());
            byte[] byteArray = os.toByteArray();
            os.close();
            return sendPostRequest(url, header, byteArray);
        } catch (SocketTimeoutException e) {
            log.error("发送请求超时", e);
            return null;
        }
    }

    /**
     * 模拟浏览器发送请求
     *
     * @param url 链接地址
     */
    public static String simulationBrowserRequest(@NonNull String url) throws IOException {
        HashMap<String, String> header = new HashMap<>(1);
        header.put("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36");
        return sendGetRequest(url, header);
    }

    /**
     * 发送普通的Get请求
     *
     * @param url 链接地址
     */
    public static String sendGetRequest(@NonNull String url) throws IOException {
        return sendGetRequest(url, new HashMap<>(0));
    }
}
```
