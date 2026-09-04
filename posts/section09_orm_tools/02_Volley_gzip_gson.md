# Android volley customize request : gzip and gson.

> 原文網址：https://boochlin.com/?p=87
> 發布日期：2015-10-14
> 文章編號：87

---

『Volley is an HTTP library that makes networking for Android apps easier and most importantly, faster. Volley is available through the open AOSP repository.』

volley也已經用了好久一陣子，也有想過來寫一些使用心得

但是一看已經有很多高手都寫過類似的資料，因此就決定不出來獻醜

那就簡單說一下為何使用 volley 最為處理網路下載的lib

當時專案在go的時候，還在用httpconection來處理各種需求，可是這真的有夠麻煩

cdn的轉址，bitmap的轉置，因此動了找個lib來搞的想法，畢竟輪子那某多，何必kobe

一找之下，首先看到的是UIL，驚為天人，在找一下（何止輪子，連藍保基尼都有啦，UIL,GLIDE,PICCASO）

不過最後決定是volley的最根本理由根本就只是 google 官方應該會一直支援上去啦

廢話結束

#### volley強歸強，但是兒子還是要自己生

以下這兩個需求還是要自己搞

##### 1. gzip

 2. custom response object

第一項其實蠻簡單實用的，就是降低傳輸量

第二項官網有提到

[Implementing a Custom Request](https://developer.android.com/training/volley/request-custom.html)

GsonRequest:

目前流行的json 格式其實蠻固定的蠻適合直接轉成 對應的物件

，因此google 推出[gson](https://github.com/google/gson)把使用者覺的dirty的部份給作掉

json –> gson parse –> Object

gson的部份就不另外提，客官自己看

ps. 後來找到一個好像更厲害的 json to object 的parse lib，看效能居然是gson的四倍，

光是衝者四倍，決定來研究看看啦（[LoganSquare](https://github.com/bluelinelabs/LoganSquare)）

#### 詳細code 上菜啦

```
package com.acer.android.volley;

import com.acer.gson.Gson;
import com.android.volley.NetworkResponse;
import com.android.volley.ParseError;
import com.android.volley.Request;
import com.android.volley.Response;
import com.android.volley.Response.Listener;
import com.android.volley.toolbox.HttpHeaderParser;

import java.io.BufferedInputStream;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.HashMap;
import java.util.Map;
import java.util.zip.GZIPInputStream;

public abstract class GzipJsonObjectRequest extends Request {

    private final Gson gson = new Gson();
    private final Class clazz;
    private final Map<String, String> headers;
    private final Listener listener;
    private static final String GZIP_CONTENT_TYJE = "application/gzip;charset=UTF-8";
    private static final String GZIP_ENCODIING_TYPE = "gzip";

    public GzipJsonObjectRequest(String url, Class clazz, Map<String, String> headers,
                                 Listener listener, Response.ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        // 傳入對應的gson parsing object: clazz
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }


    @Override
    protected Response parseNetworkResponse(NetworkResponse response) {
        try {
            String jsonString = null;
            String contentType = response.headers.get("Content-Encoding");
            // 如果有支援gzip則利用GZIPInputStream把資料給讀出來
            if (contentType.equalsIgnoreCase(GZIP_ENCODIING_TYPE)) {
                try (GZIPInputStream zis = new GZIPInputStream(new BufferedInputStream(new ByteArrayInputStream(response.data)));
                     ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
                    byte[] buffer = new byte[1024];
                    int count;
                    while ((count = zis.read(buffer)) != -1) {
                        baos.write(buffer, 0, count);
                    }
                    byte[] bytes = baos.toByteArray();
                    jsonString = new String(bytes);
                }
            } else {
                jsonString = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            }
            // 最後gzip 拿出來的string 在利用gson parsing 成需要的 clazz
            return Response.success(
                    gson.fromJson(jsonString, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (IOException e) {
            return Response.error(new ParseError(e));
        }
    }
    @Override
    protected Map<String, String> getParams() {
        // 在send request 加入 gzip 要求，如果有支援的話
        HashMap<String, String> hashMap = new HashMap<>();
        hashMap.put("Accept-Encoding", GZIP_ENCODIING_TYPE);
        return hashMap;
    }
}
```

reference:

[gzipInputStream](https://github.com/bluelinelabs/LoganSquare)

[LoganSquare](https://github.com/bluelinelabs/LoganSquare)
