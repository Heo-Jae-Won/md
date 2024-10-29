## <span style="color:#802548">_aggreagation constructor_</span>
- aggregation은 외부의 생성자를 통해서 param으로 field를 채우는 형식이다.
- Spring의 DI 형식이 이러한 형태를 사용한다.
- association이란 별도의 관계가 존재하지만, source code상에서는 aggregation과 동일하게 구현된다.

```java
// has a 관계
// 결합도
// 1. Aggregation (결합 관계) - 약한 소유권
// 외부(생성자)를 통해 param으로 받아오는 관계
// Spring에서 쓰는 DI의 개념은 Aggregation

class Car {
   //자동차가 engine을 포함하고 있기에 has a 관계.
   private Engine engine ;
   public Car(Engine engine) {
      this.engine = engine;
   }
   
   public void aseembleCar(Engine engine) {
      this.engine = engine;
   }
}

class Engine { }
```

- 더 구체적인 예시를 살펴보자.
- 아래는 Department가 Student를 field로 가진다.
- 외부에서 생성자로 주입해서 Student field를 채운다.
- 둘은 서로 독립하여 존재하기 때문에 Department 안에서 new로 만들지 않는 것이다.

```java
package chapter006.extend;
// Importing required classes
import java.util.ArrayList;
import java.util.List;


// Class 1
// Student class
class Student {

    // Attributes of Student
    private String studentName;
    private int studentId;

    // Constructor of Student class
    public Student(String studentName, int studentId)
    {
        this.studentName = studentName;
        this.studentId = studentId;
    }

    public int getstudentId() { 
      return studentId; 
    }

    public String getstudentName() {
      return studentName; 
    }
}

// Class 2
// Department class 
// Department class contains list of Students0
class Department1 {

    // Attributes of Department1 class
    private String deptName;
    private List<Student> students;

    // Constructor of Department1 class
    public Department1(String deptName, List<Student> students)
    {
        this.deptName = deptName;
        this.students = students;
    }

    public List<Student> getStudents() {
      return students; 
    }

    public void addStudent(Student student)
    {
        students.add(student);
    }
}

// Class 3
// Institute class
// Institute class contains the list of Department1s
class Institute {

    // Attributes of Institute
    private String instituteName;
    private List<Department1> Department1;

    // Constructor of Institute class
    public Institute(String instituteName,
                     List<Department1> Department1)
    {
        // This keyword refers to current instance itself
        this.instituteName = instituteName;
        this.Department1 = Department1;
    }

    public void addDepartment(Department1 department)
    {
       Department1.add(department);
    }

    // Method of Institute class
    // Counting total students in the institute
    public int getTotalStudentsInInstitute()
    {
        int noOfStudents = 0;
        List<Student> students = null;

        for (Department1 dept : Department1) {
            students = dept.getStudents();

            for (Student s : students) {
                noOfStudents++;
            }
        }
        return noOfStudents;
    }
}

// Class 4
// main class
public class Aggregation {

    // main driver method
    public static void main(String[] args)
    {
        // Creating independent Student objects
        Student s1 = new Student("Parul", 1);
        Student s2 = new Student("Sachin", 2);
        Student s3 = new Student("Priya", 1);
        Student s4 = new Student("Rahul", 2);

        // Creating an list of CSE Students
        List<Student> cse_students = new ArrayList<Student>();
        cse_students.add(s1);
        cse_students.add(s2);

        // Creating an initial list of EE Students
        List<Student> ee_students = new ArrayList<Student>();
        ee_students.add(s3);
        ee_students.add(s4);

        // Creating Department object with a Students list
        // using Aggregation (Department "has" students)
        Department1 CSE = new Department1("CSE", cse_students);
        Department1 EE = new Department1("EE", ee_students);

        // Creating an initial list of Departments
        List<Department1> departments = new ArrayList<Department1>();
        departments.add(CSE);
        departments.add(EE);

        // Creating an Institute object with Departments list
        // using Aggregation (Institute "has" Departments)
        Institute institute = new Institute("BITS", departments);

        // Display message for better readability
        System.out.print("Total students in institute: ");

        // Calling method to get total number of students
        // in the institute and printing on console
        System.out.print(
            institute.getTotalStudentsInInstitute());
    }
}
```

## <span style="color:#802548">_aggreagation setter_</span>

- 아래와 같이 setter를 통해 aggregation을 만들수도 있다.
- Bank는 Employees라는 field를 setter를 통해 구현한다.

