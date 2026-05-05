
## <span style="color:#802548">_OOP 적용 안하고 막 만들어보기_</span>
- interface, extends, abstract 등이 없이 만들어 보려면 아래와 같이 노가다를 거친다.

- novice class에 쓰는 name, profession, stat field는 사실 magician 등의 직업 class에서도 공유한다.

```java
public class Novice {
	String name;
	String profession;
	Stat1 stat;
	
	public Novice(String name) {
		this.name = name;
		this.profession = "초보자";
		this.stat = new Stat1();
	}

	public void doSkill() {
		System.out.println("아직 스킬이 없습니다");
	}

	public void doBasicAttack() {
		System.out.println("기본공격");
		
	}
	
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "[ID: " + this.name 
				+"(" + this.profession + "), stat: "
				+ "힘(" + this.stat.him + "), "
				+ "민(" + this.stat.min + "), "
				+ "지(" + this.stat.ji + ")]";
	}
	
}
```

- 하지만 부모를 extends 하지 않았기 때문에 여기도 그대로 써줘야 한다.
- 또한 자유 전직 이벤트 발생 시, 이미 전직한 class가 Magician으로 바뀌는 것이다.
	- 따라서 각 직업 별 생성자를 전부 미리 마련해둬야 한다.
	- 직업별 정보를 출력하는 toString도 부모에 있으면 부모걸 그대로 쓰면 편한데, 개별 class마다 구현해야 한다.
- skill, attack 또한 개별 class에 묶이게 된다.
	- 누가 오타를 내서 method를 dSkill로 만들면 Magician class는 타 class와 다르게 dSkill()이 되어버린다.


```java
public class Magician {
	String name;
	String profession;
	Stat1 stat;
	
	public Magician(Novice novice) {
		this.name = novice.name;
		this.profession = "마법사";
		this.stat = novice.stat;
	}
	
	public Magician(Knight knight) {
		this.name = knight.name;
		this.profession = "마법사";
		this.stat = knight.stat;
	}
	
	public Magician(Thief thief) {
		this.name = thief.name;
		this.profession = "마법사";
		this.stat = thief.stat;
	}

	public void doSkill() {
		System.out.println("메테오");
		
	}

	public void doBasicAttack() {
		System.out.println("매직 애로우 공격");
		
	}
	
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "[ID: " + this.name 
				+"(" + this.profession + "), stat: "
				+ "힘(" + this.stat.him + "), "
				+ "민(" + this.stat.min + "), "
				+ "지(" + this.stat.ji + ")]";
	}

}
```

- 사실상 똑같은 내용이지만 또 써준다.

```java
public class Knight {
	String name;
	String profession;
	Stat1 stat;
	
	public Knight(Novice novice) {
		this.name = novice.name;
		this.profession = "기사";
		this.stat = novice.stat;
	}
	
	public Knight(Magician magician) {
		this.name = magician.name;
		this.profession = "기사";
		this.stat = magician.stat;
	}
	
	public Knight(Thief thief) {
		this.name = thief.name;
		this.profession = "기사";
		this.stat = thief.stat;
	}
	
	public void doSkill() {
		System.out.println("쇼크스턴");
		
	}

	public void doBasicAttack() {
		// TODO Auto-generated method stub
		System.out.println("강력한 배쉬 공격");
		
	}
	
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "[ID: " + this.name 
				+"(" + this.profession + "), stat: "
				+ "힘(" + this.stat.him + "), "
				+ "민(" + this.stat.min + "), "
				+ "지(" + this.stat.ji + ")]";
	}

}
```

- 사실상 똑같은 내용이지만 또 써준다.

```java
public class Thief  {
	String name;
	String profession;
	Stat1 stat;
	
	public Thief(Novice novice) {
		this.name = novice.name;
		this.profession = "도적";
		this.stat = novice.stat;
	}

	public Thief(Knight knight) {
		this.name = knight.name;
		this.profession = "도적";
		this.stat = knight.stat;
	}
	
	public Thief(Magician magician) {
		this.name = magician.name;
		this.profession = "도적";
		this.stat = magician.stat;
	}
	
	public void doSkill() {
		System.out.println("사신의 발걸음");
		
	}

	public void doBasicAttack() {
		System.out.println("표창 던지기");
		
	}
	
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "[ID: " + this.name 
				+"(" + this.profession + "), stat: "
				+ "힘(" + this.stat.him + "), "
				+ "민(" + this.stat.min + "), "
				+ "지(" + this.stat.ji + ")]";
	}

}
```

- 활용할 때도 extends, interface 등을 고려하지 않고 만들었으니 다형성이 부족하다.
- 아래처럼 각 class 별로 전부 참조를 담는 변수를 만들어 null로 만들었다가 뺐다가 관리해야 한다.
- 또한 해당 class의 변수가 null인지 아닌지 확인하는 분기문을 통해 logic을 전개해야 한다.

```java
public static void main(String[] args) {
		Novice novice = null;
		Knight knight = null;
		Magician magician = null;
		Thief thief = null;
		try (Scanner scan = new Scanner(System.in)) {
			while(true) {
				GameUtil.printMenu();
				int menu = scan.nextInt();
				
				switch (menu) {
					case 1:
						System.out.println("캐릭터를 생성합니다.");
						System.out.print("사용하실 아이디를 입력해주세요.");
						String nickname = scan.next(); //nextLine쓰면 이상하게 씹힘.
						System.out.println("스탯을 부여합니다.");
						while (true) {
							novice = new Novice(nickname);
							knight = null;
							magician = null;
							thief = null;
							GameUtil.printStatInfo(novice);
							System.out.println("스탯을 다시 받으시겠습니까? (y/n)");
							String choice = scan.next(); //nextLine쓰면 이상하게 씹힘.
							if (choice.equals("N") || choice.equals("n")) {
								break;
							} 
						}
						System.out.println("현재 정보로 저장합니다.");
						break;
					case 2:
						if (novice != null) {
							System.out.println(novice.toString()); 
						} else if (knight != null) {
							System.out.println(knight.toString()); 
						} else if (magician != null) {
							System.out.println(magician.toString()); 
						} else if (thief != null) {
							System.out.println(thief.toString()); 
						} else {
							System.out.println("먼저 캐릭터를 생성해주세요");
						}
						break;
					case 3:
						if (knight != null || thief != null || magician != null) {
							System.out.println("이미 전직이 완료되었습니다.");
							break;
						}
						if( novice == null) {
							System.out.println("먼저 캐릭터를 생성해주세요");
							break;
						}
						GameUtil.printProfessionMenu();
						int professionMenu = scan.nextInt();
						switch (professionMenu) {
							case 1:
								System.out.println("기사로 전직합니다.");
								knight = new Knight(novice);
								novice = null;
								thief = null;
								magician = null;
								break;
							case 2:
								System.out.println("도적으로 전직합니다.");
								thief = new Thief(novice);
								novice = null;
								magician = null;
								knight = null;
								break;
							case 3:
								System.out.println("마법사로 전직합니다.");
								magician = new Magician(novice);
								novice = null;
								knight = null;
								thief = null;
								break;
							default:
								System.out.println("제대로된 거 쓰셈");
						}
						break;
					case 4:
						if (novice != null) {
							novice.doBasicAttack();
						} else if (knight != null) {
							knight.doBasicAttack();
						} else if (magician != null) {
							magician.doBasicAttack();
						} else if (thief != null) {
							thief.doBasicAttack();
						} else {
							System.out.println("먼저 캐릭터를 생성해주세요");
						}
						break;
					case 5:
						if (novice != null) {
							novice.doSkill();
						} else if (knight != null) {
							knight.doSkill();
						} else if (magician != null) {
							magician.doSkill();
						} else if (thief != null) {
							thief.doSkill();
						} else {
							System.out.println("먼저 캐릭터를 생성해주세요");
						}
						break;
					case 0:
						System.out.println("사용을 종료합니다.");
						System.exit(0);
					default:
						System.out.println("메뉴에 있는 번호를 선택해주세요");
				}
			}
		}
	}
```

## <span style="color:#802548">_interface_</span>
- 위와 같이 다형성을 전혀 고려하지 않고 만든 소스 코드를 refactoring해보자.
- interface로 skill을 쓰라는 hint를 주었는데, 곰곰이 생각해보면 이유가 있다.
   - 확장성을 위해서였다. 캐릭터 sub class에 skill, basicAttack을 주는 것도 가능할 것이다.
   - 하지만 일반몬스터, 보스 몬스터 등도 skill을 하고 attack을 한다.
   - 그 경우 skill, attack을 쓰려고 monster가 character class를 extends하는 것은 말이 안되는 일이다.
- 따라서 skill interface를 따로 만드는 것이다.

```java
public interface Skill {
	
	void doSkill();
	void doBasicAttack();

}
```

- 아래와 같은 형태로 Skill을 전부 impl하게 된다.

```java
public class Novice implements Skill {
	//
	//
	//
	@Override
	public void doSkill() {
		System.out.println("아직 스킬이 없습니다");
	}

	@Override
	public void doBasicAttack() {
		System.out.println("기본공격");
		
	}
};

public class Knight implements Skill {
	//
	//
	//
	@Override
	public void doSkill() {
		System.out.println("쇼크스턴");
		
	}

	@Override
	public void doBasicAttack() {
		// TODO Auto-generated method stub
		System.out.println("강력한 배쉬 공격");
		
	}
};

public class Magician implements Skill {
	//
	//
	//
	@Override
	public void doSkill() {
		System.out.println("메테오");
		
	}

	@Override
	public void doBasicAttack() {
		System.out.println("매직 애로우 공격");
		
	}
};

public class Thief implements Skill {
	//
	//
	//
	@Override
	public void doSkill() {
		System.out.println("사신의 발걸음");
		
	}

	@Override
	public void doBasicAttack() {
		System.out.println("표창 던지기");
		
	}
};

//아래는 필요하다면...
public class BossMonster implements Skill {

};
```

## <span style="color:#802548">_OOP에 따른 responsibility divide-->stat_</span>

- 우선 힘, 민, 지의 stat은 이것이 stat임을 확실하게 보여주기 위해 class로 만든다.
- 이것이 강타입인 java class의 장점이다. class 이름을 통해 어떤 역할을 하는 지 추측할 수 있다.

```java
public class Stat {
	int him;
	int min;
	int ji;

}
```

- stat의 총합이 반드시 15가 되어야 하는 조건이 있다.
- stat은 random 값이기에 random을 불러주고, do - while 문으로 15 이상이 될 떄까지 반복시킨다.

```java
public class Stat {
	int him;
	int min;
	int ji;
	
	Random random = new Random();
	
	public Stat() {
		do {
			this.him = random.nextInt(7) + 1;
			this.min = random.nextInt(7)+ 1;
			this.ji = random.nextInt(7)+ 1;
		} while (him + min + ji < 15);
	}
}
```

## <span style="color:#802548">_OOP에 따른 responsibility divide-->character_</span>

- 그런데 Knight, Magician, Thief, Novice 모두 Character다.
- Character로서의 stat, name, profession을 가진다.
- 따라서 공통 부모가 될 class를 만들어준다.
- Character를 생성할 떄, 실제로는 Character가 아닌 Novice, Knight, Magician, Thief class로 instance를 만들게 된다.

```java
public class Character implements Skill {
	String name;
	String profession;
	Stat stat;
	
}
```

- 따라서 실제 concrete class를 만들어주자.
- Novice는 누구나 반드시 거치게 되는 곳이다.
- 따라서 아래와 같이 이름을 받게 만들었다.
- 하지만 생각해보면, Novice가 name, stat을 초기화하는 게 아니다.
- character가 생성되면서 name, stat이 초기화되는 것이다.

```java
package example;

public class Novice extends Character implements Skill {
	
   public Novice(String name) {
		this.name = name;
		this.profession = "초보자";
		this.stat = new Stat();
	}

	@Override
	public void doSkill() {
		System.out.println("아직 스킬이 없습니다");
	}

	@Override
	public void doBasicAttack() {
		System.out.println("기본공격");
		
	}
	
}
```

- 따라서 Character class에 name을 받는 생성자를 만들어준다.
- stat 또한 Character를 만들 때 생성해준다.

```java
public class Character implements Skill {
	String name;
	String profession;
	Stat stat;

   public Character(String name) {
		this.name = name;
		this.stat = new Stat();
	}
	
}
```

- 그럼 Novice는 profession만 넣게 된다.
- 아까와 다르게 Character를 extends한 것을 볼 수 있다.
- 그렇게 되면 field를 중복 선언하지 않고 쓸 수 있다.
- 그 외에도 개념적으로 Novice는 Character의 sub class이기 때문이라도 extends를 한다.

```java
public class Novice extends Character implements Skill {
	
	public Novice(String name) {
		super(name);
		this.profession = "초보자";
	}

	@Override
	public void doSkill() {
		System.out.println("아직 스킬이 없습니다");
	}

	@Override
	public void doBasicAttack() {
		System.out.println("기본공격");
		
	}
	
}
```

- 다른 class들의 경우, super(name)을 호출할 수가 없다.
- 그럼 스탯이 초기화되어 버리기 때문이다.
- 따라서 아래처럼 profession만 바꾸고 나머지 값은 그대로 가져와야 한다.
- skill은 각 직업에 맞게 구현시켜준다.

```java
public class Knight extends Character implements Skill  {

	public Knight(Character character) {
		this.name = character.name;
		this.stat = character.stat;
		this.profession = "기사";
	}

	@Override
	public void doSkill() {
		System.out.println("쇼크스턴");
		
	}

	@Override
	public void doBasicAttack() {
		// TODO Auto-generated method stub
		System.out.println("강력한 배쉬 공격");
		
	}

}
```

