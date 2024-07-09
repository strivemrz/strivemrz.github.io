---
title: 调用第三方api流式接口返回流式数据(仿chatgpt)
categories: springBoot
tags:
  - springBoot
excerpt: ''
cover: https://pic.mac89.com/pic/202112/06114136_72df3bb8ce.jpeg
---

#### 非流式接口

~~~java
@ApiOperation(value = "会话接⼝-非流式", notes = "")
@PostMapping(value = "/chatStream",produces = MediaType.APPLICATION_JSON_VALUE)
public String chatStream(@RequestBody String postBody ) {
    String result = Network.httpURLConnectionPOST("https://ws.scu.baidu.com/agent/chatStream/agi/api/v1/bot/chatStream",postBody);
    return result;
}
~~~

~~~java
public static String httpURLConnectionPOST (String url, String body) {
//        CloseableHttpClient httpClient = HttpClients.createDefault();
//        List<NameValuePair> params = new ArrayList<NameValuePair>();
//        for (Map.Entry<String, String> entry : requestParams.entrySet()) {
//            params.add(new BasicNameValuePair((String) entry.getKey(),
//                    (String) entry.getValue()));
//        }

        HttpPost post = new HttpPost(url);
        String ak = "AK密钥";
        String sk = "SK密钥";
        String requestTime = new Date().getTime()+"";
        String signatureBefore = ak+sk+requestTime+body;
        //添加header参数
        post.setHeader("Content-Type", "application/json;charset=UTF-8");
        post.setHeader("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
        post.setHeader("connection", "Keep-Alive");
        post.setHeader("accept", "*/*");

        post.setHeader("signature", getSHA256StrJava(signatureBefore));
        post.setHeader("requestTime", requestTime);
        post.setHeader("account", ak);

        UrlEncodedFormEntity formEntity = null;
        CloseableHttpResponse response = null;
        String res = "";
        try {
            CloseableHttpClient httpClient = buildSSLCloseableHttpClient();
            post.setEntity(new StringEntity(body, Encoding));
            RequestConfig requestConfig  =  RequestConfig.custom().setSocketTimeout(20000).setConnectTimeout(20000).setExpectContinueEnabled(true).setStaleConnectionCheckEnabled(true).build();
            post.setConfig(requestConfig);
//            formEntity = new UrlEncodedFormEntity(params, "utf-8");
//            post.setEntity(formEntity);
            response = httpClient.execute(post);
            HttpEntity resEntity = response.getEntity();
            res = EntityUtils.toString(resEntity, "utf-8");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if(response!=null){
                try {
                    EntityUtils.consume(response.getEntity());
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return res;
    }
~~~

#### 流式接口

~~~java
@ApiOperation(value = "会话接⼝-流式", notes = "")
@RequestMapping(value = "/chatStream_t",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter chatStream_t(@RequestBody String postBody) {
    SseEmitter emitter = null;
    try {
        JSONObject o = JSON.parseObject(postBody);
        String uid = "";
        if(!o.containsKey("userId")||StringUtils.isEmpty(o.getString("userId"))){
            uid = UUID.randomUUID().toString().replace("-", "");
            emitter = SseEmitterServer.connect(uid);
        }else {
            uid = o.getString("userId");
            emitter = SseEmitterServer.getSseEmitter(uid);
        }
        EventSourceUtil eventSourceUtil = new EventSourceUtil();
        eventSourceUtil.test(uid,postBody);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return emitter;
}
~~~

~~~java
package com.ceirp.qasearch.util;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

public class SseEmitterServer {

    private static final Logger logger = LoggerFactory.getLogger(SseEmitterServer.class);

    /**
     * 当前连接数
     */
    private static AtomicInteger count = new AtomicInteger(0);

    /**
     * 使用map对象，便于根据userId来获取对应的SseEmitter，或者放redis里面
     */
    private static Map<String, SseEmitter> sseEmitterMap = new ConcurrentHashMap<>();

    /**
     * 创建用户连接并返回 SseEmitter
     * @param employeeCode 用户ID
     * @return SseEmitter
     */
    public static SseEmitter connect(String employeeCode) {
        // 设置超时时间，0表示不过期。默认30秒，超过时间未完成会抛出异常：AsyncRequestTimeoutException
        SseEmitter sseEmitter = new SseEmitter(3600000L);
        // 注册回调
        sseEmitter.onCompletion(completionCallBack(employeeCode));
//        sseEmitter.completeWithError(errorCallBack(employeeCode));
        sseEmitter.onTimeout(timeoutCallBack(employeeCode));
        sseEmitterMap.put(employeeCode, sseEmitter);
        // 数量+1
        count.getAndIncrement();
        logger.info("创建新的sse连接，当前用户：{}", employeeCode);
        return sseEmitter;
    }

    /**
     * 根据code返回 SseEmitter
     * @param employeeCode 用户ID
     * @return SseEmitter
     */
    public static SseEmitter getSseEmitter(String employeeCode) {
        if(sseEmitterMap.containsKey(employeeCode)) return sseEmitterMap.get(employeeCode);
        else return connect(employeeCode);
    }

    /**
     * 给指定用户发送信息
     * @param employeeCode
     * @param jsonMsg
     */
    public static void sendMessage(String employeeCode, String jsonMsg) {
        try {
            SseEmitter emitter = sseEmitterMap.get(employeeCode);
            if (emitter == null) {
                logger.warn("sse用户[{}]不在注册表，消息推送失败", employeeCode);
                return;
            }
            emitter.send(jsonMsg, MediaType.APPLICATION_JSON);
        } catch (IOException e) {
            logger.error("sse用户[{}]推送异常:{}", employeeCode, e.getMessage());
            removeUser(employeeCode);
        }
    }

    /**
     * 群发消息
     * @param jsonMsg
     * @param employeeCodes
     */
    public static void batchSendMessage(String jsonMsg, List<String> employeeCodes) {
        employeeCodes.forEach(employeeCode -> sendMessage(employeeCode, jsonMsg));
    }

    /**
     * 群发所有人
     * @param jsonMsg
     */
    public static void batchSendMessage(String jsonMsg) {
        sseEmitterMap.forEach((k, v) -> {
            try {
                v.send(jsonMsg, MediaType.APPLICATION_JSON);
            } catch (IOException e) {
                logger.error("用户[{}]推送异常:{}", k, e.getMessage());
                removeUser(k);
            }
        });
    }

    /**
     * 移除用户连接
     */
    public static void removeUser(String employeeCode) {
        SseEmitter emitter = sseEmitterMap.get(employeeCode);
        if(emitter != null){
            emitter.complete();
        }
        sseEmitterMap.remove(employeeCode);
        // 数量-1
        count.getAndDecrement();
        logger.info("移除sse用户：{}", employeeCode);
    }

    /**
     * 获取当前连接信息
     */
    public static List<String> getIds() {
        return new ArrayList<>(sseEmitterMap.keySet());
    }

    /**
     * 获取当前连接数量
     */
    public static int getUserCount() {
        return count.intValue();
    }

    private static Runnable completionCallBack(String employeeCode) {
        return () -> {
            logger.info("结束sse用户连接：{}", employeeCode);
            removeUser(employeeCode);
        };
    }

    private static Runnable timeoutCallBack(String employeeCode) {
        return () -> {
            logger.info("连接sse用户超时：{}", employeeCode);
            removeUser(employeeCode);
        };
    }

    private static Throwable errorCallBack(String employeeCode) {
        logger.info("sse用户连接异常：{}", employeeCode);
        removeUser(employeeCode);
        return new Throwable();
    }


}
~~~

~~~java
package com.ceirp.qasearch.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.beust.jcommander.internal.Nullable;
import lombok.Data;
import okhttp3.*;
import okhttp3.sse.EventSource;
import okhttp3.sse.EventSourceListener;
import okhttp3.sse.EventSources;
import org.apache.http.conn.ssl.AllowAllHostnameVerifier;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLSession;
import java.util.Date;
import java.util.concurrent.TimeUnit;

import static com.ceirp.qasearch.util.Network.getSHA256StrJava;
import static com.ceirp.qasearch.util.SseEmitterServer.sendMessage;

@Data
public class EventSourceUtil {

    public void test(String uid,String bodyPar){
        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(10, TimeUnit.SECONDS)
                .writeTimeout(50, TimeUnit.SECONDS)
                .readTimeout(10, TimeUnit.MINUTES)
                .hostnameVerifier(new AllowAllHostnameVerifier())
                .build();

        EventSource.Factory factory = EventSources.createFactory(client);

        String ak = "AK密钥";
        String sk = "SK密钥";
        String requestTime = new Date().getTime()+"";
        String signatureBefore = ak+sk+requestTime+bodyPar;
        RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8"),bodyPar);
        // 请求对象
        Request request = new Request.Builder()
                .url("https://ws.scu.baidu.com/agent/chatStream/agi/api/v1/bot/chatStream")
                .post(body)
                .addHeader("Content-Type","application/json;charset=UTF-8")
                .addHeader("user-agent","Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)")
                .addHeader("connection","Keep-Alive")
                .addHeader("accept","*/*")
                .addHeader("signature",getSHA256StrJava(signatureBefore))
                .addHeader("requestTime",requestTime)
                .addHeader("account",ak)
                .build();

        // 自定义监听器
        EventSourceListener eventSourceListener = new EventSourceListener() {
            @Override
            public void onOpen(EventSource eventSource, Response response) {

                super.onOpen(eventSource, response);
                System.out.println("open");
            }

            @Override
            public void onEvent(EventSource eventSource, @Nullable String id, @Nullable String type, String data) {
                //   接受消息 data
                super.onEvent(eventSource, id, type, data);
                JSONObject o = JSON.parseObject(data);
                o.put("userId",uid);
                try {
                    System.out.println(JSON.toJSONString(o));
                    sendMessage(uid,JSON.toJSONString(o));
                }catch (Exception e){
                    e.printStackTrace();
                }
            }

            @Override
            public void onClosed(EventSource eventSource) {
                System.out.println("closed");
                super.onClosed(eventSource);
            }

            @Override
            public void onFailure(EventSource eventSource, @Nullable Throwable t, @Nullable Response response) {
                super.onFailure(eventSource, t, response);
                System.out.println("onFailure");
                t.printStackTrace();
            }
        };

        // 创建事件
        EventSource eventSource = factory.newEventSource(request, eventSourceListener);
//        CountDownLatch countDownLatch = new CountDownLatch(1);
//        try {
//            countDownLatch.await();
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }



    }

    private static HostnameVerifier trusts() {

        return new HostnameVerifier() {
            @Override
            public boolean verify(String hostname, SSLSession session) {
                HostnameVerifier verifier = HttpsURLConnection.getDefaultHostnameVerifier();
                return verifier.verify("*.baidu.com", session);
            }
        };

    }

}

~~~

#### 删除流式会话接口

~~~java
@ApiOperation(value = "删除会话接⼝", notes = "")
@RequestMapping(value = "/closeSSE")
public void closeSSE(String userId) {
    SseEmitterServer.removeUser(userId);
}
~~~

