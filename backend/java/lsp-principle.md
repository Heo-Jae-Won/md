```java
public class BookDelivery {

     String titles;
     Integer userID;

     void getDeliveryLocations() {
           ...
     }
}
```

```java
public class PosterMapDelivery extends BookDelivery {

     @Override
     void getDeliveryLocations() {
              ...
     }
}
```

```java
public class AudioBookDelivery extends BookDelivery {

     @Override
     void getDeliveryLocations() {

           /* can't be implemented since

            * Audio book doesn't have

            * a physical location. */

     }

}
```


```java
public class BookDelivery {

     String title;
     Integer userID;

}


public class OfflineDelivery extends BookDelivery {

     void getDeliveryLocations() {
          ...
     }
}


public class OnlineDelivery extends BookDelivery {

     void getSoftwareOptions() {
          ...
     }
}
```


```java
public class PosterMapDelivery extends OfflineDelivery {

     @Override
     void getDeliveryLocations() {
          ...
     }

}

public class AudioBookDelivery extends OnlineDelivery {     

     @Override
     void getSoftwareOptions() {
          ...
     }

}
```

- lSP를 잘 지키지 않은 사례다.

```java
public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    @Override
    public void setHeight(int height) {
        this.height = height;
        this.width = height;
    }
}
```

- abstract class로 만들어 LSP를 지켰다.

```java
public abstract class Shape {
    public abstract int getArea();
}
public class Rectangle extends Shape {
    protected int width;
    protected int height;
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    @Override
    public int getArea() {
        return width * height;
    }
}
public class Square extends Shape {
    private int side;
    public Square(int side) {
        this.side = side;
    }
    @Override
    public int getArea() {
        return side * side;
    }
}
```