- 그러나 오류가 난다.
- Character class에 name을 받는 parameter가 있는 constructor를 만들었다.
- implicit constructor가 사라지면서 Character()가 사라졌다.
- 그런데 자식 class 들은 무조건 부모의 생성자 초기화가 요구된다.
- 그렇다고 super(name)을 했다간 의도에 맞지 않는다. stat이 초기화 되어버린다.

```java
public class Knight extends Character implements Skill  {

	public Knight(Character character) {
		//super(); 가 늘 생략되어 있는 것. 부모의 기본생성자가 없다면 부모의 paramter 있는 생성자를 넣어야.. 어쨌든 부모의 초기화가 필요.
		this.name = character.name;
		this.stat = character.stat;
		this.profession = "기사";
	}

	@Override
	public void doSkill() {
		System.out.println("쇼크스턴");
		
	}

	@Override
	public void doBasicAttack() {
		// TODO Auto-generated method stub
		System.out.println("강력한 배쉬 공격");
		
	}

}
```

- 따라서 아래와 같이 Character class에 기본 생성자를 명시적으로 만들어준다.

```java
public class Character implements Skill {
	String name;
	String profession;
	Stat stat;

   public Character(String name) {
		this.name = name;
		this.stat = new Stat();
	}

	public Character() {

	}
	
}
```

- 물론 단지 field 중복을 줄이겠다고 extends를 선언하면 안된다.
- extends를 쓴다는 것은 긴밀한 관계로 결합됨을 의미하며, 그 외의 다른 class가 결합될 가능성을 잃는다.
- concrete class가 extends할 class는 1개 뿐이기 때문이다.
- 따라서 연관관계가 extends로 묶여야 할만한 이유가 있을 때 쓰자. 
- 그냥 코드를 줄이겠다고 쓰면 안 된다.

## <span style="color:#802548">_abstract class_</span>
- 처음에는 Character class를 abstract class로 만들지 않았다.
- 그런데 Character class로 만들어 전부 활용하려 했더니, 문제가 있었다.
   - Character는 concrete class라서 skill(), attack()을 구현하지 않으면 하위 class들도 쓰지 못하는 것이다.

```java
Character character = null;
//...
//...
//...
case 4:
   if( character == null) {
      System.out.println("먼저 캐릭터를 생성해주세요");
      break;
   }
   character.doBasicAttack(); // error. Character class는 concrete class인데, doBasicAttack()을 구현하지 않았음.
   break;
case 5:
   if( character == null) {
      System.out.println("먼저 캐릭터를 생성해주세요");
      break;
   }
   character.doSkill(); 	//  error. Character class는 concrete class인데, doBasicAttack()을 구현하지 않았음.
      break;
```

- Character는 실제로 new로 호출될 일이 없어 skill, attack의 내용이 없다.
	- 그런데 하위 class들이 쓰기 위해 구현하는 것은 바람직하지 않다.
	- 하지만 Character를 extends하는 class들이 skill()과 attack()을 활용하게 하려면 어쩔수 없이 구현해야 했다.

```java
public class Character implements Skill {
	String name;
	String profession;
	Stat stat;
	
	/**
	 * @param name
	 */
	public Character(String name) {
		this.name = name;
		this.profession = "초보자";
		this.stat = new Stat();
	}
	
	/**
	 * 이게 없으면 Character를 계승한 모든 class에서 constructor 에러가 난다.
	 * 자식 class는 무조건 super()를 호출하게끔 되어있기 때문이다.
	 */
	public Character() {
		
	}

   @Override
	public void doSkill() {
		// 내용이 없는 overriding
		
	}

	@Override
	public void doBasicAttack() {
		// 내용이 없는 overriding
		
	}
}
```

- 그러한 이유에 따라 Character clss를 abstract class로 변경했다.
- 이렇게 하면 굳이 Character에서 Skill interface를 구현할 필요가 없다.
- 구현 책임을 하위 concrete class에게 넘길 수 있다.
- 그러면서 동시에 Character class를 참조하는 변수를 만들어도 skill(), attack()을 호출할 수 있다.

```java
abstract public class Character implements Skill {
	String name;
	String profession;
	Stat stat;
	
	/**
	 * @param name
	 */
	public Character(String name) {
		this.name = name;
		this.stat = new Stat();
	}
	
	public Character() {
		
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "[ID: " + this.name 
				+"(" + this.profession + "), stat: "
				+ "힘(" + this.stat.him + "), "
				+ "민(" + this.stat.min + "), "
				+ "지(" + this.stat.ji + ")]";
	}
}
```

- 이제는 아래와 같이 사용해도 오류가 나지 않는다.
- 추상 클래스로 변경하여 다형성을 활용하였다.

```java
Character character = null;
case 4:
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		break;
	}
	character.doBasicAttack(); //ok
	break;
case 5:
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		break;
	}
	character.doSkill();  //ok
	break;
```

- 이젠 abstract class에 전부 의존하면 되므로, concrete class에 implements Skill도 전부 없앤다.
- 없애도 Character abstract class가 Skill을 implements하기 때문에 구현 책임은 concrete class인 Novice에 있다.
- 따라서 concrete class 단에 굳이 implements Skill을 쓸 필요가 없다.

```java
public class Novice extends Character {
	
	public Novice(String name) {
		super(name);
		this.profession = "초보자";
	}

	@Override
	public void doSkill() {
		System.out.println("아직 스킬이 없습니다");
	}

	@Override
	public void doBasicAttack() {
		System.out.println("기본공격");
		
	}
	
}

public class Knight extends Character  {

	public Knight(Character character) {
		super(character);
		this.profession = "기사";
	}

	@Override
	public void doSkill() {
		System.out.println("쇼크스턴");
		
	}

	@Override
	public void doBasicAttack() {
		// TODO Auto-generated method stub
		System.out.println("강력한 배쉬 공격");
		
	}

}

public class Thief extends Character {

	
	public Thief(Character character) {
		super(character);
		this.profession = "도적";
	}


	@Override
	public void doSkill() {
		System.out.println("사신의 발걸음");
		
	}

	@Override
	public void doBasicAttack() {
		System.out.println("표창 던지기");
		
	}

}

public class Magician extends Character {

	
	public Magician(Character character) {
		super(character);
		this.profession = "마법사";
	}

	@Override
	public void doSkill() {
		System.out.println("메테오");
		
	}

	@Override
	public void doBasicAttack() {
		System.out.println("매직 애로우 공격");
		
	}

}
```


## <span style="color:#802548">_polymorphism_</span>

- 위처럼 abstract class와 interface를 활용하여 다형성을 구현할 수 있다.
- 이는 util class를 만들 때 특히 도움이 된다.
- 만약, 부모 class 없이 모두 개별 concrete class로 만들었다면 해당 정보를 개별 method로 만들어서 써야할 것이다.
- 결국 같은 코드나 다름없는데, 노가다식의 overloading을 하게 되어버린다.

```java
public static void printStatInfo(Novice novice) {
		System.out.println(	
				"부여된 스탯 정보: "
				+ "힘[" + novice.stat.him + "], "
				+ "민[" + novice.stat.min + "], "
				+ "지[" + novice.stat.ji + "]"
		);
	}

public static void printStatInfo(Knight knight) {
	System.out.println(	
			"부여된 스탯 정보: "
			+ "힘[" + knight.stat.him + "], "
			+ "민[" + knight.stat.min + "], "
			+ "지[" + knight.stat.ji + "]"
	);
}

public static void printStatInfo(Magician magician) {
	System.out.println(	
			"부여된 스탯 정보: "
			+ "힘[" + novice.stat.him + "], "
			+ "민[" + novice.stat.min + "], "
			+ "지[" + novice.stat.ji + "]"
	);
}

public static void printStatInfo(Thief thief) {
	System.out.println(	
			"부여된 스탯 정보: "
			+ "힘[" + thief.stat.him + "], "
			+ "민[" + thief.stat.min + "], "
			+ "지[" + thief.stat.ji + "]"
	);
}
```

- 하지만 다형성을 잘 생각해서 abstract class로 만들었기에 아래와 같이 간단하게 해주면 된다.
- 굳이 Novice, Knight, Thief, Magician으로 parameter를 바꿔가며 overloading을 할 필요가 없다.
- Character로 parameter를 받아주면 그만이다.

```java
public static void printStatInfo(Character character) {
		System.out.println(	
				"부여된 스탯 정보: "
				+ "힘[" + character.stat.him + "], "
				+ "민[" + character.stat.min + "], "
				+ "지[" + character.stat.ji + "]"
		);
	}
```


- 메뉴 선택 시에도 마찬가지다.
- 전부 Character 객체로 통일해서 진행하면 된다.
- 매번 새로운 참조 타입을 담는 변수를 만들어 진행할 필요가 전혀 없다.

```java
Character character = null;
switch(menu) {
	case 1: //생성
		character = character = new Novice(nickname);
		break;
	case 2: //전직
		character = GameUtil.getProfession(character, professionMenu);
		break;
	case 3: //스킬
		character.doSkill();
		break;
	case 4: //기본공격
		character.doBasicAttack();
		break;
	case 5: //정보출력
		GameUtil.printStatInfo(character);
		break;

}
```



- 마찬가지로 character가 전직할 때도 Character class로 parameter를 두어 받으면 그만이다.

```java
public static Character getProfession(Character character, int professionMenu) {
	switch (professionMenu) {
		case 1:
			System.out.println("기사로 전직합니다.");
			character = new Knight(character);
			break;
		case 2:
			System.out.println("도적으로 전직합니다.");
			character = new Thief(character);
			break;
		case 3:
			System.out.println("마법사로 전직합니다.");
			character = new Magician(character);
			break;
		default:
			System.out.println("제대로된 거 쓰셈");
	}
	
	return character;
}
```

- 참조 떄문에 어쩔수 없이 여러가지 변수를 만들어, null을 의도적으로 통제할 필요가 없다.
- 소스코드 컨트롤이 훨씬 편해진다. 괜히 null을 잘 넣었나 뺐나 확인할 필요가 없다.
	- Character class를 참조하는 변수 하나만 만들면 끝이다.
	- toString()도 Character class에 규정해두고 그냥 쓰면 된다. field 값은 어차피 알아서 instance 값을 찾아간다.
	- printStatInfo()도 Character parameter로 쓰면 각 직업 class별로 parameter를 받게 overloading할 필요가 없다.
	- getProfession()도 Character parameter로 쓰면 각 직업 class별로 parameter를 받게 overloading할 필요가 없다.

```java
public class Game {
	
	public static void main(String[] args) {
		Character character = null;
		try (Scanner scan = new Scanner(System.in)) {
			while(true) {
				GameUtil.printMenu();
				int menu = scan.nextInt();
				
				switch (menu) {
					case 1:
						System.out.println("캐릭터를 생성합니다.");
						System.out.print("사용하실 아이디를 입력해주세요.");
						String nickname = scan.next(); //nextLine쓰면 이상하게 씹힘.
						System.out.println("스탯을 부여합니다.");
						while (true) {
							character = new Novice(nickname);
							GameUtil.printStatInfo(character);
							System.out.println("스탯을 다시 받으시겠습니까? (y/n)");
							String choice = scan.next(); //nextLine쓰면 이상하게 씹힘.
							if (choice.equals("N") || choice.equals("n")) {
								break;
							} 
						}
						System.out.println("현재 정보로 저장합니다.");
						break;
					case 2:
						if( character == null) {
							System.out.println("먼저 캐릭터를 생성해주세요");
							break;
						} else {
							System.out.println(character.toString()); 
						}
						break;
					case 3:
						if( character == null) {
							System.out.println("먼저 캐릭터를 생성해주세요");
							break;
						}
						//실제 만들 떄는 전직가능 flag 값을 만들어서 진행해야한다.
						//직업 군 내에서의 전직(전사 내 용기사, 아이언 등) or 직업군( 전사 -> 마법사)
						if (!(character instanceof Novice)) {
							System.out.println("이미 전직이 완료되었습니다.");
							break;
						}
						GameUtil.printProfessionMenu();
						int professionMenu = scan.nextInt();
						character = GameUtil.getProfession(character, professionMenu);
						break;
					case 4:
						if( character == null) {
							System.out.println("먼저 캐릭터를 생성해주세요");
							break;
						}
						character.doBasicAttack();
						break;
					case 5:
						if( character == null) {
							System.out.println("먼저 캐릭터를 생성해주세요");
							break;
						}
						character.doSkill();
						break;
					case 0:
						System.out.println("사용을 종료합니다.");
						System.exit(0);
					default:
						System.out.println("메뉴에 있는 번호를 선택해주세요");
				}
			}
		}
	}

}
```

- constructor도 공통으로 공유하게 쓰면 된다.
- 기존에는 각 개별 class의 경우, 아래같이 전부 character.name과 stat을 받아와서 직접 넣었다.

```java
public Knight(Character character) {
		this.name = character.name;
		this.stat = character.stat;
		this.profession = "기사";
	}

	public Magician(Character character) {
		this.name = character.name;
		this.stat = character.stat;
		this.profession = "마법사";
	}

	public Thief(Character character) {
		this.name = character.name;
		this.profession = "도적";
		this.stat = character.stat;
	}
```

- 하지만 super class에 미리 설정해놓으면 그럴 필요가 없다.
- 만들어진 걸 가져다 쓰면 되기 때문이다.
- 이렇게 부모 생성자를 호출하면, 필요없는 기본 생성자는 이제 지워도 된다.
	- 딱히 기능도 없이 그냥 호출을 위해 존재만 했던 것이니 없애는 것이 더 좋다.

```java
public Character(Character character) {
	this.name = character.name;
	this.stat = character.stat;
}

// public Character() {

// }

public Knight(Character character) {
	super(character);
	this.profession = "기사";
}

public Magician(Character character) {
	super(character);
	this.profession = "마법사";
}

public Thief(Character character) {
	super(character);
	this.profession = "도적";
}

```

## <span style="color:#802548">_method로 빼기_</span>

