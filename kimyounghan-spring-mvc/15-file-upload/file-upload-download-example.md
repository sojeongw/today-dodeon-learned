# 파일 업로드, 다운로드 예제

- 상품 관리
    - 상품 이름
    - 첨부 파일 한 개
    - 이미지 파일 여러 개
- 첨부 파일을 업로드, 다운로드 할 수 있다.
- 업로드 한 이미지를 웹 브라우저에서 확인할 수 있다.

## Item, UploadFile

{% tabs %} {% tab title="Item.java" %}

```java

@Data
public class Item {

    private Long id;
    private String itemName;
    private UploadFile attachFile;
    private List<UploadFile> imageFiles;

}
```

{% endtab %} {% tab title="UploadFile.java" %}

```java

@Data
public class UploadFile {

    // 업로드 한 파일 이름
    // 나중에 고객이 업로드 한 파일 리스트를 보여줄 때 출력한다.
    private String uploadFileName;

    // 시스템에 올린 파일 이름
    // 사용자들이 같은 이름으로 파일을 올릴 수 있기 때문에 둘을 구분한다.
    private String storeFileName;

    public UploadFile(String uploadFileName, String storeFileName) {
        this.uploadFileName = uploadFileName;
        this.storeFileName = storeFileName;
    }
}
```

{% endtab %} {% tab title="ItemRepository.java" %}

```java

@Repository
public class ItemRepository {
    private final Map<Long, Item> store = new HashMap<>();
    private long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }
}
```

{% endtab %} {% endtabs %}

- UploadFile은 파일 이름을 두 가지로 구분한다.
    - uploadFileName
        - 고객이 업로드 한 파일명
    - storeFileName
        - 서버 내부에서 관리하는 파일명
        - 여러 고객이 같은 이름을 쓰면 충돌이 나기 때문에 별도로 관리한다.

## FileStore

{% tabs %} {% tab title="FileStore.java" %}

```java

@Component
public class FileStore {

    @Value("${file.dir}")
    private String fileDir;

    public String getFullPath(String filename) {
        return fileDir + filename;
    }

    public List<UploadFile> storeFiles(List<MultipartFile> multipartFiles) throws IOException {
        List<UploadFile> storeFileResult = new ArrayList<>();

        for (MultipartFile multipartFile : multipartFiles) {
            if (!multipartFile.isEmpty()) {
                storeFileResult.add(storeFile(multipartFile));
            }
        }

        return storeFileResult;
    }

    // multipart 파일을 uploadFile로 변환한다.
    public UploadFile storeFile(MultipartFile multipartFile) throws IOException {
        if (multipartFile.isEmpty()) {
            return null;
        }

        // 사용자가 업로드 한 파일 이름
        String originalFilename = multipartFile.getOriginalFilename();
        // 서버에 저장할 파일 이름
        String storeFileName = createStoreFileName(originalFilename);

        // 지정한 디렉터리에 파일을 생성한다.
        multipartFile.transferTo(new File(getFullPath(storeFileName)));
        return new UploadFile(originalFilename, storeFileName);
    }

    // UUID를 이용해 파일 이름을 생성한다. 단, 어떤 파일인지 알기 위해 확장자는 남겨둔다.
    private String createStoreFileName(String originalFilename) {
        String ext = extractExt(originalFilename);
        String uuid = UUID.randomUUID().toString();

        return uuid + "." + ext;
    }

    // 확장자를 꺼낸다.
    private String extractExt(String originalFilename) {
        int pos = originalFilename.lastIndexOf(".");
        return originalFilename.substring(pos + 1);
    }
}
```

{% endtab %} {% endtabs %}

- multipart 파일을 서버에 저장한다.
- createStoreFileName()
    - 서버에 저장할 파일 이름은 UUID를 써서 충돌을 방지한다.
- extractExt()
    - 확장자를 따로 빼서 서버에 저장할 파일명에 붙여준다.
    - ex. 51041c62-86e4-4274-801d-614a7d994edb.png

## ItemForm

{% tabs %} {% tab title="ItemForm.java" %}

```java

@Data
public class ItemForm {

    private Long itemId;
    private String itemName;

    // 요구 사항에서 이미지는 여러 개를 첨부할 수 있으므로 MultipartFile를 리스트로 받는다.
    private List<MultipartFile> imageFiles;
    private MultipartFile attachFile;

}
```

