- OCP가 적용되지 않은 예시다.
- instance를 확인하는 if문이 추가된다.

```java
public interface CalculatorOperation {}

public class Addition implements CalculatorOperation {
  private double left;
  private double right;
  private double result = 0.0;

  public Addition(double left, double right) {
      this.left = left;
      this.right = right;
  }

  // getters and setters

}

public class Subtraction implements CalculatorOperation {
  private double left;
  private double right;
  private double result = 0.0;

  public Subtraction(double left, double right) {
      this.left = left;
      this.right = right;
  }

  // getters and setters
}

public class Calculator {

  public void calculate(CalculatorOperation operation) {
      if (operation == null) {
          throw new InvalidParameterException("Can not perform operation");
      }

      if (operation instanceof Addition) {
          Addition addition = (Addition) operation;
          addition.setResult(addition.getLeft() + addition.getRight());
      } else if (operation instanceof Subtraction) {
          Subtraction subtraction = (Subtraction) operation;
          subtraction.setResult(subtraction.getLeft() - subtraction.getRight());
      }
  }
}
```


- interface를 따로 분리해서 OCP를 지킨 사례다.
- if문으로 어떤 instance인지 확인하지 않아도 된다.

```java
public interface CalculatorOperation {
  void perform();
}

public class Addition implements CalculatorOperation {
  private double left;
  private double right;
  private double result;

  // constructor, getters and setters

  @Override
  public void perform() {
      result = left + right;
  }
}


public class Division implements CalculatorOperation {
  private double left;
  private double right;
  private double result;

  // constructor, getters and setters
  @Override
  public void perform() {
      if (right != 0) {
          result = left / right;
      }
  }
}


public class Calculator {

  public void calculate(CalculatorOperation operation) {
      if (operation == null) {
          throw new InvalidParameterException("Cannot perform operation");
      }
      operation.perform();
  }
}
```



- OCP를 위반한 사례다.

```java
class Square() {
  int height;
  int area() { return height * height; }
}

public class OpenOpenExample {

  public int compareArea(Square a, Square b) {
    return a.area() - b.area();
  } 

}

class Circle {
  int r;
  int area() { return Math.PI*r*r*;}
}


class OpenOpenExample {

  public int compareArea(Square a, Square b) {
    return a.area() - b.area();
  }

  public int compareArea(Circle x, Circle y) {
   return x.area() - y.area();
  }

}
```

- OCP를 잘 지킨 사례다.
- interface를 활용해서 parameter로 받는 곳에서 소스변경이 없게끔 구현했다.


```java
interface Shape {
  int area();
} 

class Circle implements Shape {
  int r;
  int area() { return Math.PI*r*r*;}
}

class Square() implements {
  int height;
  int area() { return height * height; }
}

public class OpenClosedExample {

  public int compareArea(Shape a, Shape b) {
    return a.area() - b.area();
  }

}
```