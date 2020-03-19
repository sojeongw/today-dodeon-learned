# Theory

## 타입
### let

변수

### const

상수. 변하지 않는다.

### var

let처럼 값을 바꿀 수 있다. 하지만 같은 변수명으로 선언해도 정상적으로 작동한다.

```javascript
var name = 'a';
console.log(name);   // a

var name = 'b';
console.log(name);   // b
```

따라서 코드량이 많이질 수록 상태를 파악하기가 힘들다.