- service logic은 이름을 부여하여 어떤 action을 취하는지 알려주는 게 좋다.
- 따라서 이를 method로 만들어주자.

```java
public class Game {
	
	public static void main(String[] args) {
		Character character = null;
		try (Scanner scan = new Scanner(System.in)) {
			while(true) {
				GameUtil.printMenu();
				int menu = scan.nextInt();
				
				switch (menu) {
					case 1:
						character = GameUtil.createCharacter(scan,character);
						break;
					case 2:
						GameUtil.printCharacterInfo(character);
						break;
					case 3:
						character = GameUtil.completeProfession(scan, character);
						break;
					case 4:
						GameUtil.attack(character);
						break;
					case 5:
						GameUtil.skill(character);
						break;
					case 0:
						System.out.println("사용을 종료합니다.");
						System.exit(0);
					default:
						System.out.println("메뉴에 있는 번호를 선택해주세요");
				}
			}
		}
	}

}
```

- 이에 필요한 util method는 아래와 같다.
- character를 만들면 return해야 된다. void type으로는 main에서 값에 할당되지 못한다.
- 따라서 생성을 해도 생성을 하지 않은 것처럼 인식된다.

```java
public static Character createCharacter(Scanner scan, Character character) {
	System.out.println("캐릭터를 생성합니다.");
	System.out.print("사용하실 아이디를 입력해주세요.");
	String nickname = scan.next(); //nextLine쓰면 이상하게 씹힘.
	System.out.println("스탯을 부여합니다.");
	while (true) {
		character = new Novice(nickname);
		GameUtil.printStatInfo(character);
		System.out.println("스탯을 다시 받으시겠습니까? (y/n)");
		String choice = scan.next(); //nextLine쓰면 이상하게 씹힘.
		if (choice.equals("N") || choice.equals("n")) {
			break;
		} 
	}
	System.out.println("현재 정보로 저장합니다.");
	
	return character;
}
```

- early return은 굳이 null을 return할 필요가 없다.
- 그냥 가져온 param 그대로 character를 return해주면 된다.

```java
public static Character completeProfession(Scanner scan, Character character) {
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		return character;
	}
	//실제 만들 떄는 전직가능 flag 값을 만들어서 진행해야한다.
	//직업 군 내에서의 전직(전사 내 용기사, 아이언 등) or 직업군( 전사 -> 마법사)
	if (!(character instanceof Novice)) {
		System.out.println("이미 전직이 완료되었습니다.");
		return character;
	}
	GameUtil.printProfessionMenu();
	int professionMenu = scan.nextInt();
	character = GameUtil.getProfession(character, professionMenu);
	
	return character;
}
```

- 나머지는 void이므로 early return 시 그냥 return해주자.

```java
public static void printCharacterInfo( Character character) {
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		return;
	} 
	System.out.println(character.toString()); 
}


public static void attack(Character character) {
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		return;
	}
	character.doBasicAttack();
}

public static void skill(Character character) {
	if( character == null) {
		System.out.println("먼저 캐릭터를 생성해주세요");
		return;
	}
	character.doSkill();
}
```

## <span style="color:#802548">_abstract class의 예제_</span>

- excel을 만들 때는 아래와 같이 excel library를 활용하는 경우가 흔하다.
- 그 때 excel의 기본 font, style 등은 정해져 있지만, 몇 행 몇 열인지는 domain마다 정해야 하는 경우가 있다.
- 그럴때 abstract class를 활용한다.
	- createBody만 concrete class에서 구현해준다.
	- createBody를 구현하는 데 필요한 것들만 protected access modifier를 붙여준다.
	- 나머지 private method는 전부 font, color, sheet, header, bodycellstyle 등 공통 요소를 만드는 것들이다.
		- 해당 private method들은 모두 abstract class인 PMSCreateExcel 생성자에서만 쓰인다.

```java
public abstract class PMSCreateExcel<T> extends CreateExcel {

	/* 상수 START */
	private static final String SHEET_NAME = "엑셀다운로드";
	private static final String FONT_NAME = "Malgun Gothic";
	private static final short FONT_SIZE = 9;
	private static final XSSFColor FONT_COLOR = rgbToXSSFColor(127, 127, 127);
	private static final XSSFColor HEADER_BACKGROUND_COLOR = rgbToXSSFColor(231, 230, 230);
	private static final XSSFColor HEADER_BORDER_COLOR = rgbToXSSFColor(191, 191, 191);
	private static final XSSFColor BODY_BORDER_COLOR = rgbToXSSFColor(0, 0, 0);
	private static final HorizontalAlignment HEADER_HORIZONTAL_ALIGNMENT = HorizontalAlignment.CENTER;
	private static final VerticalAlignment HEADER_VERTICAL_ALIGNMENT = VerticalAlignment.CENTER;
	private static final VerticalAlignment BODY_VERTICAL_ALIGNMENT = VerticalAlignment.CENTER;
	private static final BorderStyle BORDER_STYLE = BorderStyle.THIN;

	private final ExcelColumn[] COLUMN_LIST;
	/* 상수 END */

	/* 파일 준비 START */
	private final XSSFWorkbook workbook;
	private final XSSFSheet sheet;

	private XSSFFont headerFont;
	private XSSFFont bodyFont;
	private XSSFCellStyle headerCellStyle;
	private XSSFCellStyle bodyCellStyleCenter;
	private XSSFCellStyle bodyCellStyleLeft;
	private XSSFCellStyle emptyCellStyle;

	private int rowNo = 0;
	private int colNo = 0;
	private XSSFRow xssfRow;

	protected PMSCreateExcel(ExcelColumn[] columnList) {
		this.COLUMN_LIST = columnList;

		this.workbook = new XSSFWorkbook();
		this.sheet = createSheet();
		this.headerFont = createFont(true);
		this.bodyFont = createFont(false);
		this.headerCellStyle = createHeaderCellStyle();
		this.bodyCellStyleCenter = createBodyCellStyle(HorizontalAlignment.CENTER);
		this.bodyCellStyleLeft = createBodyCellStyle(HorizontalAlignment.LEFT);
		this.emptyCellStyle = workbook.createCellStyle();
	}

	private XSSFSheet createSheet() {
		XSSFSheet sheet = workbook.createSheet(SHEET_NAME);

		for (int i = 0; i < COLUMN_LIST.length; i++) {
			int width = COLUMN_LIST[i].getWidth();
			excelSetColumnWidth(sheet, i, width);
		}

		return sheet;
	}

	private XSSFFont createFont(boolean isBold) {
		XSSFFont font = workbook.createFont();
		font.setFontName(FONT_NAME);
		font.setFontHeightInPoints(FONT_SIZE);
		font.setColor(FONT_COLOR);
		font.setBold(isBold);

		return font;
	}

	private XSSFCellStyle createCellStyle() {
		return workbook.createCellStyle();
	}

	private XSSFCellStyle createHeaderCellStyle() {
		XSSFCellStyle headerCellStyle = createCellStyle();
		headerCellStyle.setFont(headerFont);
		headerCellStyle.setAlignment(HEADER_HORIZONTAL_ALIGNMENT);
		headerCellStyle.setVerticalAlignment(HEADER_VERTICAL_ALIGNMENT);
		headerCellStyle.setFillForegroundColor(HEADER_BACKGROUND_COLOR);
		headerCellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
		headerCellStyle.setBorderColor(BorderSide.LEFT, HEADER_BORDER_COLOR);
		headerCellStyle.setBorderColor(BorderSide.RIGHT, HEADER_BORDER_COLOR);
		headerCellStyle.setBorderLeft(BORDER_STYLE);
		headerCellStyle.setBorderRight(BORDER_STYLE);

		return headerCellStyle;
	}

	private XSSFCellStyle createBodyCellStyle(HorizontalAlignment horizontalAlignment) {
		XSSFCellStyle bodyCellStyle = createCellStyle();
		bodyCellStyle.setFont(bodyFont);
		bodyCellStyle.setAlignment(horizontalAlignment);
		bodyCellStyle.setVerticalAlignment(BODY_VERTICAL_ALIGNMENT);
		bodyCellStyle.setBorderColor(BorderSide.LEFT, BODY_BORDER_COLOR);
		bodyCellStyle.setBorderColor(BorderSide.RIGHT, BODY_BORDER_COLOR);
		bodyCellStyle.setBorderColor(BorderSide.TOP, BODY_BORDER_COLOR);
		bodyCellStyle.setBorderColor(BorderSide.BOTTOM, BODY_BORDER_COLOR);
		bodyCellStyle.setBorderLeft(BORDER_STYLE);
		bodyCellStyle.setBorderRight(BORDER_STYLE);
		bodyCellStyle.setBorderTop(BORDER_STYLE);
		bodyCellStyle.setBorderBottom(BORDER_STYLE);

		return bodyCellStyle;
	}
	/* 파일 준비 END */

	/* 파일 데이터 입력 START */
	/** 파일 생성 **/
	public Workbook create(List<T> dataList) {
		createHeader();
		createBody(dataList);

		return workbook;
	}

	/** 파일 생성(헤더) **/
	private void createHeader() {
		newRow();

		for (ExcelColumn column : COLUMN_LIST) {
			String headerText = column.getName();
			createHeaderCell(headerText);
		}

		for (int i = 0; i < 3; i++) {
			createEmptyCell();
		}
	}

	/** 파일 생성(바디) **/
	protected abstract void createBody(List<T> assetList);
	/* 파일 데이터 입력 END */

	/* 유틸리티 함수 START */
	protected void newRow() {
		xssfRow = sheet.createRow(rowNo++);
		colNo = 0;
	}

	private void createCell(String value, XSSFCellStyle cellStyle) {
		excelAddCell(value, cellStyle, colNo++, xssfRow);
	}

	private void createHeaderCell(String value) {
		createCell(value, headerCellStyle);
	}

	protected void createBodyCellCenter(String value) {
		createCell(value, bodyCellStyleCenter);
	}

	protected void createBodyCellLeft(String value) {
		createCell(value, bodyCellStyleLeft);
	}

	private void createEmptyCell() {
		createCell("", emptyCellStyle);
	}

	protected String dateText(Date date) {
		return date == null ? "" : DateUtil.dateToString(date);
	}
	/* 유틸리티 함수 END */

}
```

- abstract class인 PMSCreateExcel class도 CreateExcel을 extends하고 있다.
- CreateExcel은 Excel을 만드는 데 도움이 되는 Util class다.
- 따라서 사실 extends가 아니라, static method로 생성하여 가져오는 게 맞다.
- 그리고 실제로 static method로 선언되어 있다.

```java
public class CreateExcel {

	/** ExcelColumn 클래스 생성 **/
	protected static ExcelColumn col(String name, int width) {
		return new ExcelColumn(name, width);
	}

	/** 엑셀 로우에 셀 추가 **/
	protected static void excelAddCell(String value, CellStyle style, int columnIndex, XSSFRow row) {
		XSSFCell xssfCell = row.createCell(columnIndex);
		xssfCell.setCellStyle(style);
		xssfCell.setCellValue(value);
	}

	/** 엑셀 컬럼 너비 설정 **/
	protected static void excelSetColumnWidth(XSSFSheet sheet, int columnIndex, int charLength) {
		final int WIDTH_PER_CHAR = 256;
		final int PADDING = 0;
		final int MAX_CHAR = 255;

		int width = Math.min((charLength * WIDTH_PER_CHAR) + PADDING, MAX_CHAR * WIDTH_PER_CHAR);
		sheet.setColumnWidth(columnIndex, width);
	}

	/** RGB -> XSSFColor **/
	protected static XSSFColor rgbToXSSFColor(int r, int g, int b) {
		Color color = new Color(r, g, b);
		IndexedColorMap colorMap = new DefaultIndexedColorMap();

		return new XSSFColor(color, colorMap);
	}

}
```

- 그렇다면 CreateExcel을 extends를 할 필요가 없다.
- 아래처럼 static import를 하면 된다. static import는 class를 import 할 수 없다. 개별 field나 method 단위로 해주자.

```java
import static xxx.asset.service.excel.CreateExcel.excelAddCell;
import static xxx.asset.service.excel.CreateExcel.excelSetColumnWidth;
import static xxx.asset.service.excel.CreateExcel.rgbToXSSFColor;

public abstract class PMSCreateExcel<T> {


}
```

- 실제 자산신청서 양식은 아래와 같이 파일을 생성한다.
- 어떤 양식으로 쓸지 다르기 때문에 createBody만 abstract method로 하여 overriding하는 것이다.
- 다만 제작자가 override를 안 달아놓았다.. 달아줘야 된다.

```java
public class RentApplicationDetailCreateExcel extends PMSCreateExcel<RentAssetDTO> {

	private static final ExcelColumn[] COLUMN_LIST = { col("자산 코드", 10), col("자산 코드(구)", 11), col("품목", 12),
			col("모델명", 35), col("모델번호", 15), col("일련번호", 20), col("추가 정보", 16), col("사용자", 13), col("대여일", 13),
			col("대여장소", 13), col("반납일", 13), col("반납장소", 13), col("신청상태", 13), col("비고", 11) };

	public RentApplicationDetailCreateExcel() {
		super(COLUMN_LIST);
	}

	/** 파일 생성(바디) **/
	//@override 이거 override 안달아주면 안되는데...
	protected void createBody(List<RentAssetDTO> rentAssetList) {
		for (RentAssetDTO asset : rentAssetList) {

			log.info("asset code : " + asset.getAssetCode());

			newRow();

			createBodyCellCenter(asset.getAssetCode());
			createBodyCellCenter(asset.getAssetCodeOld());
			createBodyCellCenter(AssetItemCodeEnum.fromCommonCode(asset.getAssetItemCode()).getAssetCodeNm());
			createBodyCellLeft(asset.getModelName());
			createBodyCellLeft(asset.getModelNo());
			createBodyCellLeft(asset.getSerialNo());
			createBodyCellLeft(asset.getAssetRemark());
			createBodyCellLeft(asset.getUser().getUserNm());
			createBodyCellCenter(asset.getRentDate());

			createBodyCellLeft(asset.getRentPlace());
			createBodyCellCenter(asset.getReturnDate());
			createBodyCellLeft(asset.getReturnPlace());

			String applyStateCode = asset.getApplyStateCode();
			String applyState = "";
			if (applyStateCode != null) {
				applyState = ApplState.fromCommonCode(applyStateCode).getDisplayName();
			}

			createBodyCellCenter(applyState);
			createBodyCellLeft(asset.getRemark());

		}
	}

}
```