```java
package chapter006.extend;
// Java Program to illustrate the

import java.util.HashSet;
import java.util.Set;



// Class 1
// Bank class
class Bank {

   // Attributes of bank
   private String bankName;
   private Set<Employee> employees;

   // Constructor of Bank class
   public Bank(String bankName) {
      this.bankName = bankName;
   }

   // Method of Bank class
   public String getBankName() {
      // Returning name of bank
      return this.bankName;
   }

   public void setEmployees(Set<Employee> employees) {
      this.employees = employees;
   }

   public Set<Employee> getEmployees() {
      return this.employees;
   }
}

// Class 2
// Employee class
class Employee {

   // Attributes of employee
   private String name;

   // Constructor of Employee class
   public Employee(String name) {
      // this keyword refers to current instance
      this.name = name;
   }

   // Method of Employee class
   public String getEmployeeName() {
      // returning the name of employee
      return this.name;
   }
}

// Class 3
// Aggregation between both the
// classes in main method
public class Aggregation {

   // Main driver method
   public static void main(String[] args) {
      // Creating Employee objects
      Employee emp1 = new Employee("Ridhi");
      Employee emp2 = new Employee("Vijay");

      // adding the employees to a set
      Set<Employee> employees = new HashSet<>();
      employees.add(emp1);
      employees.add(emp2);

      // Creating a Bank object
      Bank bank = new Bank("ICICI");

      // setting the employees for the Bank object
      bank.setEmployees(employees);

      // traversing and displaying the bank employees
      for (Employee emp : bank.getEmployees()) {
         System.out.println(emp.getEmployeeName() + " belongs to bank " + bank.getBankName());
      }
   }
}
```


## <span style="color:#802548">_Composition new_</span>
- composition은 외부의 생성자를 통해 field를 받지 않는다.
- 직접 class 안에서 field를 new로 호출해 생성한다.
- 이로써 강하게 묶이게 된다.

```java
// has a 관계
// 2. composition (구성 관계) - 강한 소유권
// 구성요소인 instance를 생성자가 아닌 field로써 new로 만들어 가질 때
class House {
   private Room rooms = new Room();
}
class Room {}
```

- 더 구체적인 예시는 아래와 같다.
- Company를 만들 때 Department가 new로 생성된다.
- 그렇게 되면 Company가 사라지면 Department도 같이 사라지게 된다.

```java
package chapter006.extend;

import java.util.ArrayList;
import java.util.List;


/**
 * composition은 field 자체도 사용하려는 class에서 new로 instance를 만들어 사용한다.
 * 그 어떠한 setter나 생성자를 사용하지 않는다.
 * 따라서 가장 결합도가 높다.
 */

//Class 1
//Department class
class Department {

   // Attributes of Department
   public String departmentName;

   // Constructor of Department class
   public Department(String departmentName) {
      this.departmentName = departmentName;
   }

   public String getDepartmentName() {
      return departmentName;
   }
}

//Class 2
//Company class
class Company {

   // Reference to refer to list of books
   private String companyName;
   private List<Department> departments;

   // Constructor of Company class
   public Company(String companyName) {
      this.companyName = companyName;
      //new로 Department field를 생성시켜버렸다. 따라서 composition
      this.departments = new ArrayList<Department>();
   }

   // Method
   // to add new Department to the Company
   public void addDepartment(Department department) {
      departments.add(department);
   }

   public List<Department> getDepartments() {
      return new ArrayList<>(departments);
   }

   // Method
   // to get total number of Departments in the Company
   public int getTotalDepartments() {
      return departments.size();
   }
}

//Class 3
//Main class
public class Composition {

   // Main driver method
   public static void main(String[] args) {
      // Creating a Company object
      Company techCompany = new Company("Tech Corp");

      techCompany.addDepartment(new Department("Engineering"));
      techCompany.addDepartment(new Department("Operations"));
      techCompany.addDepartment(new Department("Human Resources"));
      techCompany.addDepartment(new Department("Finance"));

      int d = techCompany.getTotalDepartments();
      System.out.println("Total Departments: " + d);

      System.out.println("Department names: ");
      for (Department dept : techCompany.getDepartments()) {
         System.out.println("- " + dept.getDepartmentName());
      }
   }
}
```

## <span style="color:#802548">_Spring DI는 composition이 아닌 이유_</span>

- Spring에서는 DI를 Composition이 아닌 association(소스코드 상으로 aggregation과 동일)로 구현한다.
- 그 이유는 뭘까?
	- lack of flexibility
	- 강한 coupling에 따른 소스코드 로직 변경의 어려움
	- Spring의 Junit을 이용한 test의 어려움


