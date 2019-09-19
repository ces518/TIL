# String 비교 "상수".equals(객체), 객체.equals("상수")


/**
 * 모든 HTML 태그를 제거하고 반환한다.
 * 
 * @param html
 * @throws Exception  
 */
public String removeTag(String html) throws Exception {
	return html.replaceAll("<(/)?([a-zA-Z]*)(\\s[a-zA-Z]*=[^>]*)?(\\s)*(/)?>", "")
}