- 자산신청이 아닌 자산 그 자체 관리의 경우는 양식이 또 다르다.
- 따라서 createBody가 다른 식으로 구현된다.

```java
public class AssetServiceCreateExcelAllAssets extends PMSCreateExcel<Asset> {

	private static final ExcelColumn[] COLUMN_LIST = { col("자산 코드", 10), col("자산 코드(구)", 10), col("품목", 11),
			col("제조사", 7), col("모델명", 34), col("모델번호", 14), col("일련번호", 19), col("OS버전", 25), col("CPU", 23),
			col("RAM", 12), col("SSD 용량", 11), col("그래픽", 18), col("제조시기", 7), col("화면크기(인치)", 20), col("최대 해상도", 20),
			col("추가 정보", 8), col("자산위치", 18), col("자산상태", 7), col("대여상태", 12), col("사용자", 8), col("대여일", 10),
			col("반납일", 10), col("신청상태", 8), col("등록자", 7), col("등록일", 9), col("수정자", 7), col("수정일", 9) };

	public AssetServiceCreateExcelAllAssets() {
		super(COLUMN_LIST);
	}

	/** 파일 생성(바디) **/
	protected void createBody(List<Asset> assetList) {
		for (Asset asset : assetList) {
			newRow();

			createBodyCellCenter(asset.getAssetCode());
			createBodyCellCenter(asset.getAssetCodeOld());
			createBodyCellCenter(asset.getAssetItem().getDisplayName());
			createBodyCellCenter(asset.getAssetMnfcCo());
			createBodyCellLeft(asset.getAssetModelNm());
			createBodyCellLeft(asset.getAssetModelNo());
			createBodyCellLeft(asset.getAssetSerNo());
			createBodyCellLeft(asset.getAssetOSVer());
			createBodyCellLeft(asset.getAssetCPU());
			createBodyCellLeft(asset.getAssetRAM());
			createBodyCellLeft(asset.getAssetSSD());
			createBodyCellLeft(asset.getAssetGraphic());
			createBodyCellLeft(asset.getAssetMnfcDt());
			createBodyCellLeft(asset.getAssetScreenSize());
			createBodyCellLeft(asset.getAssetMaxRes());
			createBodyCellLeft(asset.getAssetRemark());
			createBodyCellLeft(asset.getAssetCurPlace());
			createBodyCellCenter(asset.getAssetState().getDisplayName());

			AssetRentState assetRentState = asset.getAssetRentState();
			createBodyCellCenter(assetRentState.getRentState().getDisplayName());

			String userNm = "";
			String rentDtStr = "";
			String returnDtStr = "";
			String applStateStr = "";

			// 231117 XXX 팀장님 요청. excel의 대여정보 기준을 대여이력에서 현재 자산의 상태를 기준으로 변경
			RentState rentState = assetRentState.getRentState();
			if (rentState == RentState.RENT_APPLY || rentState == RentState.RENTED || rentState == RentState.RETURN_APPLY) {
				User lastUser = assetRentState.getApplAsset().getUserSeq();

				userNm = lastUser.getUserNm();
				rentDtStr = dateText(assetRentState.getApplAsset().getRentDt());
				returnDtStr = dateText(assetRentState.getApplAsset().getReturnDt());

				applStateStr = rentState.getDisplayName();
			}

			createBodyCellLeft(userNm);
			createBodyCellCenter(rentDtStr);
			createBodyCellCenter(returnDtStr);
			createBodyCellCenter(applStateStr);

			createBodyCellLeft(asset.getRegUser().getUserNm());
			createBodyCellCenter(dateText(asset.getRegDt()));

			User updateUser = asset.getUpdateUser();
			String updateUserStr = updateUser == null ? "" : updateUser.getUserNm();
			createBodyCellLeft(updateUserStr);
			createBodyCellCenter(dateText(asset.getUpdateDt()));
		}
	}

}
```



### 객체 나누기 코드 사례
```java
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu");
    private Menu menu2 = new Menu("menu");
    private Button button1 = new Menu("button1");
    
    private String currentMenu = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
        //button2.setonClickListener(this);
    }

    public void clicked(Component eventSource) {
        if ("menu1".equals(eventSource.getId())) {
            changeUIToMenu1();
        } else if ("menu2".equals(eventSource.getId())) {
            changeUIToMenu2();
        } else if ("button1".equals(eventSource.getId())) {
            if (currentMenu == null) {
                return;
            }
            if ("menu1".equals(currentMenu)) {
                processButton1WhenMenu1();
            } else if ("menu2".equals(currentMenu)) {
                processButton1WhenMenu2();
            }
        }/* else if (eventSource.getId().equals("button2")) {
            if (currentMenu == null) {
                return;
            }
            if (currentMenu.equals("menu1")) {
                processButton1WhenMenu1();
            } else if (currentMenu.equals("menu2")) {
                processButton1WhenMenu2();
            } */
    }


    private void changeUIToMenu1() {
        currentMenu = "menu1";
        sysout("메뉴1 화면으로 전환");
    }

    private void changeUIToMenu2() {
        currentMenu = "menu2";
        sysout("메뉴2 화면으로 전환");
    }

    private void processButton1WhenMenu1() {
        sysout("메뉴1 화면의 버튼1 처리")
    }

    private void processButton1WhenMenu2() {
        sysout("메뉴2 화면의 버튼1 처리")
    }

    /*private void processButton2WhenMenu1() {
        sysout("메뉴1 화면의 버튼2 처리")
    }

    private void processButton2WhenMenu2() {
        sysout("메뉴2 화면의 버튼2 처리")
    }*/

}
```

- interface를 활용하여 코드를 나눈다.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
}

public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
}
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    public void clicked(Component eventSource) {
        String sourceId = eventSource.getId();
        if ("menu1".equals(sourceId)) {
            currentScreen = new Menu1ScreenUI();
            currentScreen.show();
        } else if ("menu2".equals(sourceId)) {
            currentSCreen = new Menu2SCreenUI();
            currentScreen.show();
        } else if ("button1".equals(sourceId)) {
            if(currentScreen == null) {
                return;
            }
            currentScreen.handleButton1Click();
        }
    }
}
```

- 버튼 클릭 처리 코드와 메뉴 클릭 처리 코드의 목적이 다르므로 분리한다.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
}

public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            }

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            }
        }
    }
}
```

- 버튼2가 추가된다.

- 맨위와 비교하면 정말 간단하게 추가되었음을 알 수 있다.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
    public void handleButton2Click();
}
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
    /*public void handleButton2Click() {
        sysout("메뉴1 화면의 버튼2 처리");
    }*/
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
    /*public void handleButton2Click() {
        sysout("메뉴2 화면의 버튼2 처리");
    }*/
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            }

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            } /*else if(sourceId.equals("button2")) {
                currentScreen.handleButton2Click();
            }*/
        }
    }
}
```

- 메뉴3가 추가되었다고 해보자.
- 메뉴3가 추가되도 이제 버튼에 관한 코드는 건들지 않아도 된다.

- Menu3ScreenUI class만 만들고, Application 생성자에 menu3 관련 clickListener를 달아준다.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
    public void handleButton2Click();
}
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴1 화면의 버튼2 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴2 화면의 버튼2 처리");
    }
}

public class Menu3ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴3 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴3 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴3 화면의 버튼2 처리");
    }
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        //menu3.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            } /*else if (eventSource.getId().equals("menu3")) {
                currentScreen = new Menu3ScreenUI();
            }*/

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            } /*else if(sourceId.equals("button2")) {
                currentScreen.handleButton2Click();
            }*/
        }
    }
}
```

### 프로시저의 단점
- 절차지향이란 말은 순서에 따른 프로그래밍이라는 말이 아니다. 프로시저를 이용한 프로그래밍 기법을 의미한다.
- 프로시저는 데이터를 공유하는 방식으로 만들어지기 때문에 데이터를 중심으로 구현된다.
- 프로시저의 단점은 요구사항이 변경되면서 예상치 못한 상태가 추가될 때 드러난다.

- boolean 타입의 이름은 isOn이라는 data가 있다. 아래와 같은 상태를 지닌다.
```
on
off
대기중 상태 추가해달라 ---> 이 데이터 사용하는 모든 프로시저 수정
```

- 프로시저의 최악의 단점은 데이터가 서로 다른 의미로 사용되는 경우에 드러난다.
- 서비스 만료일 데이터가 null일 때 오류로 처리하게 만료 확인 프로시저를 만들었다고 하자.
- 그런데 회원 정보 수정 프로시저에서 서비스를 무한정 사용한다는 의미로 서비스 만료일 데이터 값을 null로 설정하게 되면?
- 이전 프로시저들이 모두 오류가 발생하게 된다. null은 오류로 처리하기 때문이다.
- 소리 크기 제어 객체가 있다고 해보자. 해당 객체는 아래 기능을 제공한다.
```
소리 크기 증가
소리 크기 감소
음 소거
```


### 객체의 operation

- 객체가 내부적으로 소리 크기를 어떤 데이터 타입 값으로 보관하는 지는 중요하지 않다. 세 개의 기능을 제공한다는 게 중요하다.
- 객체의 기능을 사용한다는 것은 객체의 operation을 사용한다는 의미고, 그러려면 사용법을 알아야 한다.
- operation의 사용법은 기능 식별 이름, 파라미터와 파라미터 타입, 결과 값을 합쳐 signature라고 한다.
```
method명 - 기능식별이름
parameter명 - parameter
결과값 - return
```
- 이런 객체가 제공하는 모든 operation을 객체의 인터페이스라고 한다.
- 이 interface는 Java 내의 interface가 아니다. 객체를 사용하기 위한 규칙이나 명세라고 생각해야 한다.
- operation의 실행을 요청하는 것을 메시지를 보낸다고 표현한다. 메서드를 호출하는 것이 메시지를 보내는 것이다.

```java
FileInpuStream is = new FileInputStream(fileName);
byte[] data = new byte[512];
int readBytes = is.read(data); // read가 메시지를 전송한 것이다. 즉 메소드 호출.
```

- 객체의 책임이 작아져야 절차지향 방식을 피할 수가 있다.
- 전부 서비스를 나누면 장점은 다음과 같다.
- 파일을 읽어오는 방법을 변경해야 한다면 파일 읽기 책임을 가진 객체의 코드만 수정된다.
- 암호화 알고리즘을 변경해야 하면, byte 암호화 객체의 코드만 수정된다.
- 즉 다른 class에 미치는 영향도가 현격하게 줄어들어 유지보수에 유리하다.


### 의존
- 한 객체가 다른 객체를 생성하거나, 다른 객체의 메서드를 호출할 때, parameter로 전달받을 때, 의존이라고 한다.

```java
public class FloWController {
    private String fileName;

    public FlowController(String fileName) {
        this.fileName = fileName;
    }
    public void process() {
        FileDataReader reader = new FileDataReader(fileName);
        byte[] plainBytes = reader.read();

        ByteEncryptor encryptor = new ByteEncryptor();         //객체 생성
        byte[] encryptedBytes = encryptor.encrypt(plainBytes); //메서드 호출
    }
}

public void process(ByteEncryptor encryptor) {
    //전달받은 파라미터를 사용할 가능성이 높다.
}
```

- 의존의 문제는 의존하는 대상의 코드가 바뀌면 자신의 코드도 바뀔 수 있다는 점이다.
- 아래와 같은 코드가 있다고 해보자. 아래와 같은 코드에서 로그를 남겨달라는 요구가 추가되었다고 해보자.
```java
public class Authenticator {
    public boolean authenticate(String id, String password) {
        Member m = findMemberById(id);
        if  (m == null) {
            return false;
        }

        return m.equalPassword(password);
    }
}

public class AuthenticationHandler {
    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator(); //객체 생성. 의존
        if (auth.authenticate(inputId, inputPassword)) {

        } else {

        }
    }
}
```

- 따라서 handler를 Exception을 throw하게 바꿨다.
```java
public class AuthenticationHandler {
    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator(); //객체 생성. 의존
        try {
            auth.authenticate(inputId, inputPassword);
        } catch(MemberNotFoundException e) {
            logService.writerLog(e);
            throw e;
        } catch(InvalidPasswordException e) {
            logService.writerLog(e);
            throw e;
        }
    }
}
```
- 그럼 아래와 같이 Authenticator도 코드를 바꿔줘야 한다.
```java
public class Authenticator throws MemberNotFountException, InvalidPasswordException {
    public void authenticate(String id, String password) {
        Member m = findMemberById(id);
        if (m == null) {
            throw new MemberNotFountException();
        }

        if(!m.equalPassword(password)) {
            throw new InvalidPasswordException();
        }
    }
}
```
- 객체지향의 의존이라는 것이 늘 장점만 있는 것은 아니다.

### encapsulation
- 회원 만료 처리를 하는 코드가 아래와 같다고 해보자.
```java
@Getter
public class Member {
    private Date expiryDate;
    private boolean male;
}
.
.
.

if (member.getExpiryDate() != null &&
        member.getExpiryDate().getDate() < System.currentTimeMills()) {
            //만료됐을 때 처리
}
```

