
https://hc.apache.org/httpcomponents-client-4.5.x/current/tutorial/pdf/httpclient-tutorial.pdf

[[HttpClient Overview, Quick Start]] 페이지에서 찾을 수 있는 PDF

한 7페이지라 좀 길긴한데 해야겠지?

## Chapter 1. Fundamentals

### 1.1 Request execution

- new HttpResponse(...).headerIterator()
- new BasicHeaderElementIterator(...)
	- => HeaderElement, getParameters
	- 음 파라미터..?
		- https://hc.apache.org/httpcomponents-core-4.4.x/current/httpcore/apidocs/org/apache/http/HeaderElement.html
		- ![[Pasted image 20230126180000.png]]
		오 이 문법 너무 좋은데 무슨 문법이지?
		어쨌든 이 정의에 따라, element도 있고(, 로 나눠짐) param도 있음 (; 로 나눠짐)
		그래서 BasicHeaderElement 에 파라미터가 있음
- entity
	- requests that use entities are referred to as **entity enclosing reqeusts**
		- POST and PUT
	- HttpClient distinguishes three kinds of entities, depending on where their content originates
		- 헐 이런게 있다니
		- streamed : 스트림
		- self-contained : 음.. 리피터블? 제일 많이 쓰인다? 커넥션이나 다른 엔터티로 독립적? 다 스트림 아니였나
		- wrapping : 콘텐츠가 다른 엔터티로부터 획득
		  음.. 어쨌든 뒤에 구분이 중요하다
		  it is suggested to consider non-repeatable entities as streamed, and those that are repeatable as self-contained. 라고 하네
	- Repeatable entities
		- ByteArrayEntity, StringEntity
	- Using Http entities
		- one can either retrieve the input stream via the HttpEntity#getContent() method, which returns an java.io.InputStream, or one can supply an output stream to the HttpEntity#writeTo(OutputStream) method
	- Ensuring release of low level resource

		```java
		CloseableHttpClient httpclient = HttpClients.createDefault();
		HttpGet httpget = new HttpGet("http://localhost/");
		CloseableHttpResponse response = httpclient.execute(httpget);
		try { 
			HttpEntity entity = response.getEntity();
			if (entity != null) { 
				InputStream instream = entity.getContent();
				try {
					// do something useful
				} finally { instream.close(); } 
			} 
		} finally { response.close(); }
		
		```

		- 위에 스트림 close 하는거는 커넥션을 유지하고 엔터티를 소비하면서 리스폰스를 닫는걸 시도하고, 뒤에 response 클로즈하는거는 그냥 닫아버린다.
		- EntityUtils#consume() 쓰면 콘텐츠 다쓰면 닫아준다
		- 일부만 가져오고 나머지 가져오는데 너무 비용 크면 response.close로 닫아줄 수 있다고 한다
	- 1.1.6 Consuming entity content
		- 아놔 튜토리얼이 왜케 어렵냐 영어 아오
		- 컨슘하는 방법 추천 : entity.getContent() 를 컨슘하거나 writeTo를 써라
		- EntityUtils 클래스 엄청나게 편하다
		- 근데 이게 HTTP Server가 신뢰할 수 있거나 제한 길이가 있지 않으면 강하게 비추
			>> 왜?
		- 한번 더 읽고 싶으면 new BufferedHttpEntity 써라
	- 1.1.7 Producing entity content
		- StringEntity, ByteArrayEntity, InputStreamEntity, FileEntity 로 데이터 담으면 된단다
		- InputStreamEntity 당연히 두번 못읽음
		- self-contained 를 구현하는걸 추천한다고..?
	- 1.1.7.1 HTML forms
		- 아니 코드 넣는데 왜이렇게 어렵냐.. 아 스트레스...
		```java
		List<NameValuePair> formparams = new ArrayList<NameValuePair>();
		formparams.add(new BasicNameValuePair("param1", "value1"));
		formparams.add(new BasicNameValuePair("param2", "value2"));
		UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams, Consts.UTF_8);
		HttpPost httppost = new HttpPost("http://localhost/handler.do");
		httppost.setEntity(entity);
		```
		- 어쨌든 위처럼 UrlEncodedFormEntity 를 쓰면 딱 url_encoded_ 그 서브밋 하는거 있잖아 그 엔터티 들어간단다
	- 1.1.7.2 Content chunking
		- setChunk(true) 
			- HTTP 버전이 지원 안하면 안먹힌단다 (1.0이 지원 안하는듯)
- 1.1.8 Response Handlers
	- ResponseHandler 쓰면 뭐 커넥션 걱정 안해도 되는데, 어떻게 안하냐하면

		```java

		CloseableHttpClient httpclient = HttpClients.createDefault();  
		HttpGet httpget = new HttpGet("http://localhost/json");  
		ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {  
		    @Override  
		    public JsonObject handleResponse(final HttpResponse response) throws IOException {  
		        StatusLine statusLine = response.getStatusLine();  
		        HttpEntity entity = response.getEntity();  
		        if (statusLine.getStatusCode() >= 300) {  
		            throw new HttpResponseException(  
		                    statusLine.getStatusCode(),  
		                    statusLine.getReasonPhrase());  
		        }  
		        if (entity == null) {  
		            throw new ClientProtocolException("no content");  
		        }
		        Gson gson = new GsonBuilder().create();  
		        ContentType contentType = ContentType.getOrDefault(entity);  
		        Charset charset = contentType.getCharset();  
		        Reader reader = new InputStreamReader(entity.getContent(), charset);  
		        return gson.fromJson(reader, MyJsonObject.class);  
		    }  
		};  
		MyJsonObject myjson = client.execute(httpget, rh);
		```

		- 마지막에 보면 MyJsonObject로 반환함
		- response 가지고 엔터티로 상태 검사하고, 콘텐츠 얻어서 인풋스트림리더로 gson으로 객체로 만들어주는 작업을 함

## 1.2 HttpClient interface

하 이제 12장이냐 절반도 못왔네..

```java
ConnectionKeepAliveStrategy keepAliveStrat = new DefaultConnectionKeepAliveStrategy() {  

    @Override  
    public long getKeepAliveDuration(  
            HttpResponse response,  
            HttpContext context) {  
        long keepAlive = super.getKeepAliveDuration(response, context);  
        if (keepAlive == -1) {  
            // Keep connections alive 5 seconds if a keep-alive value  
            // has not be explicitly set by the server            keepAlive = 5000;  
        }  
        return keepAlive;  
    }  
};  
CloseableHttpClient httpclient = HttpClients.custom()  
        .setKeepAliveStrategy(keepAliveStrat)  
        .build();

```

요런식으로 뭐 인증이나 리다이렉트나 킵어라이브나 커넥션 영속 같은 전략 인터페이스 구현으로 선택할 수 있게 한다 뭐 그렇다고 한다 의미는 알겠네

- 1.2.1 HttpClient thread safety : 스레드 세이프하게 해야한단다 같은 인스턴스로 리퀘스트 여러개 쓰는걸 추천한단다
- 1.2.2 HttpClient resource deallocation : finnaly 에서 close 해서 할당제거해란다.

## 1.3 HTTP execution context


아놔 망했다 그만하자

