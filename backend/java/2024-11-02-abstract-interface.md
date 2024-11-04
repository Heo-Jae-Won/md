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
