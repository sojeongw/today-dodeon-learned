# 템플릿/콜백의 응용

템플릿/콜백은 DI처럼 스프링의 독점 기술이 아니라 객체 지향적인 패턴이지만 스프링이 이러한 기능을 적극적으로 활용할 수 있도록 도와준다.

따라서 스프링 개발자라면 템플릿/콜백 패턴을 잘 사용할 줄 알아야 한다. 고정된 작업 흐름에 반복되는 코드가 있다면 분리하는 습관을 기르자.

- 중복된 코드를 메소드로 분리해보기
- 일부에 변경이 필요한 경우 인터페이스를 사이에 두고 분리해 전략 패턴과 DI를 활용하기
- 바뀌는 부분이 여러 개 만들어져야 한다면 템플릿/콜백 패턴 이용하기

## try/catch/finally

`try/catch/finally`는 템플릿/콜백 패턴을 적용하기에 적합한 코드다. 파일에 담긴 모든 숫자를 더해주는 코드를 만들어보자.

```text
1
2
3
4
```

먼저 테스트 코드를 작성한다.

```java
package springbook.learningtest.template;
...
public class CalcSumTest {
    @Test
    public void sumOfNumbers() throws IOException {
        Calculator calculator = new Calculator();
        int sum = calculator.calcSum(getClass().getResource("numbers.txt").getPath());
        assertThat(sum, is(10));
    }   
}
```

아래는 실제 애플리케이션 코드다.

```java
package springbook.learningtest.template;
...
public class Calculator {
    public Integer calcSum(String filepath) throws  IOException {
        // 한 줄씩 읽기 편하게 BufferedReader로 가져온다.
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            Integer sum = 0;
            String line = null;
            while((line = br.readLine()) != null) {
                sum += Integer.valueOf(line);
            }
            return sum;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            // BufferedReader 오브젝트가 생성되기 전에 예외가 발생할 수 있으므로 null 체크를 반드시 한다.
            if(br != null) {
                try {
                    // 한 번 연 파일은 반드시 닫아준다.
                    br.close();
                }
                catch(IOException e) {
                    System.out.println(e.getMessage());
                }
            }
        }
    }
}
```

파일을 읽거나 처리하다가 예외가 발생할 수 있으므로 `try/catch/finally`로 처리했다. 

## 중복 제거와 템플릿/콜백 설계

만약 곱하기 기능을 추가해야 한다면 어떻게 해야할까? 템플릿에 담을 반복 작업이 무엇인지 먼저 생각해본다. 그리고 템플릿이 콜백으로, 콜백이 템플릿으로 전달해야 할 내용이 무엇인지 생각한다.

여기서는 파일을 열고 라인을 읽는 기능을 콜백에게 전달하고 콜백은 각자 라인을 읽어서 결과를 템플릿에 돌려주도록 해보자.

인터페이스로 표현하면 이렇게 된다.

```java
...
public interface BufferedReaderCallback {
    Integer doSomethingWithReader(BufferedReader br) throws IOException;
}
```

템플릿 부분을 메소드로 분리한다.

{% tabs %}
{% tab title="After" %}
```java
public class Calculator {
    // 템플릿을 사용하는 calcSum() 메소드
    // 템플릿을 제외한 나머지 코드를 BufferedReaderCallback 인터페이스로 만든 익명 내부 클래스에 옮긴다.
    public Integer calcSum(String filepath) throws IOException {
        BufferedReaderCallback sumCallback = 
            new BufferedReaderCallback() {
                public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                    Integer sum = 0;
                    String line = null;
                    while((line = br.readLine()) != null) {
                        sum += Integer.valueOf(line);
                    }
                    return sum; 
            }       
        };
        // 템플릿이 처리할 파일과 익명 내부 클래스 오브젝트를 전달한다.
        return fileReadTemplate(filepath, sumCallback);
    }

    public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            // 콜백 오브젝트를 호출한다. 
            // 템플릿에서 만든 컨텍스트 정보(BufferedReader)를 전달하고 콜백의 결과를 받는다.
            int ret = callback.doSomethingWithReader(br);
            return ret;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if(br != null) {
                try { br.close(); }
                catch(IOException e) { System.out.println(e.getMessage()); }
            }
        }
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class Calculator {
    public Integer calcSum(String filepath) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            Integer sum = 0;
            String line = null;
            while((line = br.readLine()) != null) {
                sum += Integer.valueOf(line);
            }
            return sum;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if(br != null) {
                try { br.close(); }
                catch(IOException e) { System.out.println(e.getMessage()); }
            }
        }
    }
}
```
{% endtab %}
{% endtabs %}

이제 본격적으로 곱하는 메소드를 만들어보자. 일단 테스트를 진행한다.

