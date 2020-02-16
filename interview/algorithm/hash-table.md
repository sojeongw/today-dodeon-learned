# 해시 테이블

누군가 유튜브에 이미 올라와 있는 컨텐츠를 다시 올리려고 하면 유튜브 측에서 중복된 영상이라는 에러를 띄운다. 영상 자체의 용량도 크거니와 유튜브에는 수 많은 영상이 올라와있는데 어떻게 감지하는 것일까?

이 모든 것은 `hash table` 덕분이다.

- 검색하려는 key 값(문자, 숫자, 파일 등)을 해시 함수에 넣어 돌린다.
 - key 값의 크기가 어떻든 동일한 함수의 해시코드를 만든다.
- 해시 코드를 반환 받는다.
- 배열의 인덱스로 환산한다.
- 해당 데이터에 접근한다.

블록체인도 해시 테이블을 사용한다. 블록체인은 10분마다 성사된 거래 내역을 블록체인 창고에 저장하고, 지금까지 일어난 모든 거래 내용을 유저들이 전자 지갑에 갖고 있게 하는 방식으로 이루어진다. 

서비스를 시작한 이래 모든 기록을 가지고 있으려면 용량이 엄청나게 클텐데 그것을 원본으로 대조하려면 시간이 오래 걸릴 것이다. 그래서 해시 코드로 만들어서 배포하고 거래 시에도 해시 코드가 같은지만 비교한다. 

해시 코드를 만드는 함수는 점이나 공백 하나만 달라져도 전혀 다른 해시 코드를 만든다. 즉, 입력 값이 완전히 일치해야 하기 때문에 누구 한 사람이 거래 내역을 조작할 수 없다.

## 특징

해시 테이블은 매우 빠른 속도의 알고리즘이다.

### 처리 과정

- 해시 함수의 결과로 정수 타입의 해시 코드를 받는다.
- 배열을 미리 고정된 크기만큼 만들어둔다.
- 배열 크기 만큼 해시 코드 값을 modulo 연산 해서 배열에 넣는다.
- 즉, 해시 코드 자체가 배열의 인덱스로 사용된다. 검색 자체를 할 필요가 없어져 해시코드로 바로 접근하므로 빠른 것이다.

## Hash Algorithm

배열의 한 방에만 계속 데이터가 쌓이고 다른 칸이 비어있으면 공간 효율이 떨어진다. 따라서 배열을 만들 때 규칙을 정하는 것이 매우 중요하다. 얼마나 잘 분배를 하느냐에 따라 퀄리티가 결정된다.

### Collision

해시 함수의 알고리즘이 좋지 않을 때 한 방에 몰리면 충돌이 생긴다. 해시 테이블의 최고 장점이 O(1)만큼 걸린다는 것인데 collision이 많으면 해당 방에 있는 모든 사람에 대해 검색을 수행해야 하므로 최악의 경우 O(n)이 걸린다.

- different keys -> same code
- different code -> same index

해시 함수는 서로 다른 키 값을 넣어도 같은 해시 코드를 반환할 수도 있다. 키 값은 문자열이라 그 수가 무한하지만 해시 코드는 정수 개만큼만 지원할 수 있기 때문이다.

또는 다른 해시 코드를 만들었는데 배열 방이 한정되어 있으니 같은 방에 배정받는 경우도 생긴다.

이렇게 한 방에 서로 겹쳐서 저장되는 경우를 모두 collision이라고 부른다.

## 구현

이 예시에서는 해당 캐릭터의 아스키값을 더해 해시코드를 만들어 본다.

> getHashCode("sung")
> s(115) + u(117) + n(110) + g(103) = 445
> j(106) + i(105) + n(110) = 321
> h(104) + e(101) + e(101) = 306
> m(109) + i(105) + n(110) = 324

> convertToIndex(hashCode)
> hashCode % size       // 해시 코드를 배열의 사이즈로 나눈 나머지를 배열 방의 인덱스로 사용한다.
> 445 % 3 = 1       // 1번 방 
> 321 % 3 = 0       // 0번 방
> 306 % 3 = 0       // 0번 방
> 324 % 3 = 0       // 0번 

위 알고리즘처럼 collision이 발생할 경우 데이터를 바로 배열에 저장하지 않고 배열 방 안에 LinkedList을 선언하고, 값을 배열 방에 할당할 때마다 LinkedList에 추가한다.

검색을 할 때는 검색할 키로 해시 함수를 통해 해시 코드를 만들고 그것을 인덱스로 환산해서 해당 방의 리스트를 돌아본다.