- 그런데 서비스를 운영하다가 여성회원은 만료 기간이 지나도 30일 간은 서비스를 사용하게 정책이 변경되었다.
- 그럼 만료가 사용되는 코드를 전부 아래와 같이수정해줘야 한다.
```java
long day30 = 1000 * 60 * 60 * 24 * 30;
if( (member.isMale && member.getExpiryDate ! = null && member.getExpiryDate().getDate() < System.currentTimeMills()) ||
    (!member.isMale() && member.getExpiryDate != null && member.getExpiryDate().getDate() < System.currentTimeMills() - day30)) {

    }
```

- 만료 처리 로직을 캡슐화해준다면 전부 수정하지 않아도 된다.
- 해당 class 내의 로직만 수정해주면 나머지는 알아서 적용된다.
- 아래와 같이 기존 만료 로직이 있다고 해보자.
```java
public class Member {
    private Date expiryDate;
    private boolean male;

    public boolean isExpired() {
        return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills();
    }
}

if(member.isExpired) {

}
```

- 기존 로직을 아래와 같이 바꿨다.
- 클래스에 캡슐화해놨기 때문에 서비스의 모든 로직을 수정할 필요가 없다.
```java
public class Member {
    private static final long DAY30 =  1000 * 60 * 60 * 24 * 30;
    private Date expiryDate;
    private boolean male;

    public boolean isExpired() {
        if (male) {
            return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills();
        }

        return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills() - DAY30;
    }
}

if(member.isExpired()) {

}
```

- 이와 같이 캡슐화의 유명한 원칙으로 두 가지가 있다.

```
Tell, Don't ask
Law of Demeter
```
- 첫째 원칙은 서비스 로직은 data를 요구하지 않는다는 것이다. 객체를 생성하고 매서드를 호출했으면 기능이 알아서 실행되게 하자는 것이다.
- 즉 내부 구현을 감추자는 이야기다.
- 두번째 원칙은 의존을 가진 객체의 method만 호출하자는 것이다.
- 다시말해 의존을 가진 객체의 객체의 method를 호출하지 말자는 의미다.

```java
public void processSome(Member member) {
    if (member.getDate().getTime() < ..) {// getDate에서 또 getTime을 부르면 데메테르 위반

    }
}
```

- 데메테르 법칙에 관한 유명한 코드는 아래와 같다.
- 아래 코드로 돈을 받을 수 있지만, 이를 실제 현실로 옮기면 다음과 같은 절차다.
- 지갑을 받는 게 아니라 고객이 돈을 지불하게 바꿔야 한다.
```
고객님 지갑주세요
지갑에 돈있는지 볼게요
돈있으니까 돈 빼갈게요
```
```java
public class Customer {
    private Wallet wallet;

    public Wallet getWallet() {
        return wallet;
    }
}

public class Wallet {
    private int money;
    public int getTotalMoney() {
        return money;
    }
    public void substractMoeny(int debit) {
        moeny -=debit;
    }
}

//돈받는 코드
int payment = 10000;
Wallet wallet = customer.getWallet();
if (wallet.getTotalMoney() >= payment) {
    wallet.substractMoney(payment);
} else {
    //다음에 요금 받게 처리
}
```

- 아래와 같이 class 내부로 캡슐화하면, 객체의 객체의 method를 부르는 일이 없어진다.
- 또한 지갑에서 돈을 받는 게 안리ㅏ 주머니에서 돈을 받는 방식으로 구현이 바뀌어도 getPayment()만 바꿔주면 된다.
```java
public class Customer {
    private Waller wallet;

    public int getPayment(int payment) {
        if (wallet == null) {
            throw new NotEnoughMoneyException();
        }
        if (wallet.getTotalMoney >= payment) {
            wallet.substractMoney(payment);
            
            return payment;
        }

        throw new NotEnoughMoneyException();
    }
}

//돈 받는 코드
int payment = 10000;
try {
    int paidAmount = customer.getPayment(payment);
} catch(NotEnoughMoneyException e) {

}
```


### 상속
```java
public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
        byte[] data = null;
        if (useFile) {
            FileDataReader fileReader = new FileDataReader();
            data = fileReader.read();
        } else {
            SocketDataReader socketReader = new SocketDataReader();
            data = socketReader.read();
        }

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

- 아래와 같이 interface를 사용해 책임을 나누자.
```java
public interface ByteSource {
    public byte[] read();
}

public class FileDataReader implements ByteSource {
    public byte[] read() {

    }

    public class SocketDataReader implements ByteSource {

    }
}

public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
       ByteSource source = null;
        if (useFile) {
          source = new FileDateReader();
        } else {
           source = new SocketDataReader();
        }

        byte[] data = source.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

- 그리고 ByteSource 또한 종류가 변경되도 FlowController가 바뀌지 않게 만든다.
- 그 방법은 class 안에 분류 로직을 만드는 것이다.
- 분류기는 매번 instance를 만들 필요가 없으니 singleton으로 만든다.
```java
public class ByteSourceFactory {
    public ByteSource create() {
        if (useFile) {
            return new FileDataReader();
        } else {
            return new SocketDataReader();
        }
    }

    private boolean useFile() {
        String useFileVal = System.getProperty("useFile");
        
        return useFileVal != null && Boolean.valueOf(useFileVal);
    }

    private static ByteSourceFactory instance = new ByteSourceFactory();
    public static ByteSourceFactory getInstance() {
        return instance;
    }
    private ByteSourceFactory() {}
}

public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
        ByteSource source = ByteSourceFactory.getInstance().create();
        byte[] data = source.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```
- FlowController를 위와 같이 바꾸면 새로운 요청 사항이 들어와도, FlowController를 건들필요가 없다.
- HTTPs를 이용해서 데이터를 읽어오려면 ByteSourceFactory의 create에만 관련 내용을 넣어주면 된다.
- 이전 코드를 보면 데이터를 읽어오는 객체를 생성하는 책임, 흐름을 제어하는 책임이 한 객체에 몰아져 있었음을 알 수 있다.

```java
public void process() {
    //데이터 읽기 객체 직접 생성
    FileDataReader reader = new FileDataReader();
    //흐름 제어: 1. 읽기
    byte[] data = reader.read();

    //흐름 제어: 2. 암호화
    Encryptor encryptor = new Encryptor();
    byte[] encryptedData = encryptor.encrypt(data);
    
    //흐름 제어: 3. 쓰기
    FileData Writer writer = new FileDataWriter();
    writer.write(encryptedData);
}
```
- 진행한 두 번의 추상화는 다음과 같다.
```
바이트 데이터 읽기: ByteSource interface 도출
ByteSource 객체 생성하기: ByteSourceFactory 도출
```
- 이러한 추상화를 통해 상위 수준의 controller 로직을 건들지 않고 그대로 놔둘 수 있었다.
- 이처럼 상위 수준(controller) logic은 바뀌지 않게 확장가능한 설계를 짜는 것이 중요하다.
- 그러한 추상화를 위해서 바로 중요한 게 interface를 활용하는 것이다.
- interface는 자바 내부의 interface가 아니라 객체의 스펙, 규격서라는 개념에서의 interface다.
```
program to interface(인터페이스에 대고 프로그래밍하기)  
다만 interface를 쓰면 복잡해지므로 변화가능성이 높은 곳에만 활용해야 한다.
```

- 인터페이스는 인터페이스 사용자 입장에서 만들어야 한다.
- FileDateReader와 SocketFileDataReader class를 모두 아우르는 naming이 필요하다.
- 더 나아가 본질을 포착해 naming을 해야한다. 어쨌든 데이터를 읽어온다는 의미에서 ByteSource라고 지어주는 게 더 좋다.
- FileDateReaderInterface는 바람직하지 않은 naming이다.
- 객체를 나누면 좋은 다른 점은 test가 가능해진다는 점이다.
```java
public class FlowController {
    public void process() {
        FileDataReader reader = new FileDataReader(); //구현못했으면 read() test 불가.. interface를 implements한 class 만들어 대리 test도 불가. IF가 아예 없음. concrete classㅁ나 존재
        byte[] data = reader.read();
    }
}

public void testProcess() {
    FlowController fc = new FlowController();
    fc.process();
}
```

- 아래와 같이 mock 객체를 활용할 수 있다.
- mock 객체를 활용하면 FileDataReader의 read가 구현이 끝나지 않았어도 test가 가능하다.
```java
public class FlowController {
    private ByteSource byteSource;

    public FlowController(ByteSource byteSource) {
        this.byteSource = byteSource;
    }

    public void process() {
        byte[] data = byteSource.read();
    }
}

public void testProcess() {
    ByteSource mockSource = new MockByteSource();
    FlowController fc = new FlowController(mockSource);
    fc.process();
}

class MockByteSource implements ByteSource {
    public byte[] read() {
        byte[] data = new byte[128];

        return data;
    }
}
```

```java
public abstract class Figure {
    private Bounds bounds = new Bounds(); //위임. 원하는 기능이 이미 다른 class에 구현되어 있으면 조립으로 가져온다.
    private void changeSize() {
        bounds.set(x, y, width, height);
    }

    public boolean contains(Point point) {
        return bounds.contains(point.getX(), point.getY());
    }
}

public abstract class Figure {
    public boolean contains(Point point) {
        Bounds bounds = new Bounds(x, y, width, height);

        return bounds.contains(point.getX(), point.getY());
    }
}

public class AService {
    @Autowired
    BService bService; //Spring의 위임. Singletone으로 bean을 만들어 사용.

    public getSth() {
        bService.get();
    }
}
```

### SOLID
```
SRP- 클래스는 단 하나의 책임을 가진다.
```

- HttpClient을 이용해서 HTML 응답 문자열을 받아 화면을 load하는 형식이다.
```java
public class DataViewer {

    public void display() {
        String data = loadHtml();
        updateGui(data);
    }

    public String loadHtml() {
        HttpClient client = new HttpClient();
        client.connect(url);
        
        return client.getResponse();
    }

    private void updateGui(String data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(String data) {

    }
}
```
- SocketClient를 이용해서 byte[]를 받아 화면을 load하는 형식으로 바뀌었다.
- 즉 데이터를 읽어오는 책임의 기능이 변경된 것이다. 
- 데이터를 보여주는 책임의 기능은 그대로지만, 데이터를 읽어오는 책임의 기능이 같이 있어 코드가 수정된다.
- 단일 책임을 지키지 않아 코드 수정이 많아진다.
```java
public class DataViewer {

    public void display() {
        byte[] data = loadHtml();
        updateGui(data);
    }

    public byte[] loadHtml() {
        SocketClient client = new SocketClient(); //변화
        client.connect(server, port);             //변화
        
        return client.read();                     //변화
    }

    private void updateGui(byte[] data) {          //변화
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(byte[] data) {
        //파싱 코드도 변경
    }
}
```

- OCP
```
기능을 변경하거나 확장할 수 있으면서 기능을 사용하는 코드는 수정하지 않는다.
```

- 아래처럼 만들고 상속을 하면 OCP를 지킬 수 있다.
```java
public class ResponseSender {
    private Data data;
    public ResponseSender(Data data) {
        this.data = data;
    }

    public Data getData() {
        return data;
    }

    public void send() {
        sendHeader();
        sendBody();
    }

    protected void sendHeader() {
        //헤더 데이터 전송        
    }

    protected void sendBody() {
        //text로 데이터 전송
    }
}
```

```java
public class ZippedREsponseEnder extends ResponseSender {
    public ZippedResponseSender(Data data) {
        super(data);
    }

    @Override
    protected void sendBody() {
        //데이터 압축 처리
    }
}
```

- 또는 이전의 ByteSource처럼 interface 등으로 추상화한다.

- OCP가 깨지는 주요 증상은 downcasting이다.
- 특정 class인지 확인하는 처리가 있다면, Character class가 확장될 때 함께 수정될 가능성이 높다.
- 이럴 떄는 drawSpecific()을 Missile이 아닌 Character에 추가하여 추상화한 뒤 사용하는 게 좋다. 
```java
public void drawCharacter(Character character) {
    if (character instanceof Missile) {
        Missile missile = (Missile) character;
        missile.drawSpecific();
    } else {
        character.draw();
    }
}
```

- 비슷한 if - else 블록이 존재해도 의심해야 한다

```java
public class Enemy extends Character {
    private int pathPattern;

    public Enemy(int pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        if (pathPattern == 1) {
            x += 4;
        } else if (pathPattern == 2) {
            y += 10;
        } else if (pathPattern == 4) {
            x += 4;
            y += 10;
        }
        //그려주는 코드
    }
}
```

- int 값을 받는 게 아니라, int를 PathPattern이라는 class로 만들어 객체를 받게 변경한다.
```java
public class Enemy extends Character {
    private PathPattern pathPattern;

    public Enemy(PathPattern pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        int x = pathPattern.nextX();
        int y = pathPattern.nextY();

        //그려주는 코드
    }
}
```

- 새로운 패턴이 생겨도, nextX()와 같은 구현 method만 바꿔주면 된다.
```java
public class Enemy extends Character {
    private PathPattern pathPattern;

    public Enemy(PathPattern pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        int x = pathPattern.nextX();
        int y = pathPattern.nextY();

        //그려주는 코드
    }
}
```

- LSP
```
LSP - 상위 타입 객체를 하위 타입 객체로 치환해도 정상으로 작동해야 한다.
```

```java
public void someMethod(SuperClass sc) {
    sc.someMethod();
}

someMethod(new SubCLass()); //subclass로 넣어도 잘 돌아가야 함.
```

```java
public class Rectangle {
    private int widht;
    private int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = heigth;
    }

    public int getWidth() {
        return width;
    }

    public int getHeight() {
        return heigth;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWIdth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}
```

