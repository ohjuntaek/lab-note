
https://hc.apache.org/httpcomponents-client-4.5.x/index.html

## Overview

- 비록 java.net 패키지가 HTTP 접근 기본적인거는 제공하지만 HttpClient 가 더 좋다


## Quick Start


```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpGet = new HttpGet("http://targethost/homepage");
CloseableHttpResponse response1 = httpclient.execute(httpGet);
// The underlying HTTP connection is still held by the response object
// to allow the response content to be streamed directly from the network socket.
// In order to ensure correct deallocation of system resources
// the user MUST call CloseableHttpResponse#close() from a finally clause.
// Please note that if response content is not fully consumed the underlying
// connection cannot be safely re-used and will be shut down and discarded
// by the connection manager. 
try {
    System.out.println(response1.getStatusLine());
    HttpEntity entity1 = response1.getEntity();
    // do something useful with the response body
    // and ensure it is fully consumed
    EntityUtils.consume(entity1);
} finally {
    response1.close();
}

HttpPost httpPost = new HttpPost("http://targethost/login");
List <NameValuePair> nvps = new ArrayList <NameValuePair>();
nvps.add(new BasicNameValuePair("username", "vip"));
nvps.add(new BasicNameValuePair("password", "secret"));
httpPost.setEntity(new UrlEncodedFormEntity(nvps));
CloseableHttpResponse response2 = httpclient.execute(httpPost);

try {
    System.out.println(response2.getStatusLine());
    HttpEntity entity2 = response2.getEntity();
    // do something useful with the response body
    // and ensure it is fully consumed
    EntityUtils.consume(entity2);
} finally {
    response2.close();
}
```


컨슘이 뭐더라.. 어쨌든 글케 어렵지 않다. 근데 요 위에꺼가

```java
/ The fluent API relieves the user from having to deal with manual deallocation of system
// resources at the cost of having to buffer response content in memory in some cases.

Request.Get("http://targethost/homepage")
    .execute().returnContent();
Request.Post("http://targethost/login")
    .bodyForm(Form.form().add("username",  "vip").add("password",  "secret").build())
    .execute().returnContent();
```

요렇게 된다고 하는데

```
implementation 'org.apache.httpcomponents:fluent-hc:4.5.5'
```

이거 넣으면 됨

