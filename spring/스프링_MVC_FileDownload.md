# Spring - FileDownload
- FileResource를 읽어오는 방법
	- Spring ResourceLoader

- FileDownload시 응답 헤더
	- Content-Dispoistion: 사용자가 해당 파일을 다운로드시 사용할 파일명 
	- Content-Type: 파일의 타입
	- Content-Length: 파일의 크기

- Apache-Tika
	- 파일의 타입 (MediaType)
```xml
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>1.21</version>
</dependency>
```
- ResponseEntity
	- 응답 상태코드
	- 응답 헤더
	- 응답 본문

- FileDownload
	- ResourceLoader를 사용해서 fileName에 해당하는 Resource를 읽어온다.
	- resource에서 얻은 File객체를 Tika 를 활용하여 해당 파일의 Type 정보를 취득
	- ResponseEntity의 응답헤더로 CONTENT_DISPOSITION, CONTENT_TYPE, CONTENT_LENGTH 정보와 본문으로 resource를 응답한다.
```java
@Autowired
ResourceLoader resourceLoader;

@GetMapping("/file/{fileName}")
public ResponseEntity fileDownload (@PathVariable String fileName) throws IOException {
	Resource resource = resourceLoader.getResource("classpath:" + fileName);
	File file = resource.getFile();

	Tika tika = new Tika();
	String mediaType =  tika.detect(file);
	return ResponseEntity.ok()
			.header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
			.header(HttpHeaders.CONTENT_TYPE, mediaType)
			.header(HttpHeaders.CONTENT_LENGTH, String.valueOf(file.length()))
			.body(resource);
}
```	
