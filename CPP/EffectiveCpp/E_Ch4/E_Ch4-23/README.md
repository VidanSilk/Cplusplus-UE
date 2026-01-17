# 4-23. 멤버 함수보다는 비멤버, 비프렌드 함수와 더 가까워지자. 

```cpp
class WebBrowser{
  public:
    void clearCache();
    void clearHistory();
    void removeCokies();
};

//멤버 함수 - 한번에 처리
class WebBrowser{
  public:
    void clearEveryThing();
};

//비멤버 함수
void clearBrowser(WebBrowser& wb){
  wb.clearCache();
  wb.clearHistory();
  wb.removeCokies();
}

```

코드를 보면, 멤버 함수와 비멤버 함수중 어떤 쪽이 캡슐화가 잘 되어 있는가? <br>
바로 비멤버 함수이다. 그럼 비멤버 함수의 특성이 뭘까? <br>
1. 패키징 유연성
2. 컴파일 의존도 감소
3. 더 나은 확장성 제공

캡슐화는 이미 있는 코드를 바꾸더라도, 제한된 사용자들 밖에 영향을 주지 않는 융통성을 확보가 가능하다. <br>
즉, 멤버 변수에 접근할 수 있는 함수의 개수가 적을수록 캡슐화가 잘 되어 있고, 비멤버 비프랜드 함수는, 어떤 클래스의 private 멤버 부분을 접근할 수 있는 함수의 개수를 늘리지 않기 때문에 캡슐화가 더 잘 되어있다.<br>
그렇다고해서, **함수는 어떤 클래스의 비멤버가 되어아한다**는 주장이 **그 함수는 다른 클래스의 멤버가 될 수 없다**는 의미가 아니다 <br>

- 캡슐화
  + 어떤 것을 캡슐화하면, 외부에서 이것을 볼 수 없다. 캡슐화를 하면 밖에서 볼 수 있는 것들이 적기 때문에, 다른 것들을 바꿀 때 필요한 유연성이 커진다.
  + 즉, 해당 객체의 데이터에 접근할 수 있는 함수가 많을수록 캡슐화 정도가 낮다고 할 수 있다.

C++에서는 이를 namespace로 구현이 가능하다.

```cpp
namespace WebBroserStuf{

  class WebBroser {...};
  void clearBrowser(WebBrowser& sb);
}
```

WebBroswer처럼 응용도가 높은 클래스는 이런 종류의 편의 함수가 꽤 생길 수 있다. <br>
즐겨찾기, 인쇄, 쿠기 관리용 함수등등...<br>
그럼, 이러한 함수들을 깔끔하고 쉬운 방법으로 해결하는 방법이 있을까? <br> 
헤더파일에 몰아서 선언하면 된다. 

```cpp
//WebBroser.h에 선언!
// 핵심 기능들이 있는 헤더파일 
namespace WebBroserStuf{
// 핵심 관련 기능, 거의 모든 사용자가 써야하는 비멤버 함수들은 여기에 들어간다.
  class WebBroser {...};
  void clearBrowser(WebBrowser& sb); 
}

//WebbroserBookmarks.h
namespace WebBorswerStuff{
  // 북마크와 관련된 함수들을 선언하는 곳. 
}

//WebbrowserCookies.h
namespace WebBorswerStuff{
  // 쿠키와 관련된 함수들을 선언하는곳. 
}
```

사실, 표준 C++라이브러리가 이런 구조로 되어 있고, 이렇게 사용하면 사용자가 필요한 기능에 대해서만 include를 하여 사용이 가능하고, 사용자는 실제로 사용하는 구성요소에 대해서만 컴파일 의존성을 고려할 수 있지만, 
멤버 함수로 사용하게 되면 기능을 쪼개는 것이 불가능하므로, 하나의 클래스는 그 전체가 통으로 정의되어야 한다. <br>
그리고 여러 개의 헤더 파일에 나누어놓으면 편의 함수 집합의 확장도 쉽게할 수 있다. 

4장 23절 정리
  - 멤버 함수 보다는 비멤버 비프렌드 함수를 자주 쓰자, 캡슐화 정도가 높아지고, 패키징 유연성이 커지며, 기능 확장성도 늘어난다.
