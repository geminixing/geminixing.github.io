---
layout: post
title: 陪你一起学OkHttp
tags:
- 开源
- 网络
categories: 开源
description: 
---
#### placeholder
    `OkHttpClient okHttpClient = new OkHttpClient().builder();//建议全局唯一
okHttpClient.cache(new Cache(fileDirectory, cacheSize)) //设置cache路径及大小,仅支持get请求缓存
.readTimeout(10, TimeUnit.SECONDS)
                .writeTimeout(10, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .pingInterval(10, TimeUnit.SECONDS) // websocket 轮训间隔
                .cookieJar(new CookieJar() { //设置cookie
saveFromResponse(HttpUrl url, List<Cookie> cookies){// 保存cookie通常使用SharedPreferences}
 List<Cookie> loadForRequest(HttpUrl url){// 从保存位置读取，注意此处不能为空，否则会导致空指针}
                  })    
                .addInterceptor(new Interceptor() {//普通拦截器，最早开始调用 })
                .addNetworkInterceptor(new Interceptor() {//连接成功（ConnectInterceptor）后调用})
                .sslSocketFactory(SSLSocketFactory, X509TrustManager) //HTTPS设置证书工厂
//post键值对，提交表单
RequestBody requestBody = new FormBody.Builder()
        .add("param", "value")
        .build(); 
//post json
RequestBody requestBody = RequestBody.create(MediaType.parse("application/json; charset=utf-8"),gson.toJson(params)); 
//文件上传
RequestBody requestBody = RequestBody.create(MediaType.parse("text/x-markdown; charset=utf-8"), new File("xxx.txt")); 
//Multipart文件上传，带字符参数
RequestBody requestBody = new MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart("fileParam", "xxx.png", RequestBody.create(MediaType.parse("image/png"), new File("xxx/xxx.png")))
                .addFormDataPart("param", "value")
                .build(); 
final Request request = new Request.Builder()
        .url("https://www.baidu.com") .addHeader("Authorization", token).addHeader("X-3GPP..","tel:..")
        .post(requestBody).build(); //有cache时强制网络获取cacheControl(CacheControl.FORCE_NETWORK)
Call call = okHttpClient.newCall(request);
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) { ... }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        String htmlStr =  response.body().string();
        String header = response.header("xxx"); //解析响应头
        List headers = response.headers("xxxList");
        XXX xxx = gson.fromJson(response.body().charStream(), XXX.class); //解析Json
    }
});`