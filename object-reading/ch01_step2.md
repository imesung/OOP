**모든 소프트웨어 모듈에는 세가지 목적이 있다.**

1. 실행 중에 제대로 동작하는 것.
2. 변경을 위해 존재하는 것.
3. 코드를 읽는 사람과 소통하는 것.



**Step 1**

<img src="https://user-images.githubusercontent.com/40616436/81817803-28b04700-9568-11ea-94ef-421c3e21267d.png" alt="image" style="zoom:50%;" />

~~~java
public class Theater {
  private TicketSeller ticketSeller;

  public Theater(TicketSeller ticketSeller) {
    this.ticketSeller = ticketSeller;
  }

  public void enter(Audience audience) {
    if (audience.getBag().hasInvitation()) {
      Ticket ticket = ticketSeller.getTicketOffice().getTicket();
      audience.getBag().setTicket(ticket);
    } else {
      Ticket ticket = ticketSeller.getTicketOffice().getTicket();
      audience.getBag().minusAmount(ticket.getFee());
      ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
      audience.getBag().setTicket(ticket);
    }
  }
}
~~~

- 소극장(Theater)가 관람객, 판매원의 위치를 침범하고 있다. 관람객의 가방에서 돈을 꺼내고, 판매원의 매표소에서 티켓과 현금을 마음대로 접근하고 있다.
- **즉, 하나의 클래스의 너무 많은 책임을 가지고 있다. 또한 Audience와 Ticket의 변경이 있을 시 Theater도 변경이 이루어진다.**
- 위의 구조는 **Theater는 관람객이 가방을 들고 있고 판매원이 매표소에서만 티켓을 판매한다는 지나치게 세부적인 사실에만 의존해서 동작한다.** 즉, Audience나 Ticket의 내부 구조를 Theater가 너무 많이 알고 있어 약간의 구조만 바뀌어도 많은 부분을 변경해야한다는 것이다. 이 얘기는 **의존성**과 관련된 이야기이고, 의존성은 어떤 객체가 변경할 때 그 객체에게 의존하는 다른 객체도 함께 변경될 수 있다는 개념이 내제되어 있다.



**Step 1 - Theater 포함 구조**

<img src="https://user-images.githubusercontent.com/40616436/81819708-a4ab8e80-956a-11ea-9e63-b5cdda2243a5.png" alt="image" style="zoom:50%;" />

- 객체 사이의 의존성이 과한 경우 **결합도** 가 높다고 말한다. 즉, 결합도가 높으면 높을 수록 객체의 변경이 다른 객체에 영향을 끼치는 경우가 많아지는 것이다.



**Step 2**

- 해당 코드가 어려운 이유는 Theater의 책임이 Audience의 책임과 TicketSeller의 책임까지 가지고 있고, Theater가 Audience와 TicketSeller의 내부를 너무 세세하게 알고 있어서 그런 것이다. 해결방법은, Theater가 Audience와 TicketSeller의 내부를 세세하게 알지 못하도록 차단하고, 책임을 분배하면 되는 것이다. 다시 말해, Audience와 TicketSeller를 **자율적인 존재**로 만드는 것이다.

**TicketSeller의 자율**