{% tabs %}
{% tab title="After" %}
```java
package springbook.learningtest.template;
...
public class CalcSumTest {
    Calculator calculator;
    String numFilepath;

    @Before
    public void setUp() {
        this.calculator = new Calculator();
        this.numFilepath = getClass().getResource("numbers.txt").getPath();
    }

    @Test
    public void sumOfNumbers() throws IOException {
        assertThat(calculator.calcSum(this.numFilepath), is(10));
    }   

    public void multiplyOfNumbers() throws IOException {
        assertThat(calculator.calcMultiply(this.numFilepath), is(24));
    } 
}
```
{% endtab %}

{% tab title="Before" %}
```java
package springbook.learningtest.template;
...
public class CalcSumTest {
    @Test
    public void sumOfNumbers() throws IOException {
        Calculator calculator = new Calculator();
        int sum = calculator.calcSum(getClass().getResource("numbers.txt").getPath());
        assertThat(sum, is(10));
    }   
}
```
{% endtab %}
{% endtabs %}

테스트를 성공시키는 코드를 작성한다.

```java
public class Calculator {
    public Integer calcMultiply(String filepath) throws IOException {
        BufferedReaderCallback multiplyCallback = 
            new BufferedReaderCallback() {
                public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                    Integer multiply = 1;
                    String line = null;
                    while((line = br.readLine()) != null) {
                        multiply += Integer.valueOf(line);
                    }
                    return multiply; 
            }       
        };
        return fileReadTemplate(filepath, multiplyCallback);
    }
    
    public Integer calcSum(String filepath) throws IOException {
       ...
        return fileReadTemplate(filepath, sumCallback);
    }

    public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws  IOException {
       ...
    }
}
```

곱하기에도 적용한다.

## 리팩토링

그런데 `calcSum()`과 `calcMultiply()`는 비슷한 패턴으로 진행되고 있다. 각 라인에서 읽은 내용을 계산하다가 최종값을 리턴하는 것이다.

{% tabs %}
{% tab title="calcMultiply()" %}
```java
Integer multiply = 1;
String line = null;
while((line = br.readLine()) != null) {
    multiply += Integer.valueOf(line);
}
return multiply;
```
{% endtab %}

{% tab title="calcSum()" %}
```java
Integer sum = 1;
String line = null;
while((line = br.readLine()) != null) {
    sum += Integer.valueOf(line);
}
return sum;
```
{% endtab %}
{% endtabs %}

다시 코드를 정리해보자. 템플릿과 콜백을 찾아낼 때는 변하는 코드의 경계를 찾고 그 경계 사이에 주고받는 일정한 정보가 있는지 확인한다. 

변하는 부분인 `doSomethingWithReader()`를 콜백 인터페이스로 만들면 다음과 같다.

```java
...
public interface LineCallback {
    Integer doSomethingWithLine(String line, Integer value);
}
```

{% tabs %}
{% tab title="After" %}
```java
public class Calculator {
    // lineReadTemplate()을 사용하도록 수정한다.
    public Integer calcSum(String filepath) throws IOException {
      LineCallback sumCallback = new LineCallback() {
            public Integer doSomethingWithLine(String line, Integer value) {
                // 변경되는 코드
                return value + Integer.valueOf(line);
            }};
            // 더하기니까 초기값을 0으로 보낸다.
            return lineReadTemplate(filepath, sumCallbac, 0);
    }

    public Integer calcMultiply(String filepath) throws IOException {
        BufferedReaderCallback multiplyCallback = new LineCallback() {
                public Integer doSomethingWithReader(String line, BufferedReader br) {
                    return multiply * Integer.valueOf(line); 
            }       
        };
        // 곱하기니까 초기값을 1로 보낸다.
        return lineReadTemplate(filepath, multiplyCallback, 1);
    }

    // 새로 만든 라인 콜백과 계산 결과를 저장할 변수의 초기값을 파라미터로 받는다.
    public Integer lineReadTemplate(String filepath, LineCallback callback, int initVal) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            // 초기 값을 저장한다.
            int res = initVal;
            // 콜백에 있던 while 돌면서 파일을 읽어오는 부분도 템플릿으로 가져온다.
            while((line = br.readLine()) != null) {
                // 각 라인의 내용을 계산하는 부분만 콜백에게 맡긴다.
                res = callback.doSomethingWithLine(line, res);
            }
            return ret;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if(br != null) {
                try { br.close(); }
                catch(IOException e) { System.out.println(e.getMessage()); }
            }
        }
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class Calculator {
    public Integer calcSum(String filepath) throws IOException {
        BufferedReaderCallback sumCallback = 
              new BufferedReaderCallback() {
                  public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                      Integer sum = 0;
                      String line = null;
                      while((line = br.readLine()) != null) {
                          sum += Integer.valueOf(line);
                      }
                      return sum; 
              }       
          };
          return fileReadTemplate(filepath, sumCallback);
    }
    
    public Integer calcMultiply(String filepath) throws IOException {
        BufferedReaderCallback multiplyCallback = 
            new BufferedReaderCallback() {
                public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                    Integer multiply = 1;
                    String line = null;
                    while((line = br.readLine()) != null) {
                        multiply += Integer.valueOf(line);
                    }
                    return multiply; 
            }       
        };
        return fileReadTemplate(filepath, multiplyCallback);
    }

    public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            int ret = callback.doSomethingWithReader(br);
            return ret;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if(br != null) {
                try { br.close(); }
                catch(IOException e) { System.out.println(e.getMessage()); }
            }
        }
    }
}
```
{% endtab %}
{% endtabs %}

