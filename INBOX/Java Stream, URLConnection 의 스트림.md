
https://www.tutorialspoint.com/java/java_files_io.htm
- ![[Pasted image 20230421192923.png]]

ByteStream
- 8비트 바이트로 통신하는 듯
- FileInput/OutputStream

Character Streams
- 16비트 유니코드 쓰는듯
- **FileReader** and **FileWriter**. Though internally FileReader uses FileInputStream and FileWriter uses FileOutputStream but here the major difference is that FileReader reads two bytes at a time and FileWriter writes two bytes at a time.



![[Pasted image 20230421193258.png]]

https://victorydntmd.tistory.com/134

![[Pasted image 20230421195606.png]]
![[Pasted image 20230421195653.png]]

- 보조 스트림은 데코레이턴 패턴으로 쓴다.


```java

new BufferReader(new InputStreamReader(urlConnection.getInputStream()))


```

- urlConnection.getInputStream 디버그 찍어보니 HttpInputStream 이다
	- 이건 또 뭐냐.. 
	- sun.net.www.protocol.http.HttpURLConnection.HttpInputStream
	- java.io.FilterInputStream 상속한거네


sun.net.www.protocol.http.HttpURLConnection#getInputStream0
![[Pasted image 20230421201357.png]]

- sun.net.www.http.HttpClient#parseHTTP
	![[Pasted image 20230421201259.png]]
	- java.net.Socket#getInputStream
		- java.net.AbstractPlainSocketImpl#getInputStream
		  ![[Pasted image 20230421201152.png]]
	- sun.net.www.http.HttpClient#parseHTTPHeader
		- ![[Pasted image 20230421201844.png]]
		- 아.. Transfer-Encoding 이라는 헤더값의 디폴트가 Chunked 구나
		- 그래서 위에서 만들어 놨던 보조스트림인 BufferedInputStream 을 받은 ChunkedInputStream 으로 변환된 후 뭔가 작업을 하는 군



자 다시 돌아와서 기본 코드


```java
URL url = new URL("http://www.google.com");  
URLConnection urlConnection = url.openConnection();  
final InputStream inputStream = urlConnection.getInputStream();  
BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
```

디버그찍어보면 


![[Pasted image 20230421202033.png]]


sun.net.www.protocol.http.HttpURLConnection.HttpInputStream
![[Pasted image 20230421202112.png]]

- 자 이것도 보조스트림이므로 FilterInputStream 상속하긴 함 일단 이거는 보인다
-  자 이거를 도대체 왜 읽을려고 InputStreamReader에 담아서 BufferedReader 에 담나? 그거는 BufferedReader를 쓸려고 그냥 바꾼거임 끝




>  - 저 parseHeader 에 많은 정보가 있어보인다.
>  - 보조스트림, 주스트림 구분은 OK. 주스트림은 파일을 읽는 건데, 바이트 기반 vs 캐릭터 기반이 있는데
>  - java.net.SocketInputStream 보면 이거는 FileInputStream 상속, 즉 바이트 기반
>  - 이게 알 필요가 있냐고? 아직은 필요를 모르겠긴 함 헤더 Chunked 이정도만 알았네
>  - 그래도 맨날 궁금했다 BufferedReader(머시기(머시기)) 딱 눈에 안들어왔는데 이제 들어오네