- 구체적인 예시를 보자.
- chatGPT가 짜준 예시다.
- Library는 field로 book을 가지는데, new로 이를 생성하고 있다.
- 그리고 그 안에서 직접 book을 만들어 내고 있다.
- 책 갯수가 20개가 넘으면 보고를 해서 국가의 지원을 받게 된다.

```java
class Book {
    private String title;

    public Book(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}

class Library {
    private List<Book> books;

    public Library() {
        books = new ArrayList<Book>();
        // Library creates its own books
        books.add(new Book("1984"));
        books.add(new Book("Brave New World"));
    }

    public List<Book> getBooks() {
        return books;
    }


    public void report() {
        if (books.size() > 20) {
            sendSMS();
        }
    }

    public void sendSms() {
        sysout("sms가 갔네요.")
    }
}
```

- 현재 이러한 Library를 이용해서 main에서 호출하면 아래와 같은 형태가 된다.
- book class를 만들어도 쓸 수가 없다. 매번 Library라는 class 안에 전부 넣어야 한다.
- 하지만 library instance마다 다른 book을 갖게하고 싶었다면? 절대로 그렇게 만들수가 없다.
- DB에는 각 도서관 별로 보유한 책을 넣어야 할탠데, 그게 불가능하다는 의미다.
- flexible 하지 못하다는 의미다.

```java
public static void main(String[] args){ 
	Library first = new Library(); //book 맘대로 못 넣음...
	Library second = new Library();
	Library thirde = new Library();
	Library fourth = new Library();
}
```

- 두번쨰로 특정 도서관들은 이제는 동시성을 고려해야 하게 바뀌었다. 
- 그 경우 ArrayList 구현체가 아니라 CopyOnWriteList나 SynchronizedList를 써야할 것이다.
- https://asuraiv.blogspot.com/2020/02/java-synchronizedlist-vs.html 참조
- 하지만 위의 경우에는, new로 직접 class type을 지정해 생성한다.
    - 따라서 Library 객체마다 같은 type을 생성하게 되어버린다.
    - 따라서 이를 고려할 필요가 없는 다른 구현체들은 속도에서 손해를 보게 된다.

```java
public static void main(String[] args){ 
	Library first = new Library(); // book의 type도 맘대로 지정 못함..
	Library second = new Library();
	Library thirde = new Library();
	Library fourth = new Library();
}
```

- 사실상 tight coupling과 lack of flexibility는 같이 가는 것이다.
- 세번째로 testing이 있다.
- testing이 불가능하다는 의미는, Spring의 Junit 강력한 unit testing이 불가능하다는 것이다
    - 만약 특정 갯수의 책 이상에서 특정한 이벤트를 발생시켜야 한다고 해보자.
    - 그런데 특정 갯수를 test 코드에서 정하지 못하고, Library class 내부에서 보유하고 있다.
    - 따라서 Library 내부에서 추가해줘야 한다. 그런데 이렇게 되면 특정 갯수 이하에서는 해당 이벤트가 발생하지 않는지 test할수가 없다.

```java
import static org.mockito.Mockito.*;
import org.junit.Test;

public class LibraryTest {
    @Test
    public void testEventOver20() {
        Library library = spy(new Library());
        assertEquals(2, library.getBooks().size()); // Works, but not isolated
        assertEquals("1984", library.getBooks().get(0).getTitle());

        library.report();
        verify(library, times(1)).report(); 
        //실제로 size가 20이 넘었을 때 해당 method가 call됐는지 확인
        // 문제는 갯수에 따라서 성공 실패를 보이게 해야 되는데, 그게 test 코드에서 해결되지 않고,
        // domain class인 library까지 test를 할 때마다 변경해야 하는 사항을 초래.
        // test 코드를 위해 본 코드를 바꿔야 하는 안 좋은 상황에 처하게 됨.
    }
}
```


- 아래처럼 association(aggregation과 source code 동일)로 사용하면 위의 문제가 모두 해결된다.
- Library의 field인 book을 생성자 혹은 setter로 값을 받게 바꾼다.

```java
class Library {
    private List<Book> books;

    public Library(List<Book> books) { // Dependency Injection
        this.books = books;
    }

    public List<Book> getBooks() {
        return books;
    }

    public void report() {
        if (books.size() > 20) {
            sendSMS();
        }
    }

    public void sendSms() {
        sysout("sms가 갔네요.")
    }
}
```

- 그럼 book을 library 별로 다르게 갖게 할 수 있다.
- library 별로 book 구현체의 속성을 다르게 할 수 있다.
- 필요하다면 똑같은 book을 쓰게할 수도 있다. instance를 재활용하는 것이다.

