# StringUtils
- java.lang.String 클래스 내의 활용가능한 메서드의 확장자원 
- http://commons.apache.org
- Lang > commons-lang-2.6-bin.zip 다운로드

- StringUtils 만으로 거의 대부분의 문자열 처리를 수행가능하다.
- 파라미터 값으로 null을 주더라도 절대 NullPointerException 을 발생시키지않는다.


- API
    - http://commons.apache.org/proper/commons-lang/javadocs/api-release/index.html


- 예제

- * StringUtilsTest.java 
```java
package com.chocolleto.board.user;

import org.apache.commons.lang.StringUtils;

public class StringUtilsTest {

    public static void main(String[] args) {

    String str;
    String str1;
    Boolean bool;

    str = "hello java.";
    // str이 java를 포함하고 있으면 true 반환.
    bool = StringUtils.contains(str, "java");
    System.out.println("contains : " + bool);

    // str이 null이면 "", 아니면 str 반환.
    str1 = StringUtils.defaultString(str);
    System.out.println("defaultString : " + str1);

    str = "h e l l o j a v a .";
    // 문자열 중 공백 문자가 있으면 모두 제거.
    str1 = StringUtils.deleteWhitespace(str);
    System.out.println("deleteWhitespace : " + str1);

    str = "chocolleto";
    str1 = "chocolleto";
    // str과 str1이 동일한지 유무 반환.
    bool = StringUtils.equals(str, str1);
    System.out.println("equals : " + bool);

    str = "JAVA";
    str1 = "java";
    // 대소문자 무시하고 str과 str1 비교.
    bool = StringUtils.equalsIgnoreCase(str, str1);
    System.out.println("equalsIgnoreCase : " + bool);

    str = "chocolleto chocolleto";
    // str에서 첫 번째 co의 인덱스를 반환. (인덱스는 0부터 시작)
    int i = StringUtils.indexOf(str, "co");
    System.out.println("indexOf : " + i);

    // str에서 마지막 to의 인덱스 반환.
    i = StringUtils.lastIndexOf(str, "to");
    System.out.println("lastIndexOf : " + i);

    // str이 null이거나 길이가 0이면 true 반환.
    bool = StringUtils.isEmpty(str);
    System.out.println("isEmpty : " + bool);

    // str이 null이 아니거나 길이가 0이 아니면 true 반환.
    bool = StringUtils.isNotEmpty(str);
    System.out.println("isNotEmpty : " + bool);

    String[] str3 = {"java", "javascript", "jQuery", "json"};
    str = " | ";
    // array에서 문자열을 읽어와 ' | '를 구분자로 연결.
    str1 = StringUtils.join(str3, str);
    System.out.println("join : " + str1);

    str = "CHOCOLLETO";
    // str을 소문자로 변환.
    str1 = StringUtils.lowerCase(str);
    System.out.println("lowerCase : " + str1);

    str = "chocolleto";
    //str을 대문자로 변환.
    str1 = StringUtils.upperCase(str);
    System.out.println("upperCase : " + str1);

    str = "HELLO java";
    // 대문자는 소문자로, 소문자는 대문자로 변환.
    str1 = StringUtils.swapCase(str);
    System.out.println("swapCase : " + str1);

    //문자열의 앞뒤 순서를 바꿈.
    str1 = StringUtils.reverse(str);
    System.out.println("reverse : " + str1);

    str = "c++, java, c#, javascript, jQuery";
    // ','를 구분자로 사용하여 분리.
    String[] str2 = StringUtils.split(str, ',');
    for(int j=0 ; j < str2.length ; j++) {
    System.out.println("split str2[" + j + "] : " + str2[j]);
    }

    str = " java ";
    // 문자열 좌우에 있는 공백 문자를 제거.(trim()과 동일.)
    str1 = StringUtils.strip(str);
    System.out.println("strip : " + str1);

    // 문자열 좌우 공백 문자 제거.
    str1 = StringUtils.trim(str);
    System.out.println("trim : " + str1);

    }

}
```