- Rectangle을 넣으면 무리없이 작동하지만, Rectangle을 상속한 Square를 넣으면 의도와 다르게 된다.
- 따라서 LSP가 지켜지지 않은 것이다.
```java
public void increaseHeight(Rectangle rec) {
    if (rec.getHeight <= rec.getWidth()) {
        rec.setHeight(rec.getWidth() + 10);
    }
}
```

- 이를 커버하기 위해 instanceof로 확인을 시도할 수 있지만, 이 자체로 LSP가 지켜지지 않는다는 의미다.
- 이처럼 우리의 개념 상으로 상속 관계여도, 프로그램 상에서는 상속 관계가 아닐 수 있다.
- 그럴 때 Square class는 Rectangle을 extends하지 않고 별도로 구현해야 한다.

- LSP를 어기게 되는 다른 예는 상위 타입에서 지정한 return 값의 범위를 지키지 않았을 때다.

```java
public class CopyUtil {
    public static void copy(InputStream in, OutputStream out) {
        byte[] data = new byte[512];
        int len = -1;

        while( (len = in.read(data)) != -1) {
            out.write(data, 0, len);
        }
    }
}
```
- read한 결과가 0이게 하위 class를 바꾸면 무한 루프를 돌아버리게 된다.
```java
public class SatanInputStream implements InputStream {
    public int read(byte[] data) {
        .
        .
        .
        
        return 0;
    }
}
```

- LSP가 위반될 가능성이 높은 경우는 아래와 같다.
```
subclass가 명세에서 벗어난 값을 return할 떄
명세에서 벗어난 exception을 throw할 때
명세에서 벗어난 기능을 수행할 떄
```

```java
public class Coupon {
    public int calculateDiscountAmount(item, item) {
        return item.getPrice() * discountRate;
    }
}

public class Coupon {
    public int calculateDiscountAmount(Item item) {
        if (item instanceof SpecialItem) {
            return 0;
        }

        return item.getPrice() * discountRate;
    }
}
```

```java
public class Item {

    public boolean isDiscountAvailable {
        return true;
    }
}

public class SpecialItem extends Item {
    @Override
    public boolean isDiscountAvailable {
        return false;
    }
}

public class Coupon {
    public int calculateDiscountAmount(Item item) {
        if (!item.isDiscountAvailable()) { //어떤 instance인지 확인할 필요가 없다. 다형성으로 parameter의 class type을 check해서 알아서 해당 class에 맞는 method가 call된다.
            return 0;
        }

        return item.getPrice() * discountRate;
    }
}
```

- ISP
```
인터페이스를 사용하는 사용자를 기준으로 분리해야 한다.
```

- 사용자를 기준으로 분리해야 기능 변경의 여파를 최소화할 수 있다.


- DIP

```
추상타입이 있다면, 추상타입이 총괄해야 한다.
```

- 의존 역전 원칙은 소스코드에서의 의존을 역전시키는 원칙이다. runtime에서의 의존을 말하는 것이 아니다.
```java
public class FlowController {
    public void process() {
        FileDataReader reader = new FileDataReader(); 
        //runtime 의존은 FlowController가 FileDataReader에 의존
    }
}

public class FileDataReader implements ByteSource {
    .... //상세 구현에서 추상 타입에 의존
//소스코드에서의 의존은 FlowController가 ByteSource에 의존
}
```

### service locator
- 사용할 객체를 제공하는 책임을 갖는 객체를 Service Locator라고 한다.

```java
public class Worker {
    public void run() {
        JobQueue jobQueue = new JobQueue();
        Transcoder transcoder = new Transcoder();

        while(someRunningCondition) {
            jobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}

public class JobCLI {
    public void interact() {
        printInputSourceMessage();
        String source = getSourceFromConsole();
        printInputTargetMessage();
        String target = getTargetFromConsole();

        JobQueue jobQueue = ...;
        jobQueue.addJob(new JobData(source, target));
    }
}
```

- locator를 transcoder라는 package 안에 넣은 이유는 패키지 간 순환 의존을 발생시키지 않기 위해서다.
- Locator가 package locator라면 transcoder package는 Locator.getInstance()에서 보듯 locator 패키지에 의존
- locator 패키지는 JobQueue에서 보듯 transcoder package에 의존
- 이렇게 하면 서로 맞물리는 의존 관계가 되어버린다.
- 순환 의존은 package의 변경이 다른 패키지에 영향을 줄 수 있다.
- 그럼 배포를 할때도 전혀 변경된 게 없어도 의존성이 있는 package 때문에 다시 배포를 해야 한다.
- 독립 배포가 불가능하게 된다.
```java
package transcoder
public class Worker {
    public void run() {
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        Transcoder transcoder = Locator.getInstance().getTranscoder();

        while(someRunningCondition) {
            jobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
package ui
public class JobCLI {
    public void interact() {
        printInputSourceMessage();
        String source = getSourceFromConsole();
        printInputTargetMessage();
        String target = getTargetFromConsole();

        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        jobQueue.addJob(new JobData(source, target));
    }
}
package transcoder
public class Locator {
    private static Locator instance;
    private static Locator getInstance() {
        return instance;
    }

    public static void init(Locator locator) {
        this.instance = locator;
    }

    private JobQueue jobQueue;
    private Transcoder transcoder;
    public Locator(JobQueue jobQueue, Transcoder transcoder) {
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public JobQueue getJobQueue() {
        return jobQueue;
    }

    public Transcoder getTranscoder() {
        return transcoder;
    }
}
```

- 그럼 Locator 객체는 누가 초기화할까?
- 그 때 main 영역이 초기화 작업을 수행한다.

```java
public class Main {
    public static void main(String[] args) {
        JobQueue jobQueue = new FileJobQueue(); //new StreamingJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();// new mpegTranscoder();

        Locator locator = new Locator(jobQueue, transcoder);
        Locator.init(locator);

        final Worker worker = new Worker();
        Thread t = new Thread(new Runnable() {
            public void run() {
                worker.run(); //Locator는 FileJobQueue, FfmpegTranscoder를 사용
            }
        });
        JobCli cli = new JobCLI();
        cli.interact();
    }
}
```

- 이 경우 main -> application으로 의존성을 갖게 된다.
- main 영역을 변경해도 application은 변경되지 않는다.
- main 영역에서 application에 쓰일 객체를 교체하는 것에 부담이 적어진다.

- 만약 ServiceLocator를 만들 때, 타입만 다르게 구조가 같은 Locator 클래스를 만들어야 할 수도 있다. 그럴 때는 generics를 활용한다.

```java
public class Locator {
    private static Map<Class<?>, Object> objectMap = 
        new HashMap<Class<?>, Object>;

    public static <T> T get(Class<T> klass) {
        return (T) objectMap.get(klass);
    }

    public static void regist(Class<?> klass, Object obj) {
        objectMap.put(klass, obj);
    }
}

public static void main(String[] args) {
    Locator.regist(JobQueue.class, new FileJobQueue());
    Locator.regist(Transcoder.class), new FfmpegTranscoder());

    JobCLI jobCli = new JobCLI();
    jobCli.interact();
}

public class JobCli {
    public void interact() {
        .
        .
        .
        JobQueue jobQueue = ServiceLocator.get(JobQueue.class);
    }
}
```
### DI
- 생성자 방식으로 보통 만든다.
- setter 방식은 필요한 dependency를 생략했다면 compile 시가 아니라 runtime에 오류가 난다.
```java
public class Worker {
    private JobQueue jobQueue;
    private Transcoder transcoder;

    public Worker(JobQueue jobQueue, Transcoder transcoder) {
        if (jobQueue == null) {
            throw new IllegalArgumentException();
        }

        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public void run() {
        while(someRunningCondition) {
            JobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget())
        }
    }
}
```
- 조립기를 별도로 분리해주면 소스코드를 변경할 때 편리하다.

```java
public class Assembler {
    public void createAndWire() {
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        this.worker = new Worker(jobQueue, transcoder);
        this.jobCLI = new JobCLI(jobQueue);

        public Worker getWorker() {
            return this.worker;
        }

        public JobCLI getJobCLI() {
            return this.jobCLI;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Assembler assembler = new Assembler();
        assembler.createAndWire();
        final Worker worker = assembler.getWorker();
        JobCLI jobCli = assembler.getJobCLI();
    }
}
```
- 위와 같이 객체 조립을 분리하면 XML로 따로 뺴서 객체를 조립해도 된다.
- 그게 바로 Spring legacy다.


- setter방식은 test할 때 많이 사용된다.
```java
@Test
public void shouldRunSuccessFully() {
    JobQueue mockJobQueue = Mock 객체;
    Transcoder mockTranscoder = Mock 객체;
    Worker worker = new Worker();
    worker.setJobQueue(mockJobQueue);
    worker.setTranscoder(mockTranscoder);
    worker.run():
}
```

- 주요 디자인 패턴

### strategy

- 아래와 같이 구현된 코드가 있다. target에 따른 if - else 분기문이다.
```java
public class Calculator {
    public int calculate(boolean firstGuest, List<Item> items) {
        int sum = 0;
        for (Item item: items) {
            if (firstGuest) {
                sum += (int) (item.getPrice() * 0.9); //첫 손님 10% 할인
            } else if (! item.isFresh()) {
                sum += (int) (item.getPrice() * 0.8); //덜 신선한 것 20% 할인
            } else {
                sum += item.getPrice();
            }
        }
    }
}
```

- ui 코드는 아래와 같이 될 것이다.
```java
public void onCalculationButtonClick() {
    Calculator cal = new Calculator();
    int price = cal.calculate(firstGuest, items);
}
```


- if - else문의 가장 큰 단점은 추가될 때마다 늘어난다는 점,
- 로직이 같이 추가되면 분석이 너무 어려워진다는 점이다.

```
서로 다른 계산 정책이 한 코드에 섞여 있다.
가격 정책이 추가될 때마다 calculate method 수정이 어려워진다.
```

- 할인과 관련된 책임은 할인 interface를 구현한 class에 맡긴다.
```java
public class Calculator {
    private DiscountStrategy discountStrategy;

    public Calculator(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public int calculate(List<Item> items) {
        int sum = 0;
        for (Item item: items) {
            sum += discountStrategy.getDiscountPrice(item);
        }
    }
}
```

- 하나의 할인정책에 전체금액과 아이템 할인 정책을 method로 가져올 수도 있다. 
```java
public interface DiscountStrategy {
    int getDiscountPrice(Item item);        //아이템 별 할인 정책
    int getDiscountPrice(int totalPrice);   //전체 금액 할인 정책
}
```

- 전체금액 할인 정책과 아이템 별 할인 정책을 분리할 수도 있다.
```java
public interface ItemDiscountStrategy {
    int getDiscountPrice(Item item);
}

public interface TotalPriceDiscountStrategy {
    int getDiscountPrice(int totalPrice);
}
```

```java
public class FirstGuestDiscountStrategy implements DiscountStrategy {

    @Override
    public int getDiscountPrice(Item item) {
        return (int) (item.getPrice() * 0.9);
    }
}
```

- ui 코드는 아래와 같다.
```java
private DiscountStrategy strategy;

public void onFirstGuestButtonClick() {
    strategy = new FirstGuestDiscountStrategy();
}

public void onLastGuestButtonClick() {
    strategy = new LastGuestDiscountStrategy();
}

//firstGuest버튼을 누르면 해당 할인 정책이 적용된다.
public void onCalculationButtonClick() {
    Calculator cal = new Calculator(strategy);
    int price = cal.calculate(items);
}
```

### template

- 일부를 빼고 동일한 절차를 가진 코드를 작성하기도 한다. 게시판에 파일을 insert하거나 update할 때, 특히 그러하다.
```java
public class DbAuthenticator {
    public Auth authenticate(String id, String pw) {
        User user = userDao.selectById(id);
        boolean auth = user.equalPassword(pw);
        if (!auth) {
            throw createException();
        }

        return new Auth(id,user.getName());
    }

    private AuthException createException() {
        
        return new AuthException();
    }
}

public class LdapAuthenticator {
    public Auth authenticate(String id, String pw) {
        boolean lauth = ldapClient.authenticate(id, pw);
        if(!auth) {
            throw createException();
        }

        LdapContext ctx = ldapClient.find(id);
            
        return new Auth(id, ctx.getAttribute("name"));
    }

    private AuthException createException() {
        
        return new AuthException();
    }
}
```

- template method는 하위 class가 아닌 상위 class가 기능 사용 여부를 결정한다.
```java
public abstract Authenticator {
    public Auth authenticate(String id, String pw) {
        if (!doAuthenticate(id, pw)) {
            throw createException();
        }

        return createAuth();
    }

    protected abstract boolean doAuthenticate(String id, String pw);

    private RuntimeException createException() {
        throw new AuthException();
    }

    protected abstract Auth createAuth(String id);
}

public class LdapAuthenticator extends Authenticator {
    @Override
    protected boolean doAuthenticate(String id, String pw) {
        return ldapClient.authenticate(id, pw); 
    }

    @Override
    protected Auth createAuth(String id) {
        LdapContext ctx = ldapClient.find(id);
        
        return new Auth(id, ctx.getAttribute("name"));
    }
}
```

- 템플릿 메서드와 전략 패턴을 함께 사용할 수도 있다.

```java
public <T> T execute(TransactionCallback<T> action) throws TransactionException {
    TransactionStatus status = this.transactionManager.getTransaction(this);
    T result;

    try {
        result = action.doInTransaction(status);
    } catch (RuntimeException ex) {
        rollbackOnException(status, ex);
        throw ex;
    }

    this.transactionManager.commit(status);
    return result;
}

transactionTemplate.execute(new TransactionCallback<String>() {
    public String doInTransaction(TransactionStatus status) {
        //트랜잭션 범위 안에서 실행될 코드
    }
})
```

### state
- 아래와 같은 요구가 있다고 가정해보자.

