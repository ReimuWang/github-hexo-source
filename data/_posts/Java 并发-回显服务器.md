---
title: Java 并发-回显服务器
date: 2017-10-12 17:24:36
tags: [Java,并发,回显]
categories: Java 并发
---

# 用套接字(Socket)编程实现一个多线程的回显(echo)服务器

<!-- more -->

```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class EchoServer {

    private final static Logger LOGGER = LoggerFactory.getLogger(EchoServer.class);

    private static class ClientHandler implements Runnable {

        private Socket client;

        public ClientHandler(Socket client) {
            this.client = client;
        }

        @Override
        public void run() {
            try(BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
                PrintWriter pw = new PrintWriter(this.client.getOutputStream())) {
                String msg = br.readLine();
                LOGGER.info("server get " + this.client.getInetAddress() + ":" + msg);
                pw.println("已处理 [" + msg + "]");
                pw.flush();
            } catch(Exception e) {
                LOGGER.error("fail to get data", e);
            } finally {
                try {
                    client.close();
                } catch (Exception e) {
                    LOGGER.error("fail to close client", e);
                }
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService es = null;
        try(ServerSocket server = new ServerSocket(6789)) {
            LOGGER.info("server start...");
            es = Executors.newFixedThreadPool(10);
            while(true) {
                Socket client = server.accept();
                es.submit(new ClientHandler(client));
            }
        } catch (Exception e) {
            LOGGER.error("server catch error", e);
        } finally {
            es.shutdown();
        }
    }
}
```

```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class EchoClient {

    private final static Logger LOGGER = LoggerFactory.getLogger(EchoClient.class);

    public static void main(String[] args) throws Exception {
        Socket client = new Socket("localhost", 6789);
        PrintWriter pw = new PrintWriter(client.getOutputStream());
        String msg = "八云蓝请求更多的鱼豆腐";
        pw.println(msg);
        pw.flush();
        LOGGER.info("client send:" + msg);
        BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
        LOGGER.info("server return:" + br.readLine());
        br.close();
        client.close();
    }
}
```

日志输出：

```
2017-10-12 17:14:59 echo.EchoServer:49 [INFO] - server start...
2017-10-12 17:15:04 echo.EchoServer:31 [INFO] - server get /127.0.0.1:八云蓝请求更多的鱼豆腐
2017-10-12 17:15:04 echo.EchoClient:21 [INFO] - client send:八云蓝请求更多的鱼豆腐
2017-10-12 17:15:04 echo.EchoClient:23 [INFO] - server return:已处理 [八云蓝请求更多的鱼豆腐]
```