## 개방폐쇄의 원칙 - OCP(Open Close Principle)

**정의**

**버틀란트 메이어는 말씀하셨다. 소프트웨어의 구성요소는 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다고..!**

이것은 변경을 위한 비용은 가능한 한 줄이고 확장을 위한 비용은 가능한 한 극대화 해야한다는 이야기이다. 즉, 요구사항의 변경이나 추가사항이 발생하더라도 기존 구성요소는 수정이 일어나지 말아야하며, 기존 구성요소를 쉽게 확장해서 재사용할 수 있어야 한다는 뜻이다.

다시 말해 **OCP는 관리가능하고 재사용 가능한 코드를 만드는 기반**이며, **OCP를 가능케하는 중요 메커니즘은 추상화와 다형성**이다.

OCP는 객체지향의 장점을 극대화 시키는 아주 중요한 원리이다.



**적용방법**

1. 변경 혹은 확장될 것과 변경 혹은 확장되지 않을 것을 엄격히 구분한다.
2. 이 두 모듈이 만나는 지점에 **인터페이스를 정의한다.**
3. 구현에 의존하기보다는 **정의한 인터페이스에 의존하도록 코드를 작성한다.**



**적용사례**

Car에 관련된 소스를 살펴보겠다.

~~~java
class Car {
 	public void ride() {
    System.out.println("자동차를 탄다");
  }

  public void move() {
    System.out.println("자동차를 이동한다.");
  }

  public void back() {
    System.out.println("자동차를 후진한다.");
  }
}
~~~

위 소스에서 만약에 Car 뿐만 아니라 비행기, 기차와 같은 다른 탈것들도 다루어야 한다면, 매번 새로운 클래스를 만들어야 할 것이다.

여기에서 알게 된것은, **변화 자체를 막을 수는 없다는 것이다.** 우리는 변화를 막는 것이 아니라, **변화에 적절히 대응해야 한다는 것이다.**

그럼 이제 변화에 적절히 대응할 수 있도록 소스를 수정해보겠다.

~~~java
public interface Vehicle {
    void ride();
    void move();
}

//Car
public class Car implements Vehicle{
    @Override
    public void ride() {
        System.out.println("자동차를 탄다");
    }
  
    @Override
    public void move() {
        System.out.println("자동차를 이동한다.");
    }
  
    public void back() {
        System.out.println("자동차를 후진한다.");
    }
}

//AirPlane
public class AirPlane implements Vehicle{
    @Override
    public void ride() {
        System.out.println("비행기를 탄다.");
    }
  
    @Override
    public void move() {
        System.out.println("비행기를 이동한다.");
    }
  
    public void fly() {
        System.out.println("비행기가 난다.");
    }
}

~~~

소스를 보면 Car와 AirPlan들을 추상화하기 위해 Vehicle이라는 인터페이스를 생성한 것이다. 즉, 새로운 탈것이 추가되면서 변경이 발생하는 부분을 추상화하여 분리하였음을 확인할 수 있는 것이다.

이로 인해, 새로운 탈것이 추가되어도 다른 탈것에는 영향이 미치지 않는 것이다. 즉, **코드의 수정을 최소화하여 결함도는 줄이고 응집도는 높이는 효과를 볼 수있는 것이다.**



**적용이슈**

1. 확장되는 것과 변경되지 않는 모듈을 분리하는 과정에서 크기 조절에 실패하면 오히려 관계가 더 복잡해질 수 있다.
2. 인터페이스는 가능하면 변경되어서는 안 된다. 따라서 인터페이스를 정의할 때는 여러 경우의 수에 대한 고려와 예측이 필요하다.
3. 인터페이스 설계에서 적당한 추상화 레벨을 선택해야한다. 행위에 대한 본질적인 정의를 통해 인터페이스를 식별해야 한다.

