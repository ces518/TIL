# AJAX 비동기시 XSS Filtering
- HttpMessageConverter를 사용하기때문에 Filter LEVEL의 처리가 적용되지 않음
- 비동기시엔 일일히 API 핸들러에서 처리를 해주어야한다.
- MessageConverter를 커스텀하여 특정 문자열을 변형해주는 형태로 사용

- XSSEsacpe 문자를 지정
- 커스텀한 문자열이 필요없을경우 StringEscapeUtils를 활용
```java
public class XSSEscapes extends CharacterEscapes {
    private final int[] asciiEscapes;

    private final CharSequenceTranslator translator;

    public XSSEscapes() {

        // 1. XSS 방지 처리할 특수 문자 지정
        asciiEscapes = CharacterEscapes.standardAsciiEscapesForJSON();
        asciiEscapes['<'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['>'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['&'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['\"'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['('] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes[')'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['#'] = CharacterEscapes.ESCAPE_CUSTOM;
        asciiEscapes['\''] = CharacterEscapes.ESCAPE_CUSTOM;

        // 2. XSS 방지 처리 특수 문자 인코딩 값 지정
        translator = new AggregateTranslator(
                new LookupTranslator(EntityArrays.BASIC_ESCAPE()),  // <, >, &, " 는 여기에 포함됨
                new LookupTranslator(EntityArrays.ISO8859_1_ESCAPE()),
                new LookupTranslator(EntityArrays.HTML40_EXTENDED_ESCAPE()),
                // 여기에서 커스터마이징 가능
                new LookupTranslator(
                        new String[][]{
                                {"(",  "&#40;"},
                                {")",  "&#41;"},
                                {"#",  "&#35;"},
                                {"\'", "&#39;"}
                        }
                )
        );
    }

    @Override
    public int[] getEscapeCodesForAscii() {
        return asciiEscapes;
    }

    @Override
    public SerializableString getEscapeSequence(int ch) {
//        return new SerializedString(translator.translate(Character.toString((char) ch)));
        return new SerializedString(StringEscapeUtils.escapeHtml4(Character.toString((char) ch)));
    }
}
```

- 앞서 정의한 XSSEscape 를 Jackson ObjectMapper에 등록
- MessageConverter로 등록해준다.
```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    super.configureMessageConverters(converters);
    converters.add(xssEscapeConverter());
}

private HttpMessageConverter<?> xssEscapeConverter () {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.getFactory().setCharacterEscapes(new XSSEscapes());
    MappingJackson2HttpMessageConverter xssEscapeConverter =
            new MappingJackson2HttpMessageConverter();
    xssEscapeConverter.setObjectMapper(objectMapper);
    return xssEscapeConverter;
}
```
