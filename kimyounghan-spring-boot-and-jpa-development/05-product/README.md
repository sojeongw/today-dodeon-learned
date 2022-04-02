# 상품 도메인 개발
## 상품 Entity 개발 및 비즈니스 로직 추가

재고를 늘리고 줄이는 로직을 추가할 것이다. 도메인 주도 설계는 Entity 자체에서 해결할 수 있는 문제는 Entity 안에 비즈니스 로직을 넣는 게 좋다. 객체지향적인 솔루션이다. 해당 Entity 데이터를 가지고 있는 곳에서 비즈니스 로직이 나가는 게 가장 응집도가 있다.

```java
@Entity
@Getter
@Setter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
public abstract class Item {

  @Id
  @GeneratedValue
  @Column(name = "item_id")
  private Long id;

  private String name;
  private int price;
  private int stockQuantity;

  @ManyToMany(mappedBy = "items")
  private List<Category> categories = new ArrayList<>();

  /**
   * stock 증가
   */
  public void addStock(int quantity) {
    this.stockQuantity += quantity;
  }

  /**
   * stock 감소
   */
  public void removeStock(int quantity) {
    int restStock = this.stockQuantity - quantity;

    if(restStock < 0) {
      throw new NotEnoughStockException("need more stock");
    }

    this.stockQuantity = restStock;
  }
}

```

옛날에는 itemService에서 stockQuantity를 가져와서 로직을 처리한 다음 item.setQuantity() 해서 많이 처리했다. 하지만 데이터가 있는 쪽에 비즈니스 메서드가 있는 것이 더 좋다.

```java
public class NotEnoughStockException extends RuntimeException {

  public NotEnoughStockException() {
    super();
  }

  public NotEnoughStockException(String message) {
    super(message);
  }

  public NotEnoughStockException(String message, Throwable cause) {
    super(message, cause);
  }

  public NotEnoughStockException(Throwable cause) {
    super(cause);
  }

  protected NotEnoughStockException(String message, Throwable cause, boolean enableSuppression,
      boolean writableStackTrace) {
    super(message, cause, enableSuppression, writableStackTrace);
  }
}
```

exception은 RuntimeException을 상속해서 메서드를 override 한다.

## 상품 리포지토리 개발

```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {
  private final EntityManager em;
  
  public void save(Item item){
    if (item.getId() == null) {
      // 없으면 신규로 등록한다.
      em.persist(item);
    } else {
      // 업데이트와 비슷하다고 일단 알아두고 나중에 설명한다.
      em.merge(item);
    }
  }
  
  public Item findIne(Long id) {
    return em.find(Item.class, id);
  }
  
  public List<Item> findAll() {
    return em.createQuery("select i from Item i", Item.class)
        .getResultList();
  }
}
```

## 상품 서비스 개발

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {
  private final ItemRepository itemRepository;

  // readOnly가 아닌 곳에 안 달아주면 저장이 안된다.
  @Transactional
  public void saveItem(Item item) {
    itemRepository.save(item);
  }

  public List<Item> findItem() {
    return itemRepository.findAll();
  }

  public Item findOne(Long itemId) {
    return itemRepository.findOne(itemId);
  }
}
```