```java
// Main method to demonstrate usage
public class Main {
    public static void main(String[] args) {
        List<Book> sharedBooks = new ArrayList<>();
        sharedBooks.add(new Book("1984"));
        sharedBooks.add(new Book("Brave New World"));

        List<Book> conSharedBooks = new CopyOnWriteList<>();
        conSharedBooks.add(new Book("채식주의자"));
        conSharedBooks.add(new Book("하늘과 바람과 별과 시"));

        Library library1 = new Library(sharedBooks);    // Library 1
        Library library2 = new Library(conSharedBooks); // Library 2
        Library library3 = new Library(sharedBooks); // Library 3
    }
}
```

- testing도 쉽게 가능하다.
- mock 객체를 통해 갯수도 맘껏 조절할 수 있고, 내용도 조절할 수 있다.
- 만약 composition 방식으로 만들어져 있었다면, book의 mock객체를 넣을 수가 없다.
- 물론 concurrency를 고려한 unit testing은 사실 어렵다. 따라서 그냥 list로 가정하겠다.
- test코드로 인해 본 코드가 바뀌는 일이 일어나지 않는다. 내가 원하는대로 test할 수도 있다.

```java
import org.junit.Test;
import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

import java.util.List;

public class LibraryTest {
    @Test
    public void testGetBooksAndReport() {
        //given
        List<Book> firstBooks = mock(List.class);
        List<Book> secondBooks = mock(List.class);
        Library library1 = spy(new Library(firstBooks)); // Injecting the mock into Library
        Library library2 = spy(new Library(secondBooks)); // Injecting the mock into Library

        //when
        when(firstBooks.size()).thenReturn(2); // Specify that the size is 2
        when(firstBooks.get(0)).thenReturn(new Book("1984")); // First book is "1984"
        when(secondBooks.size()).thenReturn(21); // Specify that the size is 21
        when(secondBooks.get(0)).thenReturn(new Book("Brave New World")); // First book is "Brave New World"

        //then
        assertEquals(2, library1.getBooks().size()); // Verify size
        assertEquals("1984", library1.getBooks().get(0).getTitle()); // Verify title of first book
        assertEquals(21, library2.getBooks().size()); // Verify size
        assertEquals("Brave New World", library2.getBooks().get(0).getTitle()); // Verify title of first book

        library1.report();
        verify(library1, times(0)).sendSMS(); // Should not send SMS, size <= 20
        library2.report();
        verify(library2, times(1)).sendSMS(); // Should send SMS, size > 20
    }
}
```


## <span style="color:#802548">_dynamic binding과 polymorphism_</span>
- list type이든, arraylist type이든, linkedList type이든 param이 List로 선언되어 있다면, 다 넣을 수 있다.
- 따라서 add method에는 List를 포함한 sub class는 전부 parameter로 들어갈 수 있다.

```java
public class C001_extend {
   
   public static void main(String[] args) {
      
      // List는 부모. 부모를 parameter로 넣으면 그 아래 자식들은 param으로 전부 집어넣을 수 있다.
      List<String> list = new ArrayList<>();    
      ArrayList<String> arrayList = new ArrayList<String>();
      LinkedList<String> linkedList = new LinkedList<String>();
      add(list);
      add(arrayList);
      add(linkedList);
   }
   
   
   
   public static void add(List<String> list) {
      list.add("r");
   }
}
```

- 하지만 실제 수행되는 list.add()는 new로 생성한 instance의 method 주소값을 따라간다.
    - overriding을 하지 않은 extends, impl class가 부모의 method를 썼다면, 부모의 method 주소값을 그대로 가져온다.
    - 하지만 overriding이 되어있다면 instance의 overriding한 method가 수행된다는 의미다.
    - 이를 dynamic binding이라고 부른다. 다형성을 지원하기 위한 binding이다.



## <span style="color:#802548">_static binding과 strong type lang_</span>

- 그런데 method와 달리 변수는 앞의 선언된 참조타입을 참고한다.
- 따라서 arrayList type을 참조로 한 변수는 trimToSize()를 사용할 수 있다.
- 하지만 List type을 참조로 한 변수는 ArrayList type인데도 trimToSize()를 사용할 수가 없다.

```java
public class C001_extend {
   
   public static void main(String[] args) {
      
      // List는 부모. 부모를 parameter로 넣으면 그 아래 자식들은 param으로 전부 집어넣을 수 있다.
      List<String> list = new ArrayList<>();    
      ArrayList<String> arrayList = new ArrayList<String>();
      LinkedList<String> linkedList = new LinkedList<String>();
      
      arrayList.trimToSize(); 
      list.trimToSize();        //error
   }
}
```

