### MultipartFile
- 파일업로드시 사용하는 메서드 아규먼트
- MultipartResolver 빈이 등록되어 있을시 사용할 수 있다. (부트는 자동적으로 등록.)
- POST mulitpart/form-data 요청에 들어있는 파일을 참조할수있다.
- List<MultipartFile> 형태로 여러개의 파일을 참조할 수 있다.
- * 파일업로드 관련 스프링부트 설정 클래스
- MultipartAutoConfiguration
- MultipartProperties  

```
@PostMapping("/files")
public String uploadFiles(
        @RequestParam MultipartFile file
        , RedirectAttributes redirectAttributes
) throws IOException {
    System.out.println(file.getOriginalFilename());

    try {
        File files = new File("/Users/june/IdeaProjects/demo-web-mvc/src/main/webapp/WEB-INF/upload/"+file.getOriginalFilename());
        file.transferTo(files);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (IllegalStateException e) {
        e.printStackTrace();
    }
    redirectAttributes.addFlashAttribute("message",file.getOriginalFilename() + "is uploaded");
    return "redirect:/files";
}
```