```
동작        조건            실행               결과
동전넣음    동전X           금액증가            제품선택가능하게변경
동전넣음    제품선택가능     금액증가            제품선택가능
제품선택    동전X           동작X               동전없음유지
제품선택    제품선택가능     제품주고잔액감소     잔액있으면제품선택가능,잔액없으면동전없음상태로 변경   
```

```java
public class VendingMachine {
    public static enum State {NOCOIN, SELECTABLE}

    private State state = State.NOCOIN;

    public void insertCoin(int coin) {
        switch (state) {
            case NOCOIN:
                increaseCoin(coin);
                state = State.SELECTABLE;
                break;
            case SELECTABLE:
                increaseCoin(coin);
        }

        public void select(int productId) {
            switch (state) {
                case NOCOIN:
                    break;
                case SELECTABLE:
                    provideProduct(productId);
                    decreaseCoin();
                    if (hasNoCoin()) {
                        state = State.NOCOIN;
                    }
            }
        }
    }
}
```

- 그러다가 자판기에 제품이 없는 경우에 동전을 넣으면 바로 동전을 되돌려달라는 요구가 들어왔다.
- 그럼 아래와 같이 state가 추가된다.
```java
public class VendingMachine {
    public static enum State {NOCOIN, SELECTABLE, SOLDOUT}

    private State state = State.NOCOIN;

    public void insertCoin(int coin) {
        switch (state) {
            case NOCOIN:
                increaseCoin(coin);
                state = state.SELECTABLE;
                break;
            case SELECTABLE:
                increaseCoin(coin);
                break;
           /* case SOLDOUT:
                returnCoin(); */
        }
    }

    public void seleect(int productId) {
        switch(state) {
            case NOCOIN:
                break;
            case SELECTABLE:
                provideProduct(productId);
                decreaseCoin();
                if (hasNoCoin()) {
                    state = state.NOCOIN;
                }
           /* case SOLDOUT: */
        }
    }
}
```

- 상태에 따라 동일한 기능 요청의 처리를 다르게 한다는 의미가 담겨있다.
- 이를 state 패턴으로 구현한다.
```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this); //상태 객체에 위임
    }

    public void select(int productId) {
        state.select(productId, this); //상태 객체에 위임
    }

    public void changeState(State newState) {
        this.state = newState;  
    }
}

public class NoCoinState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
        vm.changeState(new SelectableState());
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        SoundUtil.beep();
    }
}

public class SelectableState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        vm.provideProduct(productId);
        vm.decreaseCoin();

        if (vm.hasNoCoin())  {
            vm.changeState(new NoCoinState());
        }
    }
}

```

- State 패턴의 장점은 class는 증가해도, if - else 조건문이 늘어나진 않아 본 코드가 복잡해지지 않는다는 점이다.
- 그리고 상태 별 class가 따로 존재하므로 해당 상태별 코드 수정을 건드리려면 해당 class만 건들면 된다는 점이다. 흩뿌려진 method를 수정하지 않아도 된다.
- 위에서는 State 객체에서 상태를 변경했다.
- 아래에서는 컨텍스트에서 상태를 변경해보자.

```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this); //상태 객체에 위임
        if (hasCoin()) {
            changeState(new SelectableState());
        }
    }

    public void select(int productId) {
        state.select(productId, this); //상태 객체에 위임
        if (state.isSelectable() && hasNoCoin()) {
            changeState(new NoCoinState());
        } 
    }

    private void changeState(State newState) {
        this.state = newState;  
    }

    private boolean hasCoin() {

    }

    private boolean hasNoCoin() {
        return !hasCoin();
    }
}

public class SelectableState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        vm.provideProduct(productId);
        vm.decreaseCoin();
    }
}
```

### 데코레이터
- 상속으로만 하면 여러 기능을 함께 제공할 때 계층 구조가 복잡해진다.
- 따라서 상속이 아닌 위임을 사용하는 decorator 패턴으로 바꾼다.

```java
public abstract class Decorator implements FileOut {
    private FileOut delegate; //위임 대상
    public Decorator(FileOut delegate) {
        this.delegate = delegate;
    }

    protected void doDelegate(byte[] data) {
        delegate.writer(data); //delegate에 쓰기 위임
    }
}

public class EncryptionOut extends Decorator {
    public EncryptionOut(FileOut delegate) {
        super(delegate);
    }

    public void write(Byte[] byte) {
        byte[] encryptedData = encrypt(data);
        super.doDelegate(encryptedData);
    }

    private byte[] encrypt(byte[] data) {

    }
}
```

- 데코레이터는 조합할 수 있다는 데 장점이 있다.
- 압축을 한 뒤 암호화를 해서 파일에 쓰고 싶다면 아래와 같이 Zipout class를 넣어주면 된다.
```java
FileOut delegate = new FileOutImpl();
FileOut fileOut = new EncryptionOut(delegate);
fileOut.write(data);

FileOut delegate = new FileOutImpl();
FileOut fileOut = new EncryptionOut(new ZipOut(delegate));
fileOut.write(data);
```

- 기능 적용 순서 변경도 쉽다.

```java
FileOut fileOut = new BufferedOut(EncryptionOut(new ZipOut(delegate))); // 버퍼 -> 암호화 -> 압축 -> 파일쓰기
FileOut fileOut = new EncryptionOut(new ZipOut(new BufferedOut(delegate))); //암호화 -> 압축 -> 버퍼 -> 파일 쓰기
```

- 다만 decorator 패턴은 기능 갯수가 적을 때 쓰는 게 좋다. 기능이 많으면 너무 복잡해진다
- 또한 데코레이션 객체가 비정상으로 동작할 때 어떻게 처리할 지 정해야 한다. 보통은 throw Exception보다는 로그를 남긴다.


### proxy 

- 제품 목록을 구성할 떄, 모든 이미지를 로딩시키면 불필요하게 메모리를 사용한다.
- 스크롤해서 보이는 것만 보여주면 되는데, 안보이는 것까지 로딩하기 때문이다.
- 또한 이미지 리소스를 로딩하는 데 시간이 더 오래걸려 반응성이 안 좋아진다.

- 그럴 때 Image class의 DynamicLoadingImage class를 사용할 수 있다.
- 하지만 이미지 로딩 방식을 변경하면 ListUI 코드를 변경해야 한다.
- 이 때 ListUI 코드를 변경하지 않고 로딩 방식을 교체할 수 있다.

```java
public class ProxyImage implements Image {
    private String path;
    private RealImage image;

    public ProxyImage(String path) {
        this.path = path;
    }

    public void draw() {
        if (image == null) {
            image = new RealImage(path); //최초 접근 시 객체 생성
        } 
        image.draw();
    }
}
```

```java
public class ListUI {
    private List<Image> images;
    public ListUI(List<Image> images) {
        this.images = images;
    }

        
    public void onScroll(int start, int end) {
        for (int i = start; i <= end; i++) {
            Image image = images.get(i);
            image.draw();
        }
    }
}
```

```java
List<String> paths = ...//이미지 경로 목록을 가져옴
List<Image> images = new ArrayList<Image>(paths.size());
for (int i = 0; i < paths.size(); i++) {
    if (i <= 4) { //상위 4개는 바로 이미지를 로딩
        images.add(new RealImage(paths.get(i)));
    } else {      //나머지는 이미지가 필요할 때 load
        images.add(new ProxyImage(paths.get(i)));
    }
}

ListUI listUI = new ListUI(images);
```

### adapter

- 게시글이 많아지면서 like의 성능이 떨어졌다.
- 검색 속도 문제 해결을 위해 Tolr라는 오픈소스 검색 서버를 도입했다.
- 문제는 TolrClient가 제공하는 interface가 내가 만든 SearchService와 맞지 않다는 점이다.
- 그럼 전부 TolrClient에 맞게 변경해야하는데, 작업량이 만만치 않다.
- 그럴 때 아래와 같이 adapter를 사용한다.
```java
public class SearchServiceTolrAdapter implements SearchService {
    private TolrClient tolrClient = new TolrClient();

    public searchResult search(String keyword) {
        TolrQuery tolrQuery = new TolrQuery(keyword);
        QueryResponse response = tolrClient.query(tolrQuery);
        SearchResult result = convertToResult(response);
        
        return result;
    }

    private SearchResult convertToResult(QueryResponse response) {
        List<TolrDocument> tolrDocs = response.getDocumentList().getDocuments();
        List<SearchDocument> docs = new ArrayList<SearchDocument>();
        for (TolrDocument tolrDoc : tolrDocs) {
            docs.add(new SearchDocument(tolrDoc.getId(), ...));
        }

        return new SearchResult(docs);
    }
}
```

- adapter pattern을 적용한 것이 바로 SLF4J다. 
- 자바 로깅, log4j, log4j2, logback 등의 프레임워크를 선택하여 사용할 수 있게 해준다.


### observer
- 웹 사이트의 상태를 확인해 응답 속도가 느리거나, 연결이 안되면 모니터링 담당자에게 이메일로 통지하는 시스템을 만든다고 해보자.

```java
public class StatusChecker {
    private EmailSender emailSender;

    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
        }
    }
}
```

- 여기서 긴급한 메시지는 SMS로 바로 알려달라는 요건이 추가되었다.
- 이를 반영해보자.

```java
public class StatusChecker {
    private EmailSender emailSender;
    private SmsSender smsSender;

    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
            smsSender.sendSms(status);
        }
    }
}
```

- 만약 회사 내부에서 사용하는 메신저로도 메시지를 보내달라는 요구가 들어오면?
```java
public class StatusChecker {
    private EmailSender emailSender;
    private SmsSender smsSender;
    private Messenger messengerSender;
    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
            smsSender.sendSms(status);
            messengerSender.sendMessenger(status);
        }
    }
}
```

- StatusChecker class가 변경되지 않게 observer pattern을 활용해보자.

```java
public abstract class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<StatusObserver>();

    public void add(StatusObserver observer) {
        observers.add(observer);
    }

    public void remove(StatusObserver observer) {
        observers.remove(observer);
    }

    public void notifyStatus(Status status) {
        for (StatusObserver observer: observers) {
            observer.onAbnormalStatus(status);
        }
    }
}
```

```java
public class StatusChecker extends StatusSubject {
    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            super.notifyStatus(status);
        }
    }

    private Status loadStatus() {

    }
}
```

```java
public interface StatusObserver {
    void onAbnormalStatus(Status status);
}

public class StatusEmailSender implements StatusObserver {

    @Override
    public void onAbnormalStatus(Status status) {
        sendEmail(status);
    }

    private void sendEmail(Status status) {

    }
}
```


- 옵저버로 등록하였고, 시스템이 비정상이 되면 StatusChecker -> StatusEmailSender -> onAbnormalStatus 호출해 상태 정보 통지
```java
StatusChecker checker = new StatusChecker();
checker.add(new StatusEmailSender());
```

- observer를 이용하면 아래와 같이 FaultStatusSMSSender로 바꿔도 StatusChecker class는 바뀌지 않는다.
- 또한 기존에 있던 StatusEmailSender도 똑같이 넣어서 작동시킬 수 있다.
```java
StatusChecker checker = ...;
StatusObserver faultObserver = new FaultStatusSMSSender();
checker.add(faultObserver);
checker.add(new StatusEmailSender());
```

- 옵저버 객체가 기능을 수행할 떄, 주제 객체의 상태가 필요할 수 있다.
- FaultStatusSMSSender class는 장애일 때만 SMS를 전송하고, 응답속도가 느려지는 장애 이외의 비정상 상태는 메시지를 전송하지 않게 구현한다고 해보자.
- 이 떄 FaultStatusSMSSender는 상태값을 확인해야 한다.
```java
public abstract class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<StatusObserver>();

    public void notifyStatus(Status status) {
        for (StatusObserver observer : observers) {
            observer.onAbnormalStatus(status); //상태를 옵저버에 전달
        }
    }
}

public class FaultStatusSMSSender implements StatusObserver {
    public void onAbnormalStatus(Status status) { //상태값 전달받음
        if (status.isFault()) { //전달받은 상태값 사용
            sendSMS(status);
        }
    }
}
```

- 만약 onAbnormalStatus()를 호출할 떄 전달한 객체만으로 부족하다면?
- 그럴 때는 직접 구현해야 한다. statusChecker를 받아 statusChecker.isContinuousFault() 조건이 추가된다.  
```java
public class SpecialStatusObserver implements StatusObserver {
    private StatusChecker statusChecker;
    private Siren siren;

    public SpeicalStatusObserver(StatusChecker statusChecker) {
        this.statusChecker = statusChecker;
    }

    public void onAbnormalStatus(Status status) {
        if (status.isFault() && statusChecker.isContinuousFault()) {
            siren.begin();
        }
    }
}
```

- GUI 프로그래밍에서 observer가 많이 사용된다.
- Button class의 상위타입인 View가 존재하고, View class가 옵저버 객체를 관리하기 위한 기능을 제공한다.
```java
public class MyActivity extends Activity implements View.OnClickListener {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        ...
        Button loginButton = getViewById(R.id.main_loginbtn);
        loginButton.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        login(id, password);
    }
}

public class MyActivity extends Activity implements View.OnClickListener {
    public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        ...
        Button loginButton = (Button) findViewById(R.id.main_loginbtn); //동일한 OnClickListener 객체 등록
        loginButton.setOnClickListener(this);
        Button logoutButton = (Button) findViewById(R.id.main_logoutbtn); //동일한 OnClickListener 객체 등록
        logoutButton.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        //주제 객체를 구분하기 위한 분기. 동일한 onClickListener라도 구분 가능.
        if (v.getId() == R.id.main_loginbtn) {
            login(id, password);
        } else if (v.getId() == R.id.main_logoutbtn) {
            logout();
        }
    }
}
```

### mediator
- ??? ㅈㄴ 어렵다. 잘 모르겠다..
```java
private VideoMediator videoMediator;

public void onSelectedItem(int selectedIdx) {
    VideoInfo videoInfo = videoList.get(selectedIdx);
    videoMediator.selectVideo(videoInfo.getFile());
}
```

