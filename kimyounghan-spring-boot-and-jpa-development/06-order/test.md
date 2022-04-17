# 주문 기능 테스트

- cmd + shift + t
    - 테스트 코드와 구현 코드 간의 이동
- cmd + opt + p
    - 파라미터로 리팩터링

## 상품 주문

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Autowired
    EntityManager em;

    @Autowired
    OrderService orderService;

    @Autowired
    OrderRepository orderRepository;

    @Test
    public void 상품주문() {
        // given
        Member member = createMember();
        Item book = createBook("시골 JPA", 10000, 10);

        int orderCount = 2;

        // when
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        // then
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals("상품 주문 시 상태는 ORDER", OrderStatus.ORDER, getOrder.getStatus());
        assertEquals("주문한 상품 종류 수가 정확해야 한다.", 1, getOrder.getOrderItems().size());
        assertEquals("주문 가격은 가격 * 수량이다.", 10000 * orderCount, getOrder.getTotalPrice());
        assertEquals("주문 수량만큼 재고가 줄어야 한다.", 8, book.getStockQuantity());
    }
}
```

## 재고 수량 초과

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Test(expected = NotEnoughStockException.class)
    public void 상품주문_재고수량초과() {
        // given
        Member member = createMember();
        Item book = createBook("시골 JPA", 10000, 10);

        int orderCount = 11;

        // when
        orderService.order(member.getId(), book.getId(), orderCount);

        // then
        // 테스트가 실패하면 메시지를 띄운다.
        fail("재고 수량 부족 예외가 발생해야 한다.");
    }
}
```

### 주문 취소

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Test
    public void 주문취소() {
        // given
        Member member = createMember();
        Item book = createBook("시골 JPA", 10000, 10);

        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        // when
        orderService.cancelOrder(orderId);

        // then
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals("주문 취소 시 상태는 CANCEL이다.", OrderStatus.CANCEL, getOrder.getStatus());
        assertEquals("주문이 취소된 상품은 그만큼 재고가 증가해야 한다.", 10, book.getStockQuantity());
    }
}
```