- Step1에서 TicketSeller는 Theater에서 접근하고 Theater는 TicketSeller의 TicketOffice에 접근하여 Ticket들을 가지고 왔었다. 하지만 TicketSeller를 자율적인 존재로 만들어줌으로써, TicketSeller가 의존하고 있던 TicketOffice도 외부에서 호출하는 이유가 사라지기 때문에 TicketSeller에서 TicketOffice를 호출하는 getTicketOffice() 메소드를 지웠다.

  ~~~java
  public class TicketSeller {
    private TicketOffice ticketOffice;
  
    public TicketSeller(TicketOffice ticketOffice) {
      this.ticketOffice = ticketOffice;
    }
  
    public void sellTo(Audience audience) {
      ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
  }
  
  //Theater - TicketOffice 접근 부분 없음. Theater는 TicketOffice의 존재 여부를 알지 못함.
  public class Theater {
      private TicketSeller ticketSeller;
  
      public Theater(TicketSeller ticketSeller) {
          this.ticketSeller = ticketSeller;
      }
  
      public void enter(Audience audience) {
          ticketSeller.sellTo(audience);
      }
  }
  ~~~

  - 즉, TicketOffice를 외부에서 호출하는 부분은 모두 지워지고 TicketOffice는 단지 TicketSeller에서만 다루어 진다. 이것을 객체 내부의 세부사항을 감추고 외부에서 접근하는 것을 방지하는 것인데, 이것이 바로 **캡슐화**이다. 
  - 어떻게 보면, Theater는 TicketSeller라는 인터페이스를 의존하고 있고, TicketSeller를 구현하는 영역은 TicketOffice로 볼 수 있는 것이다.

  <img src="https://user-images.githubusercontent.com/40616436/81821706-21d80300-956d-11ea-8389-b0052d56263b.png" alt="image" style="zoom:50%;" />
  - Theater와 TicketOffice의 의존관계가 제거된 것을 볼 수 있다. 즉, TicketOffice와 협력하는 TicketSeller의 내부 구현이 성공적으로 캡슐화 된 것이다.



**Audience의 자율**

- Audience도 마찬가지로 Theater에서 Bag에 접근하는 모든 로직을 Audience 내부로 감추고 TicketSeller의 sellTo 메소드에 의해 getBag 메소드에 접근하는 buy 메소드에 해당 로직을 옮기는 것이다.

  ~~~java
  public class Audience {
    private Bag bag;
  
    public Audience(Bag bag) {
      this.bag = bag;
    }
  
    public Long buy(Ticket ticket) {
      if (bag.hasInvitation()) {
        bag.setTicket(ticket);
        return 0L;
      } else {
        bag.setTicket(ticket);
        bag.minusAmount(ticket.getFee());
        return ticket.getFee();
      }
    }
  }
  ~~~

  - 해당 소스의 결과를 살펴보면 Audience는 자신의 가방에 초대장이 있는지 스스로 확인하게 된다. 이전처럼 제 3자가 자신의 가방을 열어보지 않다는 것이다. 또한, 이제 Audience에서 Bag을 직접 처리하므로 Bag은 외부에서 접근불가한 상태가 되어도 된다는 것이고, 이것 또한 캡슐화가 되는 것이다.

  <img src="https://user-images.githubusercontent.com/40616436/81823635-754b5080-956f-11ea-9db5-18f2d9d51703.png" alt="image" style="zoom:50%;" />

  

**개선사항**

- 이로 인해 Audience나 TicketSeller는 코드 변경에 매우 유연하게 변한 것을 확인할 수 있을 것이다. 계산 방법이 바뀌거나 장소가 바뀔 시 해당 클래스에서만 수정이 진행되면 된다는 말이다!! **변경에 매우 용이해졌다.**
- 또한, Audience나 TicketSeller는 자신이 처리하는 클래스들은 외부로부터 숨겨두고 자신만 처리할 수 있도록 변경하여 Theater가 최소한의 것만 접근할 수 있도록 하였다. 이 말은, Audience나 TicketSeller나 내부에 있는 클래스들의 사소한 변경에도 Theater는 영향을 안 받는다는 말이다. **즉, 객체의 자율성이 높아졌다는 이야기이고, 이로 인해 변경에도 다른 소스에 영향을 크게 주지 않는다는 것이다.**
- 우리는 Bag과 TicketOffice를 Audience와 TicketSeller에게 숨김으로써, Theater는 밀접하게 연관된 작업(Audience와 TicketSeller)만 수행하고 연관성 없는 Bag과 TicketOffice의 작업은 다른 객체인 Audience와 TicketSeller에게 넘긴 것을 보았다. **이런 구조를 응집도가 높다고 말하는 것이다.** 즉, **자신의 데이터를 스스로 처리하는 자율적인 객체를 만들면 결합도를 낮출 뿐더러 높은 응집도를 구성할 수 있는 것이다.**



**절차지향과 객체지향**

- 우리는 개선하기 전에 Theater의 enter 메소드에서 Audience와 TicketSeller로부터 Bag과 TicketOffice를 가져와 관람객을 입장시키는 절차를 구현했다. 이 때, Theater enter 메소드는 **프로세스**이고, Audience, TicketSeller, Bag, TicketOffice는 **데이터**이다. 이 처럼 프로세스와 데이터를 별도의 모듈에 위치시키는 방식을 **절차적 프로그래밍** 이라고 부른다. 절차적으로 부르는 이유는, 모든 처리가 하나의 클래스 안에 위치하고 나머지 클래스는 단지 데이터의 역할만 수행하기 때문이다. 그리고 위에서 살펴보았듯이 절차적 프로그래밍은 변경에 유연하지 않는다는 큰 단점을 가지고 있다는 사실을 알고 있어야 한다.
- 절차지향적 프로그래밍을 객체지향적 프로그래밍으로 변경하는 방법은 자신의 데이터를 스스로 처리하도록 프로세스의 적절한 단계를 Audience와 TicketSeller로 이동시키는 것이다. **즉 자신의 책임에 해당되는 데이터와 프로세스를 자신의 객체 내부에 위치하도록 하는 프로그래밍을 객체지향 프로그래밍이라고 하는 것이다.**
- **객체지향 프로그래밍의 특징을 한 문장으로 표현하면,,,,** **캡슐화를 이용해 의존성을 적절히 관리함으로써 객체 사이의 결합도를 낮춰주고 응집도를 높여주고, 자신의 일만 자율적으로 책임지게 되므로 변경도 외부에 영향을 안 주는 것이다.**

- 절자지향은 책임이 Theater에게만 집중되어 있다.

  <img src="https://user-images.githubusercontent.com/40616436/81829029-70899b00-9575-11ea-872b-7727a5294a5c.png" alt="image" style="zoom:50%;" />

- 객체지향은 책임이 분리되어 자신이 맡은 일만 하는 것을 볼 수 있다.

  <img src="https://user-images.githubusercontent.com/40616436/81829485-e5f56b80-9575-11ea-81a3-cce89e4e282e.png" alt="image" style="zoom:50%;" />

- 객체지향적으로 프로그래밍 하고 싶은가? **그러면 데이터와 데이터를 사용하는 프로세스가 동일한 객체 안에 위치하도록 구현해라!**



**Step 3**

**Audience에 있는 Bag을 자율적으로 만들어보자**

- Bag의 내부상태에 접근하는 모든 로직을 Bag 안으로 캡슐화해서 결합도를 낮추는 것이다. (Audience에 있는 buy 메소드의 로직을 Bag 내부로 옮기자.)

  ~~~java
  public class Bag {
    private Long amount;
    private Ticket ticket;
    private Invitation invitation;
  
    public Long hold(Ticket ticket) {
      if (hasInvitation()) {
        setTicket(ticket);
        return 0L;
      } else {
        setTicket(ticket);
        minusAmount(ticket.getFee());
        return ticket.getFee();
      }
    }
  
    private void setTicket(Ticket ticket) {
      this.ticket = ticket;
    }
  
    private boolean hasInvitation() {
      return invitation != null;
    }
  
    private void minusAmount(Long amount) {
      this.amount -= amount;
    }
  }
  ~~~

- 기존 public 메소드였던 hasInvention과 minusAmount, setTicket은 더 이상 외부에서 접근하지 않기에 private 메소드로 변경하여 구현을 캡슐화 시켰다. 그럼 이제 Audience는 Bag의 hold 메소드만 접근하여 로직을 구성한다.

  ~~~java
  public class Audience {
    private Bag bag;
  
    public Audience(Bag bag) {
      this.bag = bag;
    }
  
    public Long buy(Ticket ticket) {
      return bag.hold(ticket);
    }
  }
  ~~~

- TicketSeller 역시 TicketOffice의 자율성을 침해하고 있으므로, 이 부분도 분리해주자.

  ~~~java
  public class TicketSeller {
    private TicketOffice ticketOffice;
  
    public TicketSeller(TicketOffice ticketOffice) {
      this.ticketOffice = ticketOffice;
    }
  
    public void sellTo(Audience audience) {
      //TicketOffice의 유일한 public 메소드인 sellTicketTo만 접근 가능.
      ticketOffice.sellTicketTo(audience);
    }
  }
  
  public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();
  
    public TicketOffice(Long amount, Ticket... tickets) {
      this.amount = amount;
      this.tickets.addAll(Arrays.asList(tickets));
    }
  
    public void sellTicketTo(Audience audience) {
      plusAmount(audience.buy(getTicket()));
    }
  
    private Ticket getTicket() {
      return tickets.remove(0);
    }
  
    private void plusAmount(Long amount) {
      this.amount += amount;
    }
  }
  ~~~

  - TicketOffice 역시 구현에 관련된 부분은 모두 private 메소드로 선언하여 캡슐화를 진행하고 있는 것을 볼 수 있다.
  - 하지만, 소스를 잘 살펴보면 TicketOffice가 분리로 인해 Audience를 의존하고 있는 것을 볼 수 있다. (sellTicektTo(Audience)) 즉, TicketOffice는 Audience에 대해서 알아야한다는 것이다. 이것은 의존성이 추가된 것인데, 의존성이 추가되면 결합도는 높아진 것이고, 결합도가 높아지면 변경도 어려워진다는 설계가 된다.. 이래도 우리는 분리를 해야하는가?

  <img src="https://user-images.githubusercontent.com/40616436/81831545-716ffc00-9578-11ea-94ab-ad0cb4a16ada.png" alt="image" style="zoom:50%;" />

  - 위의 상황은, **Audience에 대한 결합도와 TicketOffice의 자율성을 모두 만족시키기에는 애매하다는 것이다.**
  - 이럴 때, 선택하는 것이 트레이드오프이다. 즉, 어떤 경우에도 모든 사람들을 만족시킬 수 있는 설계는 없고, 설계는 트레이드오프의 산물이라는 것이다. 협의를 통해 둘 중 하나를 선택해라!