{% endtab %} {% endtabs %}

- 상품 저장을 요청하는 객체를 생성한다.
- Item 도메인 객체와는 달리 UploadFile 대신 MultipartFile로 받는다.
- MultipartFile은 @ModelAttribute를 통해 사용할 수 있다.

## ItemController

{% tabs %} {% tab title="ItemController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class ItemController {
    private final ItemRepository itemRepository;
    private final FileStore fileStore;

    @GetMapping("/items/new")
    public String newItem(@ModelAttribute ItemForm form) {
        return "item-form";
    }

    @PostMapping("/items/new")
    public String saveItem(@ModelAttribute ItemForm form,
                           RedirectAttributes redirectAttributes) throws IOException {
        // 업로드 요청 한 파일과 이미지를 가져온다.
        UploadFile attachFile = fileStore.storeFile(form.getAttachFile());
        List<UploadFile> storeImageFiles = fileStore.storeFiles(form.getImageFiles());

        // 도메인 객체로 변환해 DB에 저장한다.
        Item item = new Item();
        item.setItemName(form.getItemName());
        // 사실 파일은 DB가 아니라 스토어 서비스에 저장한다. DB에 저장하는 건 파일을 저장한 곳의 상대 경로다.
        item.setAttachFile(attachFile);
        item.setImageFiles(storeImageFiles);

        itemRepository.save(item);

        redirectAttributes.addAttribute("itemId", item.getId());
        return "redirect:/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

- 등록과 저장 화면을 위한 컨트롤러를 정의한다.
- 저장한 아이템에 대한 페이지로 리다이렉트 한다.

{% tabs %} {% tab title="ItemController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class ItemController {
   
    ...

    // 고객이 업로드 한 파일 리스트를 보여준다.
    @GetMapping("/items/{id}")
    public String items(@PathVariable Long id, Model model) {
        Item item = itemRepository.findById(id);
        model.addAttribute("item", item);

        return "item-view";
    }

    // 파일 리스트에서 이미지를 보여줄 때 HTML img 태그에 넣을 이미지 주소를 반환한다.
    @ResponseBody
    @GetMapping("/images/{filename}")
    public Resource downloadImage(@PathVariable String filename) throws MalformedURLException {
        // UrlResource로 파일을 읽어서 @ResponseBody로 이미지 바이너리를 반환한다.
        // fullPath로 전체 경로 /Users/... 를 가져온다.
        return new UrlResource("file:" + fileStore.getFullPath(filename));
    }

    // 파일을 다운로드 할 때 사용한다.
    @GetMapping("/attach/{itemId}")
    public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
        // itemId로 요청을 받으면 접근 권한을 체크하는 등의 로직을 추가할 수 있다.
        Item item = itemRepository.findById(itemId);

        String storeFileName = item.getAttachFile().getStoreFileName();
        String uploadFileName = item.getAttachFile().getUploadFileName();

        // 실제 다운로드 받으려면 서버에 저장된 이름으로 가져와야 한다.
        UrlResource resource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));
        log.info("uploadFileName={}", uploadFileName);

        // 화면에 파일 이름을 노출할 때는 고객이 업로드 할 때 사용했던 이름을 사용한다.
        // 인코딩을 해줘야 파일명이 깨지는 위험을 방지할 수 있다.
        String encodedUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);
        String contentDisposition = "attachment; filename=\"" + encodedUploadFileName + "\"";

        // content-disposition 헤더를 추가하지 않으면 다운로드 하려고 파일명을 누르면
        // 다운로드가 되지 않고 열기로 작동해 파일 내용이 화면에 노출된다.
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
                .body(resource);
    }
}
```

{% endtab %} {% endtabs %}

- 고객이 업로드 한 파일과 이미지를 보여준다.
- @GetMapping("/images/{filename}")
    - <img> 태그로 이미지를 조회할 때 사용한다.
    - UrlResource
        - 저장했던 이미지 파일을 읽는다.
    - @ResponseBody
        - 이미지를 바이너리로 반환한다.
- @GetMapping("/attach/{itemId}"
    - 파일을 다운로드 한다.
    - 고객이 업로드 했던 이름으로 화면에 노출한다.
    - Content-Disposition 헤더를 적용해야 다운로드가 정상적으로 실행된다.
