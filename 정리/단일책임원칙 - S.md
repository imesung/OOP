## 단일책임원칙 - SRP(Single Responsibility Principle)

**정의**

작성된 클래스는 단 하나의 기능만 가지며 **클래스가 제공하는 모든 서비스는 그 하나의 책임을 수행하는데 집중**되어야 한다는 원칙이다. 이는 다시 말해서, 어떤 변화에 의해 클래스를 변경해야하는 이유는 오직 하나뿐이어야 한다는 의미이다.

SRP 원리를 적용하면 **책임 영역이 확실해지기 때문에 한 책임의 변경에서 다른 책임의 변경으로 연쇄작용이 자유로울 수 있다.** 이로 인해 **코드의 가독성 향상과 유지보수 용이라는 이점**을 얻을 수 있고, **OCP 원리 뿐만 아니라 다른 원리들을 적용하는 기초**가 된다.



**적용방법**

여러 원인에 의한 변경

- Extract Class(두 개의 클래스가 해야할 일을 하나의 클래스에서 하는 것)를 통해 혼재된 각 책임을 각각의 개별 클래스로 분할하여 클래스 당 하나의 책임만을 맡도록 하는 것이다. **여기에서 중요한 것은 책임만을 분리하는 것이 아니라 분리된 두 클래스간의 관계 복잡도를 줄이도록 설계하는 것이다.** 또한, 만약에 Extract Class 각각의 클래스들이 유사하고 비슷한 책임을 가지고 있다면 Extract Superclass(인터페이스, 추상클래스)를 사용할 수 있다. 따라서, Extract Class들이 유사한 책임들은 Superclass에게 명백히 위임하고 다른 책임들은 각자에게 정의할 수 있다.



산탄총 수술

**Move Field** : 필드가 자신이 정의된 클래스보다 다른 클래스에 의해서 더 많이 사용된다면, 타겟 클래스에 새로운 필드를 만들고 기존 필드를 사용하는 부분을 모두 변경하는 것이다.

**Move Method** : 메소드가 자신이 정의된 클래스보다 다른 클래스의 기능을 더 많이 사용하고 있다면(책임이 맞지 않다면) 이 메소드를 가장 많이 사용하고 있는 클래스에 새로운 메소드를 만들고 이전 메소드는 간단히 위임하거나 완전히 제거한다.

- Move Field와 Move Method를 통해 책임을 기존의 어떤 클래스로 모으거나, 책임을 가질 클래스가 없다면 새로운 클래스를 만들어 해결할 수 있다. 즉 여러 곳에 분포된 책임들을 한 곳에 모으면서 설계를 깨끗하게 하는 것으로, **응집성을 높이는 작업이다.**



**적용사례**

~~~java
//SRP 적용 전
class Car {
 	private String serialNumber;
  private int price;
  private String color;
  
  public Car(String serialNumber, int price, String color) {
    this.serialNumber = serialNumber;
    this.price = price;
    this.color = color;
  }
}
~~~

위 소스를 살펴보면, serialNumber는 고유정보이므로 변경이 되지 않고, price와 color는 특정 정보로서 변경이 발생할 수 있는 부분이다. **이로 인해 특정 정보의 변경이 발생하면 해당 클래스를 수정해야하는 부담이 발생하게 되어 SRP를 적용해야하는 것이다.** 



~~~java
//SRP 적용 후
class Car {
 	private String serialNumber;
  private CarSpec carSpec;
  
  public Car(String serialNumber, CarSpec carSpec) {
    this.serialNumber = serialNumber;
    this.carSpec = carSpec
  }
}

class CarSpec{
  private int price;
  private String color;
  ...
}
~~~

위 소스를 살펴보면, 고유정보와 특정 정보를 분리한 것을 확인할 수 있다. 이로 인해, 특정 정보의 변경이 일어나면 CarSpec 클래스만 변경하게되므로 **변경되는 부분을 한 곳에서만 관리할 수 있게 된 것이다.**



**적용 이슈**

올바른 클래스의 이름은 해당 클래스의 책임을 나타낼 수 있는 가장 좋은 방법이다. 즉, 클래스의 이름을 명확히 지어라.

또한, 무조건 책임을 분리한다고 해서 SRP가 적용되는 것은 아니다. **각 개체의 응집력이 있다면 병합을.. 각 개체의 결합력이 있다면 분리를..**