```java
import java.util.LinkedList;

class HashTable {
    // 해시테이블에 저장할 데이터를 노드에 담는다.
    class Node {
        String key;     // 검색할 키
        Strinv value;       // 검색 결과로 보여줄 
        
        // 노드를 생성할 때 키와 값을 받아서 객체에 할당한다.
        public Node(String key, String value) {
            this.key = key;
            this.value = value;
        }

        String getValue() {
            return value;
        }

        void setValue(String value) {
            this.value = value;
        }
    }

    // 위에서 만든 노드를 linkedList에 넣는다.
    LinkedList<Node>[] data;

    // 해시테이블을 만들 때 linkedList의 크기를 정해서 배열 방을 미리 만들어둔다.
    HashTable(int size) {
        this.data = new LinkedList[size];
    }

    // 해시 알고리즘을 구현하는 함수
    int getHashCode(String key) {
        int hashcode = 0;

        // 키의 문자 각각을 가져와서
        for(char c : key.toCharArray()) {
            // 해시 코드를 더한다.
            hashcode += c;
        }
        return hashcode;
    }

    int convertToIndex(int hashcode) {
        // 해시 코드를 배열방의 크기로 나눈 나머지 값을  index로 정한다. 
        return hashcode % data.length;
    }

    // 인덱스로 배열 방을 찾은 이후에 해당 방에 노드가 여러 개 존재할 경우
    // 검색 키를 가지고 해당 노드를 찾아온다.
    Node searchKey(LinkedList<Node> list, String key) {
        // 배열 방이 null이면 null을 반환한다.
        if(list == null) return null;
        // 그렇지 않으면 배열 방에 있는 linkedList를 돌면서 
        for(Node node : list) {
            // 해당 노드의 키와 검색하는 키가 같은지 확인한다.
            if(node.key.eqauls(key)) {
                // 같으면 노드를 반환한다.
                return node;
            }
        }
        // 같은 데이터를 못 찾았다면 null을 반환한다.
        return null;
    }

    // 데이터를 저장하는 함수에 저장할 키와 값을 받는다. 
    void put(String key, String value) {
        // key를 가지고 해시코드를 받아온다.
        int hashcode = getHashCode(key);
        // hashcode를 가지고 저장할 배열 방 번호 받아온다.
        int index = convertToIndex(hashcode);
        
        // 해시코드 확인용
        System.out.println(key + ". hashcode(" + hashcode + "), index(" + index + ")");

        // 배열 방 번호를 이용해서 기존 배열방에 있는 데이터를 가져온다.
        LinkedList<Node> list = data[index];

        // 배열 방이 null이면
        if(list == null) {
            // linkedList를 새로 생성한다.
            list = new LinkedList<Node>();
            // 새로 만든 리스트를 배열 방에 넣어준다.
            data[index] = list;
        }
        // 배열 방에 혹시 기존 키로 데이터를 가지고 있는지 확인하기 위해 노드를 받아온다.
        Node node = searchKey(list, key);
        // null이면 데이터가 없다는 뜻이므로
        if(node == null) {
            // 받아온 정보로 노드를 생성해서 리스트에 추가한다.
            list.addLast(new Node(key, value));
        } else {
            // 기존에 데이터가 있다면 해당 데이터를 새롭게 교체해준다.
            node.setValue(value);
        }
    }

    // 해당 key의 데이터를 가져오는 함
    String get(String key) {
        // key를 가지고 해시코드를 받아온다.
        int hashcode = getHashCode(key);
        // hashcode를 가지고 저장할 배열 방 번호 받아온다.
        int index = convertToIndex(hashcode);
        // 배열 방 번호를 이용해서 기존 배열방에 있는 데이터를 가져온다.
        LinkedList<Node> list = data[index];

        // linkedList에서 해당 key를 가진 노드를 검색한다.
        Node node = searchKey(list, key);

        // node를 못 찾았다면 not found, 찾았다면 해당 노드의 값을 리턴한다.
        return node == null ? "NOT FOUND" : node.getValue();
    }
}

public class Test {
    public static void main (String[] args) {
        // 고정된 방 3개를 생성한다.
        HashTable h = new HashTable(3);

        // 데이터를 넣는다.
        h.put("sung", "She is pretty");
        h.put("jin", "She is a model");
        h.put("hee", "She is an angel");

        // 해당 키의 값을 잘 가져오는지 확인한다.
        System.out.println(h.get("sung"));  // she is pretty
        System.out.println(h.get("jin"));
        System.out.println(h.get("hee"));
        // 없는 데이터를 넣었으므로 NOT FOUND
        System.out.println("jae");
        // 데이터 덮어쓰기
        h.put("sung", "She is cute");
        System.out.println("sung");
    }
}
```