- 이러한 이유는 변수는 참조 타입이 중요하기 때문이다.
    - 즉, 멤버변수는 컴파일 시점에 참조 타입을 기준으로 결정. 정적 바인딩이다.
    - 컴파일 시점에 참조 타입이 결정되면, 안전하게 변수의 사용 가능 범위를 제한하고 오류를 사전에 방지한다.
    - compile 시점에 문법적으로 해당 타입에 그러한 method가 존재하지 않으면, 오류를 뱉는다.
- compile 시점의 오류가 없음이 판단되면, 그 떄부턴 runtime으로 넘어간다.
    - runtime 시에는 method의 경우, 동적 바인딩으로 작동한다.
    - method가 실제 호출되는 경우에는, 참조가 아닌 생성된 instance를 기준으로 결정된다는 의미다.
    - 컴파일 시점이 아니라 런타임 시점에 어떤 class의 method를 호출할 지 결정하는 이유는 다형성 떄문이다.
        - 다형성을 위해 구현된 overriding 기능을 지원하기 위해 이러한 형태로 사용하는 것이다. 
        - 만약 부모가 아니라 구체적인 형태로 바꾸고 싶다면 instanceof를 써서 분기시킨다.
        - 명시적인 casting ( downcasting ) 을 실행해야 한다는 의미다.
        - 다만 instanceof는 기본적으로 OOP가 제대로 달성되지 못하고 있다는 신호다.
        - 해당 method가 과한 책임을 지고 있는지 확인할 필요가 있다.



## <span style="color:#802548">_abstract class와 template method_</span>

- 자유도를 낮추고 특정한 방식으로만 사용하게 제한하고 싶을 때?
- 그럴 때 abstract class를 만들고 이를 extends하게 강제하기도 한다.

```java
package chapter006.extend;

abstract class Car1 {
   
   private void startCar() {
      System.out.println("시동을 켭니다.");
   }
   
   // private으로 만들면 상속 자체가 불가. 
   // final로 만들면 상속은 되지만 overriding이 불가
   // 이렇게 만들면 안 됨.. drive()는 하위 class에 구현 책임을 맡길 것이기 때문.
   // 그래야 abstract class로 만든 이유가 있음.
   /*
    * public void drive() { System.out.println("운전을 합니다."); }
    */
   abstract public void drive();
   
   // 이렇게 만들면 안 됨.. drive()는 하위 class에 구현 책임을 맡길 것이기 때문.
   /*
    * public void stop() { System.out.println("멈춥니다"); }
    */
   abstract public void stop();
   
   private void turnOff() {
      System.out.println("시동을 끕니다.");
   }
   
   
   //run에 final이 붙었으니 overriding 불가.
   // startCar와 turnOff가 private이니 override 불가
   // 하위 class는 drive와 stop만 자기 맘대로 바꿀 수 있음.
   // 따라서 하위 클래스들은 일부의 method에만 자유도가 있음.
   // 이렇게 abstract class의 method를 사용해야만 하는 method를 template method라고 함.
   final public void run() {
      startCar();
      drive();
      stop();
      turnOff();
   }
}
```

- AICar와 ManualCar는 drive와 stop만 구현가능하다.
- 그 범위 내에서만 자유도를 가지고 바꿀 수 있는 것이다.

```java
class AICar extends Car1 {

   @Override
   public void drive() {
      System.out.println("알아서 AI가 운전시작합니다.");
   }

   @Override
   public void stop() {
      System.out.println("알아서 AI가 멈춥니다.");
   }
   
}

class ManualCar extends Car1 {

   // 부모로부터 받은 게 public이면 public 고정.
   // 부모로부터 받은 거보다 넓은 범위로만 access modifier 수정 가능
   // 가능한 현실적인 경우의 수는 protected --> public 정도임..
   @Override
   public void drive() {
      System.out.println("사람이 운전합니다.");
   }

   @Override
   public void stop() {
      System.out.println("브레이크로 정지합니다.");
   }
   
}
```

- 아래의 run() method는 abstract class인 Car1의 method가 호출된다.
- final로 overriding을 막았기 때문이다.
- 대신 그 안의 drive()와 stop()은 구현된 instance의 method가 호출된다.

```java
public class AbstractKlass {
   
   public static void main(String[] args) {
      Car1 aiCar = new AICar();
      aiCar.run();
      
      System.out.println();
      
      Car1 manualCar = new ManualCar();
      manualCar.run();
   }

}
```

