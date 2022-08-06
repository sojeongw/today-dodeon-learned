# Heap 영역

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java/01/스크린샷%202020-07-06%20오전%2012.47.29.png)

Heap 영역은 Young, Old, Perm 세 가지로 나뉘는데, Young 영역에서 발생한 GC를 Minor, 나머지 두 영역에서 발생한 GC를 Major 혹은 Full이라고 부른다.

## Young

새롭게 생성한 객체가 위치하는 곳. 대부분의 객체가 이 영역에 생성되었다가 금방 닿을 수 없는 상태가 되어 사라진다.

## Old

Young 영역에서 참조할 수 있는 상태를 유지해 살아남은 객체가 이곳으로 복사된다. 대부분 Young 영역보다 크게 할당하며, 크기가 큰 만큼 Young 보다 GC가 적게 발생한다.

## Perm

클래스의 메타 정보나 static, 상수 정보들이 저장되는 공간이다. Java 8부터 static 객체는 heap 영역으로, 나머지는 Native Memory 영역으로 옮겨졌다.

## 객체의 이동 순서

Eden -> Survivor -> Old

메모리에 객체가 생성되면 그림에서의 Eden 영역에 객체가 지정된다. Eden 영역에 데이터가 쌓이면 기존 객체는 다른 곳으로 옮겨지거나 삭제된다. 이때 옮겨가는 위치가 Survivor Space라고 표시된 곳이다.

Survivor Space는 보통 두 개로 나눠져있고 우선순위가 따로 없이 비어있는 곳에 지정되기 때문에 둘 중 한 개의 영역은 반드시 비어있어야 한다.

그러다가 더 큰 객체가 생성되거나 더 이상 Young 영역에 공간이 남아있지 않다면 Old 영역으로 이동하게 된다.