### abstract Factory
- 게임 플레이를 진행하는 Stage를 class로 나타내보자.
- 몇 단계인지 에따라 장애물, 보스, 적이 달라진다.

```java
public class Stage {
     ....   
    private void createEnemies() {
        for (int i = 0; i <= ENEMY_COUNT; i++) {
            if (stageLevel == 1) {
                enemies[i] = new DashSmallFlight(1,1); //공격/수비 1
            } else if (stageLevel == 2) {
                enemies[i] = new MissileSmallFlight(1,1);
            }
        }
        if (stageLevel == 1) {
            boss = new StrongAttackBoss(1, 10); //공격/수비 1/10
        } else if (stageLevel == 2) {
            boss = new CloningBoss(5, 20);
        }
    }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
            if (stageLevel == 1) {
                obstacles[i] = new RockObstacle();
            } else {
                obstacles[i] = new BombObstacles();
            }
        }
    }
}
```

- if - else 문이 많아 꽤나 복잡하다.
- 이를 abstract Factory를 이용해 바꿔보자.

```java
public abstract class EnemyFactory {
    public static EnemyFactory getFactory(int level) {
        if (level == 1) {
            return EasyStageEnemyFactory(); //1레벨도 빡세면 EasyStageEnemyFactory를  SuperEasyStageEnemyFactory로 바꿀 수있다.
        } else {
            return HardEnemyFactory();
        }
    }

    public abstract Boss createBoss();
    public abstract SmallFlight createSmallFlight();
    public abstract Obstacle createObstacle();
}

public class EasyStageEnemyFactory extends EnemyFactory {
    public Boss createBoss() {
        return new StrongAttackBoss();
    }

    public abstract SmallFlight createSmallFlight() {
        return new DashSmallFlight();
    }

    public abstract Obstacle createObstacle() {
        return new RockObstacle();
    }
}

public class HardEnemyFactory extends EnemyFactory {
    public Boss createBoss() {
        return new CloningAttackBoss();
    }

    public abstract SmallFlight createSmallFlight() {
        return new MissileSmallFlight();
    }

    public abstract Obstacle createObstacle() {
        return new BombObstacle();
    }
}
```

```java
public class Stage {
   private EnemyFactory enemyFactory;
   private Enemies enemies;
   private Boss boss;
   private Obstacles obstacles;

   public class Stage(int level) {
        enemyFactory = EnemyFactory.getFactory(level);
   }   

   private void createEnemies() {
    for (int i = 0; i <= ENEMY_COUNT; i++) {
        enemies[i] = enemyFactory.createSmallFlight();
    }

    boss = enemyFactory.createBoss();
   }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
           obstacles[i] = enemyFactory.createObstacle();
        }
    }
}
```

```java
public class Stage {
   private EnemyFactory enemyFactory;
   private Enemies enemies;
   private Boss boss;
   private Obstacles obstacles;

   public class Stage(int level, EnemyFactory enemyFactory) {
        this.level = level;
        this.enemyFactory = EnemyFactory enemyFactory;
   }   

   private void createEnemies() {
    for (int i = 0; i <= ENEMY_COUNT; i++) {
        enemies[i] = enemyFactory.createSmallFlight();
    }

    boss = enemyFactory.createBoss();
   }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
           obstacles[i] = enemyFactory.createObstacle();
        }
    }
}
```



## design pattern
- AbstractFacotry 패턴
  - 일련의 인스턴스군을 모아서 생성하기

```
DBMS를 가져올 때 필요한 instance를 모아둔다.
```

```java
//interface로 만들어 어떤 DBMS든 가져올 수 있게끔 만든다.
public interface Facotry {
    Connection getConnection();

    Configuration getConfiguration();
}

//abstrac class로 만들어 어떤 Connection 방식이든 가능하게 한다.
public abstract class Connection {

}

//abstrac class로 만들어 어떤 Configuration 방식이든 가능하게 한다.
public abstract class Configuration {

}

//Factory interface를 구현하여 PostgreSQLFactory와 관련된 connection, configuration을 얻어오게 한다.
public class PostgreSQLFactory implements Facotry {
    @Override
    public Connection getConnection() {
        return new PostgreSQLConnection();
    }

    @Override
    public Confugration getConfiguration() {
        return new PostgreSQLConfiguration();
    }
}

//실제 PostgreConnection을 구현하는 class다.
public class PostreSQLConnection extends Connection {

}

//실제 PostgreSQLConfiguration을 구현하는 class다.
public class PostgreSQLConfiguration extends Configuration {

}
```

- 아래는 Postgre가 아닌 Mysql용으로 만든 것이다.

```java
public class MYSQLFactory implements Factory {
    @Override
    public Connection getConnection() {
        return new MYSQLConnection();
    }

    @Override
    public Configuration getConfiguration() {
        return new MYSQLConfiguration();
    }
}

public class MYSQLConnection extends Connection {

}

public class MYSQLConfiguration extends Configuration {

}
```

- 실제 Factory 사용 class는 아래와 같다.

```java
public class SampleMain {
    public static void main(String ...args) {
        String env = "PostgreSQL";

        Facotry facotry = createFactory(env);
        Connection connection = facotry.getConnection();
        Configuration configuration = factory.getConfiguration();
    }

    private static Factory createFactory(String env) {
        switch (env) {
            case "PostgreSQL":
                return new PostgreSQLFactory();
            case "MYSQL":
                return new MySQLFactory();
            default:
                throw new IllegalArgumentException(env);
        }
    }
}
```

- 즉 순서는 아래와 같다.

```
Factory에 필요한 기능을 확장할 수 있게 abstract class를 만든다. --> abstract class Connection, Configuration
실게 기능을 구현한 class를 만든다. --> class PostreSQLConnection extends Connection, class PostgreSQLConfiguration extends Configuration
Facotry용 interface를 만든다. --> Facotry interface
경우에 맞는 실제 Facotry class를 만든다. --> class MYSQLFactory implements Factory
실제 Factory를 생성한다 --> Facotry facotry = new PostgreSQLFactory();, new MySQLFactory();
```

- Builder 패턴
  - 복합화된 instance의 생성 과정을 은폐한다.

```java
public interface Builder {
    void createHeader();
    
    void createContents();

    void createFooter();

    Page getResult();
}

public class Page {
    private String header;

    private String content;

    private String footer;
}

public class TopPage extends Page {

}

public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public Page construct() {
        builder.createHeader();
        builder.crateContents();
        builder.createFooter();

        return builder.getResult();
    }
}

public class TopPageBuilder implements Builder {
    private TopPage page;

    public TopPageBuilder() {
        this.page = new TopPage();
    }

    @Override
    public void createHedaer() {
        this.page.setHeader("Header");
    }

    @Override
    public void createContents() {
        this.page.setContent("Contents");
    }

    @Override
    public void createFooter() {
        this.page.setFooter("Footer");
    }

    @Override
    public Page getResult() {
        return this.page;
    }
}
```

- 위처럼 Builder를 만들어두면 복잡한 instance를 만드는 과정이 전부 은폐되고 3줄짜리로 instance를 만드는 것처럼 보이게 된다.

```java
public class SampleMain {
    public static void main(String ...args) {
        Builder builder = new TopPageBuilder();
        Director director = new Director(builder);

        Page page = director.construct();
    }
}
```

- Adapter 패턴
  - 인터페이스에 호환성이 없는 클래스들을 조합시키기
  - 기존 시스템과 새로운 시스템이 완전히 다른 method를 사용하는 경우 사용할 수 있다.

```java
public class OldSystem {
    public void oldProcess() {
        //기존처리
    }
}

public abstract class Target {
    abstract void process();
}

public class Adapter extends Target {
    private OldSystem oldSystem;

    public Adapter() {
        this.oldSystem = new OldSystem();
    }

    @Override
    public void process() {
        this.oldSystem.oldProcess();
    }
}

public class SampleMain {
    public static void main(String[] args) {
        Target target = new Adapter();
        target.process();
    }
}
```

- Composite 패턴
  - 재귀적 구조 쉽게 처리
  - 파일 시스템에 쓰기 좋다.

- 아래는 실제 Java의 File, Directory class는 아니다. sudo다.

```java
public interface Entry {
    void add(Entry entry);

    void remove();

    void rename(String name);
}

public class File implmenets Entry {
    private String name;

    public File(String name) {
        this.name = name;
    }

    @Override
    public void add(Entry entry) {
        throw new UnsupportedOpertaionException();
    }

    @Override
    public void remove() {
        System.out.println(this.name + "를 삭제했다.");
    }

    @Override
    public void rename(String name) {
        this.name = name;
    }
}

public class Directory implements Entry {
    private String name;

    private List<Entry> list;

    public Directory(String name) {
        this.name = name;
        this.list = new ArrayList<>();
    }

    @Override
    public void add(Entry entry) {
        list.add(entry);
    }

    @Override
    public void remove() {
        Iterator<Entry> itr = list.iterator(); //Entry를 구현한 class는 다 들어올 수 있다. 그게 File과 Directory다.
        while (itr.hasNext()) {// 한마디로 File인지, Directory인지 신경안쓰고 모두 remove가 가능하다.
            Entry entry = itr.next();
            entry.remove();
        }
        System.out.println(this.name + "을 삭제했다.");
    }

    @Override
    public void rename(String name) {
        this.name = name;
    }
}
```

- Command 패턴
  - 명령을 instance로 취급한다.
  - 할인 정책 같은 곳에 많이 쓰인다.

```java
@Setter
public abstract class Command {
    protected Book book;

public abstract void execute();
}

@Getter
@Setter
@RequiredArgsConstructor
public class Book {
    private double amount;
}

public class DiscountCommand extends Command {
    @Override
    public void execute() {
        double amount = book.getAmount();
        book.setAmount(amount * 0.9);
    }
}

public class SpecialDiscountCommand extends Command {
    @Override
    public void execute() {
        double amount = book.getAmount();
        book.setAmount(amoutn * 0.7);
    }
}

public class SampleMain {
    public static void main(String... args) {
        Book comic = new Book(5000);

        Book technicalBook = new DiscountCommand();

        Command discountCommand = new DiscountCommand();

        Command specialDiscountCommand = new SpecialDiscountCommand();

        discountCommand.setBook(comic);
        discountCommand.execute();
        System.out.println("할인 후 금액은 " + comic.getAmount() + "원");

        discountCommand.setBook(technicalBook);
        discountCommand.execute();
        System.out.println("할인 후 금액은 " + technicalBook.getAmount() + "원");

        speicalDiscountCommand.setBook(technicalBook);
        speicalDiscountCommand.execute();
        System.out.println("할인 후 금액은" + technicalBook.getAmount() + "원");
    }
}
```

- Strategy 패턴
  - 처리 알고리즘(전략)을 간단하게 전환
  - 처리 조건에 따라 전략을 바꿀 때 좋음. ex) 할인

```java
public interface Strategy {
    void discount(Book book);
}

@RequiredArgsConstructor
@Setter
@Getter
public class Book {
    private double amount;
}

public class DiscountStrategy implements Strategy {
    @Override
    public void discount(Book book) {
        double amount = book.getAmount();
        book.setAmount(amount * 0.9);
    }
}

public class SpecialDiscountStrategy implements Strategy {
    @Override
    public void discount(Book book) {
        double amount = book.getAmount();
        book.setAmount(amount * 0.7);
    }
}

public class Shop {
    private Strategy strategy;

    public Shop(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = stratgey;
    }

    public void sell(Book book) {
        this.stratgey.discount(book);
    }
}

public class SampleMain {
    public static void main(String... args) {
        Book comic = new Book(5000);

        Book technicalBook = new Book(25_000);

        Strategy discountStrategy = new DiscountStrategy();

        Shop shop = new Shop(discountStrategy);
        shop.sell(comic);
        System.out.println("할인 후 금액은 " + comic.getAmount() + "원" );

        shop.setStrategy(specialDiscountStrategy);
        shop.sell(technicalBook);
        System.out.println("할인 후 금액은" + technicalBook.getAmount() + "원");
    }
}
```

- Observer pattern
  - 어떤 인스턴스의 상태가 변화할 때 그 인스턴스 자신이 상태의 변화를 통지하는 구조를 제공한다.
  - 다른 시스템으로부터 데이터를 수신, 사용자가 버튼을 누름 -> 상태 변화의 계기. 이걸 감지하여 처리하는 프로그램 구현
  - 상태가 바뀔 때 필요한 처리 호출하면 상태 보관 클래스와 호출 클래스가 결합도가 높아짐. 확장성 떨어짐

```java
public interface Observer {
    void update(Subject subject);
}

public abstract class Subject {
    private List<Observer> observers = new ArrayList<>();

    public void addObserver(Observer observer) {
        this.observers.add(observer);
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
    public abstract void execute();
}

public class Client implements Observer {
    @Override
    public void update(Subject subject) {
        System.out.println("통지를 수신했다.");
    }
}

public class DataChanger extends Subject {
    private int status;

    @Override
    public void execute() {
        status++;
        System.out.println("상태가" + status + "로 바뀌었다.");
        notifyObservers(); 
    }
}

public class SampleMain {
    public static void main(String... args) {
        Observer observer = new Client();        // 상태 변경을 감시
        Subject dataChanger = new DataChanger(); // 상태 변경을 통지

        dataChagnger.addObserver(observer);
        for (int count = 0; count < 10; count ++){
            dataChanger.execute();

            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

- 핵심은 Subject는 통지할 Observer를 보관한다.
- notifyObserver method가 호출되면 Observer에 통지한다. (update method 호출)
- 정보를 '통지'하는 구조가 Observer interface와 Subject 추상 클래스에서 제공된다.
- 실제 처리는 각각을 구현하고 상속한 class에서 이뤄진다.