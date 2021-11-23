# 메시지, 국제화

## 메시지

웹 페이지 상의 `상품명`이라는 이름을 `상품 이름`으로 바꿔야 한다면 수 십, 수 백개의 파일을 고쳐야 한다.

이런 다양한 메시지를 한 곳에서 관리하는 기능을 메시지 기능이라고 한다.

{% tabs %} {% tab title="messages.properties" %}

```properties
item=상품
item.id=상품 ID
item.itemName=상품명
item.price=가격
item.quantity=수량
```

{% endtab %} {% tab title="addForm.html" %}

```html
<label for="itemName" th:text="#{item.itemName}"></label>
```

{% endtab %} {% endtabs %}

메시지 관리용 파일을 만들면 각 html에서 key 값으로 데이터를 불러 사용할 수 있다.

## 국제화

{% tabs %} {% tab title="messages_en.properties" %}

```properties
item=Item
item.id=Item ID
item.itemName=Item Name
item.price=price
item.quantity=quantity
```

{% endtab %} {% tab title="messages_ko.properties" %}

```properties
item=상품
item.id=상품 ID
item.itemName=상품명
item.price=가격
item.quantity=수량
```

{% endtab %} {% endtabs %}

메시지를 각 나라별로 관리해 국제화 할 수 있다.

- 국가 인식 방법
    - HTTP accept-language 헤더 값
    - 사용자가 직접 언어 선택 후 쿠키를 사용해 처리