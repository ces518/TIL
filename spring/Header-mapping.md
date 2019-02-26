## Header

- Header의 정보를 이용하여 매핑이가능하다.
1. header = "key" : 해당 키가 있을경우 매핑
2. header = "!key" : 해당 키가 없을경우 매핑
3. header = "key=value" : 해당 키와 벨류가 있을경우 매핑

- parmeter정보를 이용한 매핑
    - 헤더와 동일한 방식이며 , params속성을 이용한다.

```
    @RequestMapping(value = "/header", headers = HttpHeaders.FROM)
    @ResponseBody
    public String header(){
        return "header";
    }
```