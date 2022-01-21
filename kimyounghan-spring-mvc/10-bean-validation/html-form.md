# Form 전송 객체 분리

실무에서는 등록할 때 Form에서 전달하는 데이터가 Item 도메인 객체와 딱 맞지 않는다. 따라서 Item 대신 별도의 객체를 만들어 전달한다.

```text
HTML Form -> Item -> Controller -> Item -> Repository
```

- 장점
    - Item 도메인 객체를 컨트롤러, 리파지토리까지 직접 전달한다.
    - 중간에 Item을 만드는 과정이 없어 간단하다.
- 단점
    - 간단한 경우에만 적용 가능하다.
    - 수정할 때 검증이 중복될 수 있어 groups를 사용해야 한다.

```text
HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
```

- 장점
    - 전송하는 폼 데이터가 복잡해도 별도의 객체로 전달받을 수 있다.
    - 등록과 수정을 별도의 객체로 만들기 때문에 검증이 중복되지 않는다.
- 단점
    - Item 객체를 생성하는 변환 과정이 추가된다.

### 명명 규칙

ItemSave, ItemSaveForm, ItemSaveRequest, ItemSaveDto 등 의미있게 지으면 된다.

### 등록, 수정 뷰 템플릿이 비슷한데 통합이 가능한가?

어설프게 합치면 분기가 생겨서 나중에 유지보수할 때 고통스러울 수 있으므로 분리하는 게 좋다.