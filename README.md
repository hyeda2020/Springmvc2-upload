Springmvc2-upload
-
리포지토리 설명 : 인프런 김영한님의 강의 '스프링 MVC 2편 - 백엔드 웹 개발 활용 기술' 파일 업로드편 정리

- 스프링은 `MultipartFile` 이라는 인터페이스로 다음과 같이 HTTP 멀티파트 파일을 매우 편리하게 지원
```
// 서버에 저장될 파일 경로 환경변수
@Value("${file.dir}")
private String fileDir;

// 파일 업로드
@PostMapping("/upload")
public String saveFile(@RequestParam MultipartFile file) throws IOException {
  if (!file.empty()) {
    String fullPath = fileDir + file.getOriginalFilename(); // 업로드 파일 명
    file.transferTo(new File(fullPath));  // 파일 저장
  }
  return "upload-form";
}

// 파일 다운로드
@GetMapping("/download/{filename}")
public ResponseEntity<Resource> download(@PathVariable String itemName) throws MalformedURLException {
  ... itemName으로 fileName 찾는 로직 생략 ...

  // fileName이름을 가진 파일 경로를 찾아서 리턴
  UrlResource resource = new UrlResource("file:" + fileStore.getFullPath(fileName));

  String encodedFileName = UriUtils.encode(fileName, StandardCharsets.UTF_8);
  String contentDisposition = "attachment; filename=\"" + fileName + "\"";
  return ResponseEntity.ok()
   .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
   .body(resource);
}
```  
  
※여러 사용자가 우연히 같은 이름의 파일을 업로드 하는 경우에 대비하기 위해  
다음과 같이 UUID를 활용해 서버에 저장되는 파일 이름이 겹치지 않도록 별도 지정하는 방안 필요  
(사용자가 업로드한 파일명과 실제 서버에 저장되는 파일명은 DB에 저장하여 매핑 및 관리)  
```
@Data
public class UploadFile {
  private String uploadFileName; // 사용자가 올린 파일 이름
  private String storeFileName;  // 실제 서버에 저장되는 파일 이름

  public UploadFile(String uploadFileName) {
    this.uploadFileName = uploadFileName;
    this.storeFileName = UUID.randomUUID().toString() + uploadFileName;
  }
}
```  

