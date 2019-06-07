# DOM (Document Object Model)

- document를 객체로 구현한것이다.

{
  document: {
    html: {
      head: {
        title: ...
      },
      body: {
        header: ...
      }
    }
  }
}

- 이러한 형태로 만든것이 DOM 이다. 
최상위는 document이고 , 자식으로 접근시에는 html을 건너뛰고 head,body 로 접근한다.


# Node와 Element
- Node 는 태그 노드와, 텍스트 노드 전체를 가리킨다.
- Element는 텍스트 노드를 제외한 <p></p> 와 같은 태그만 가리키게 된다.
- 선택자 사용시 주의하며 사용해야한다.



# 속성

태그.nodeType 
- 태그 선택 후, nodeType 속성을 검색하면 , 해당 태그의 종류를 알려주는 숫자가 나온다.

1(Node.ELEMENT_NODE) -> Element 
3(Node.TEXT_NODE) -> 텍스트
8(Node.COMMNET_NODE) -> 주석
9(Node.DOCUMENT_NODE) -> Document 
10(Node.DOCUMENT_TYPE_NODE) -> DOCTYPE 
11(Node.DOCUMENT_FRAGMENT_NODE) -> Document Fragment 

2.4.5.6 은 더이상 쓰이지 않으며 7은 거의 안 쓰인다.


태그.children, 태그.childNodes 

document.body.children; // [header, main, footer, script]

- 자식 으로 접근시 children(텍스트 제외) , childNodes(텍스트 포함) 을 사용 한다.

* 모든 태그 에서 사용할 수 있다.
document.getElementById('header').children; 
- #header 의자식 들을 선택할 수 있다.



태그.firstChild, 태그.firstElementChild, 태그.lastChild, 태그.lastElementChild
- 태그의 첫번째, 마지막 자식을 선택할때 사용되며 , Element 냐 Node냐에 따라 달라진다.



태그.parentNode, 태그.parentElement 
- 부모를 찾을때 사용하는 속성이다. 부모는 항상 하나이기 때문에 단수형이다.


태그.previousSibling, 태그.nextSibling, 태그.previousElementSibling, 태그.nextElementSibling
- 형제태그를 찾을때 사용한다. 바로전, 후 의 형제를 찾을때 이용된다.


태그.innerHTML, 태그.outerHTML
- 선택한 태그의 내용을 얻어오거나, 바꿀수 있다.
- outerHTML : 현재 태그까지 포함
- innerHTML : 현재 태그미포함


태그.속성
- 태그를 선택하고 , 그 속성을 조회할 수 있다.
id,className(class), name,value,placeholder... 등등 


태그.attributes
- 해당 태그가 가진 모든 속성을 리턴한다.

태그.clientHeight, 태그.clientWidth
- 태그의 margin, border, scrollbar를 제외한 높이와 너비를 리턴한다.


태그.offsetHeight, 태그.offsetWidth
- 태그의 margin만 제외한 높이와너비를 리턴한다.

태그.scrollHeight, 태그.scrollWidth
- 스크롤 가능한 범위까지 포함한 높이와 너비를 리턴한다.



# 메서드 


태그.appendChild
- createElement() 함수로 생성한 태그를 추가할때 사용한다.
마지막 순서의 자식태그로 추가된다.


태그.removeChild
- 선택한 자식태그를 삭제한다. 

부모.insertBefore(넣을태그,기준태그);
- 선택한 태그를 기준의 이전에 추가한다. 

태그.cloneNode
- 자신을복사한다. 
