# 템플릿 메소드 패턴

특정한 환경이나 상황에 맞게 확장하고 변경할 때 유용한 패턴이다.

추상 클래스에 메인이 되는 로직을 일반 메소드로 선언하고, 구현 클래스에 메소드를 선언해 호출한다.

## 장점

구현 클래스에서는 추상 클래스에 선언된 메소드만 사용하기 때문에 핵심 로직을 관리하기가 수월하다. 객체를 추가하고 확장하는 것이 가능해진다.

## 단점

추상 메소드가 많아지면 왔다갔다 수정해야 하므로 클래스 관리가 복잡하다.

## 적용 사례

```java
// template 추상 클래스를 하나 생성한다.
public abstract class HouseTemplate {
    // template method에 핵심 로직을 선언한다.
    // final로 선언해 서브 클래스에서 override 하지 못하도록 한다.
    public final void buildHouse() {
        buildFoundation();
        buildPillars();
        buildWalls();
        buildWindows();
        System.out.println("House is built.");
    }

    // default implementation
    private void buildFoundation() {
		System.out.println("Building foundation with cement,iron rods and sand");
	}

    private void buildWindows() {
		System.out.println("Building Glass Windows");
	}

    // 서브 클래스가 별도로 구현하길 원하는 메소드는 abstract으로 선언해서 그쪽에서 정의하도록 한다.
    public abstract void buildWalls();
    public abstract void buildPillars();
}

public class WoodenHouse extends HouseTemplate {
    // abstract로 선언된 메소드를 따로 정의해준다.
    @Override
    public void buildWalls() {
        System.out.println("Building Wooden Walls");
    }
    @Override
    public void buildPillars() {
        System.out.println("Building Pillars with Wood coating");
    }
}

public class GlassHouse extends HouseTemplate {
	@Override
	public void buildWalls() {
		System.out.println("Building Glass Walls");
	}
	@Override
	public void buildPillars() {
		System.out.println("Building Pillars with glass coating");
	}
}

public class HousingClient {
    public static void main(String[] args){
      HouseTemplate houseType = new WoodenHouse();
      houseType.buildHouse();

      houseType = new GlassHouse();
      houseType.buildHouse();;
    }
}
```

```text
Building foundation with cement,iron rods and sand
Building Pillars with Wood coating
Building Wooden Walls
Building Glass Windows
House is built.
************
Building foundation with cement,iron rods and sand
Building Pillars with glass coating
Building Glass Walls
Building Glass Windows
House is built.
```

결과는 이렇게 찍힌다. 즉, 공통 항목은 똑같이 사용하고 상황에 맞게 수정해야 할 것은 따로 수정할 수 있다.