파일 로직 코드는 템플릿으로 분리되고 `Calculator` 클래스에 순수한 계산 로직만 남아 관심사를 명확하게 확인할 수 있다. 데이터를 가져와 계산한다는 핵심 기능에만 충실하게 바뀌었다.

## 제네릭스를 활용한 콜백 인터페이스

지금까지 만든 `LineCallback`과 `lineReadTemplate()`은 결과가 `Integer`로 고정되어 있었다. 결과 타입을 다양하게 하고 싶다면 제네릭스를 활용하면 된다.

{% tabs %}
{% tab title="After" %}
```java
public interface LineCallback<T> {
    T doSomethingWithLine(String line, T value);
}
```
{% endtab %}

{% tab title="Before" %}
```java
public interface LineCallback {
    Integer doSomethingWithLine(String line, Integer value);
}
```
{% endtab %}
{% endtabs %}

콜백 메소드의 리턴값과 파라미터 값을 제네릭 타입 `T`로 선언했다.

{% tabs %}
{% tab title="After" %}
```java
public class Calculator {
    public Integer calcSum(String filepath) throws IOException {
      LineCallback sumCallback = new LineCallback() {
            public Integer doSomethingWithLine(String line, Integer value) {
                return value + Integer.valueOf(line);
            }};
            return lineReadTemplate(filepath, sumCallbac, 0);
    }

    public Integer calcMultiply(String filepath) throws IOException {
        BufferedReaderCallback multiplyCallback = new LineCallback() {
                public Integer doSomethingWithReader(String line, BufferedReader br) {
                    return multiply * Integer.valueOf(line); 
            }       
        };
        return lineReadTemplate(filepath, multiplyCallback, 1);
    }

    // 제네릭스 덕분에 이제 String도 처리할 수 있게 되었다.
    public String concatenate(String filepath) throws IOException {
        // 사용할 타입을 String으로 지정했다.
		LineCallback<String> concatenateCallback = 
			new LineCallback<String>() {
			public String doSomethingWithLine(String line, String value) {
				return value + line;
			}};
            // 템플릿 메소드의 T는 모두 스트링이 된다.
			return lineReadTemplate(filepath, concatenateCallback, "");
	}	

    // 타입 파라미터 T를 추가해서 제네릭 메소드로 만든다.
    // T 타입으로 선언된 LineCallback 메소드를 호출해서 처리한 뒤 T 타입의 결과를 리턴하는 메소드가 된다.
    public T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            // 결과 값도 T로 바꿔준다.
            T res = initVal;
            while((line = br.readLine()) != null) {
                res = callback.doSomethingWithLine(line, res);
            }
            return ret;
        } catch (IOException e) {
            ...
        } finally {
           ...
        }
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class Calculator {
    public Integer calcSum(String filepath) throws IOException {
        ...
    }

    public Integer calcMultiply(String filepath) throws IOException {
        ...
    }

    public Integer lineReadTemplate(String filepath, LineCallback callback, int initVal) throws  IOException {
        BufferedReader br = null;
        
        try {
            br = new BufferedReader(new FileReader(filepath));
            int res = initVal;
            while((line = br.readLine()) != null) {
                res = callback.doSomethingWithLine(line, res);
            }
            return ret;
        } catch (IOException e) {
           ...
        } finally {
           ...
        }
    }
}
```
{% endtab %}
{% endtabs %}

해당 기능을 테스트해보자. 

```java
public class CalcSumTest {
	Calculator calculator;
	String numFilepath;
	
	@Before public void setUp() {
		this.calculator = new Calculator();
		this.numFilepath = getClass().getResource("numbers.txt").getPath();
	}
	
	@Test public void sumOfNumbers() throws IOException {
		assertThat(calculator.calcSum(this.numFilepath), is(10));
	}
	
	@Test public void multiplyOfNumbers() throws IOException {
		assertThat(calculator.calcMultiply(this.numFilepath), is(24));
	}
	
	@Test public void concatenateStrings() throws IOException {
		assertThat(calculator.concatenate(this.numFilepath), is("1234"));
	}
}
```

파일에 있는 1, 2, 3, 4를 스트링으로 받아 합치므로 `1234` 가 나와야 한다.

기존에 만든 `calcSum()`이나 `calcMultiply`도 String 대신 `Integer`로 인터페이스를 정의하면 그대로 사용할 수 있다.