**모든이에게 생명을 부여해라. 물건도! 너희는 능동적으로 행동할 수 있다.**

- 우리가 지금까지 설계한 것들을 보면 Audience와 TicketSeller를 제외하고도 Bag이나 TicketOffice에게 자율성을 주려고 노력했다. 이것은 어떤 말인가..? 모든 객체는 능동적으로 행동할 수 있다는 것이다. **객체지향 세계에 들어오면 물건이나 건물도 생물처럼 능동적으로 행동할 수 있게 된다. 즉, 능동적이고 자율적인 존재로 바뀌는 것이다.**
- **훌륭한 객체지향 설계는 모든 객체들이 자율적으로 행동하도록 하는 설계이다. 다른 방향으로는 오늘 요구하는 기능을 온전히 수행하되 내일의 변경이 매끄럽게 진행되는 설계이다.** 
- 자율적으로 행동하며 변경을 수용할 수 있는 설계가 중요한 이유는 무엇일까? **바로 버그를 줄이기 위함이다.** 버그는 소스를 변경하지 않는 이상 발생하지 않을 것이다. 즉, 변경에 의해 다른 소스를 수정하는 상황이 발생한다면 버그 발생률은 점점 높아지고 범위도 넓어지게 될 것이다.



>  **진정한 객체지향설계는, 객체가 자율성을 가지고 있어 해당 객체의 변경으로 다른 객체가 영향을 받지 않는 설계 방식이다.**
>
> **즉, 객체는 책임 단위로 분리되어야 한다는 이야기인데, 객체지향 세계에서 가장 중요한 것은 객체들의 상호작용이고, 이 상호작용은 서로 주고 받는 메시지로 표현된다는 사실을 꼭 기억해야한다.**
>
> **이런 책임과 메시지로 인해 객체는 자율적이면서 어느 정도의 의존서이 필요한 것은 불가피하다. 이로 인해 우리는 훌륭한 객체지향 설계를 하려면 객체 사이의 의존성을 적절하게 관리하는 것이 베스트인 것이다.**