# 4-25. 예외를 던지지 않는 swap에 대한 지원도 생각해보자. 

## std::swap
swap은 초창기 STL에 포함된 이래로, 예외 안전성 프로그래밍에 없어선 안되는 함수로, 자기 대입 현상의 가능성에 대처하기 위한 대표적인 매커니즘 함수이다. 그렇기 때문에 swap을 어떻게 제대로 구현하느냐가 굉장히 중요하다. <br>
std의 표준 swap를 보자.

```cpp

namespace std{
  template<typename T>
  void swap(T& a, T& b) {
    T temp(a);
    a = b;
    b = temp;
  }
}

```

표준 swap은 복사 생성자와 복사 대입 연산자를 통해 이루어진다. 복사만 지원하는 타입이라면 어떤 객체든 맞바꾸기 동작을 수행해주며, 타입에 상관 없이 한 번 호출에 3번의 복사가 일어난다. <br>
a → temp, b → a, temp → b

**한 번의 호출에 3번의 복사가 일어나기 때문에**, 다른 타입의 실제 데이터를 가리키는 포인터가 주성분 타입일 경우 복사 시 손해를 본다. <br>
이런 개념을 설계의 미학으로 끌어올려서 사용하는 기법이 pumpl 관용구이다. (Pointer to implementation라는 뜻으로 Herb Sutter가 처음 정리하여 제안한 용어이다)

```cpp

class WidgetImpl{
private:
  int a, b, c;
  std::vector<double> v; // 아마도 많은 데이터가 있고, 어쩄든 복사 비용이 높음. 
};

///pImpl 관용구를 사용하는 위젯 클래스 
class Widget{
public:
  Widget(const Widget& rhs);
  Widget& operator=(const Widget& rhs){ //위젯을 복사하기 위해, 자신의 WidgetImpl 객체를 복사
    *pImpl = *(rhs.pImpl);
  }

private:
  WidgetImpl* pImpl; //위젯의 실제 데이터를 가진 객체에 대한 포인터. 
};

```

Widget 객체를 직접 맞바꾼다면, pImpl 포인터만 바꾸면 되지만, 표준 라이브러리의 swap은 Widget 객체를 3번 복사하고, 거기에 WidgetImpl 객체도 3번 복사해서 총 6번의 복사를 하게 되는데, 
따라서 Widget 객체를 맞바꿀때는 pImpl 포인터만 맞바꾸라고 std::swap에 알려준다면 비효율적인 문제가 개선된다. 즉, std::swap을 Widget에 대해 특수화를 하는 것이다. 

---------------------------------------------------------------------

## 완전 템플릿 특수화 (Total Template Specialization)

```cpp

namespace std {
  template<>
  void swap<Widget>(Widget& a, Widget& b) {
    swap(a.pImpl, b.pImpl); //private 멤버에 접근 불가로 에러가 발생! 
  }
}

```

위의 코드를 보면, **template<>**는 std::swap의 완전 템플릿 특수화 함수라는 것을 컴파일러에게 알려주는 부분이다. 즉, 완전 특수화가 적용되었지만, pImpl에 접근할 수 없어 에러가 발생한다. 
일반적으로 std namespace의 구성요소는 함부로 변경이 불가능하지만, 사용자가 직접 만든 타입에 대해 표준 템플릿을 완전 특수화하는 것은 가능하다.<br>
결국 private 멤버에 접근이 불가능하기 때문에 에러가 발생함으로 이를 해결하려면, Widget의 멤버 함수 swap을 만들고 std::swap의 특수화 함수에서 멤버 함수를 호출하는 것으로 문제 해결이 가능하다. 

```cpp

class Widget {
public:
  // 아래 완전 특수화된 std::swap에서 멤버 함수를 호출하기 전에, 미리 선언해주기 
  void swap(Widget& other) {
    using std::swap;
    swap(pImpl, other.pImpl);
  }

};

namespace std {
  template<>
  void swap<Widget>(Widget& a, Widget& b) {
    a.swap(b); // widget의 멤버 함수를 호출, 그 대신 멤버 함수의 swap 함수를 신규로 미리 선언해야함. 
  }
}

```

하지만 만약 Widget과 WidgetImpl이 클래스가 아니라, 클래스 템플릿으로 만들어져 있다면 어떻게 될까? <br>
C++은 클래스 템플릿에 대해서는 부분 특수화를 허용하지만, 함수 템플릿(std::swap)에 대해서는 허용하지 않는다. 그리고 std에서는 특수화 제외하고는 아무것도 변경할 수 없는 규칙이 있다. <br>
즉, 다음과 같은 코드는 에러를 뱉는다. 

```cpp

namespace std {
  template<typename T>
  void swap<Widget<T>>(Widget<T>& a, Widget<T>& b) { //에러 발생!
     a.swap(b);
  }
}

```

이에 대한 해결법은 멤버 swap을 호출하는 비멤버 swap을 선언해놓고, 이 비멤버 함수를 std::swap의 특수화 버전이나 오버로딩 버전으로 **선언하지 않으면 된다** <br>
예를 들어 Widget 관련 기능이 WidgetStuff namespace에 들어가 있다고 가정하면 다음 코드처럼 작성하면 된다.

```cpp

namespace WidgetStuff {
  template<typename T>
  class Widget {...};

  template<typename T>
  void swap(Widget<T>& a, Widget<T>& b){ //비멤버 swap, std 네임스페이스에 포함되면 안된다.2
    a.swap(b);
  }
}

```

어떤 코드가 두 Widget 객체에 대해 swap을 호출하더라도, 컴파일러는 C++의 이름 탐색 규칙(argument-dependent lookup or Koenig lookup)에 따라 WidgetStuff namespace 안에서 Widget 특수화 버전을 찾아내서 사용한다. <br>
이 방법은 클래스 템플릿 뿐ㄴ만 아니라, 클래스에 대해서도 잘 통하므로 sawp을 특수화하고 싶다면 클래스와 동일한 namespace 안에 비멤버 버전의 swap을 만들고, std::swap의 특수화 버전도 준비하면 된다.

**argument-dependent lookup or Koenig lookup**이란? <br>
우선 인자 기반 탐색(argument-dependent lookup)은 어떤 함수에 어떤 타입의 인자가 있으면, 그 함수의 이름을 찾기 위해 해당 타입의 인자가 위치한 네임스페이스 내부의 이름을 탐색해 들어간다는 규칙이다 흔히 ADL라고도 한다. <br>
쾨니크 탐색(Koenig lookup)은 C++ 표준화 위원회 임원인 앤드류 쾨니크의 이름에서 따온걸로 TC++PL라는 책을 보거나, 구글링을 해보자. 참고로, 2.95 이상의 gcc는 ADL를 완벽하게 따르고, 7/7.1 미만의 MSVC는 연산자 함수에 대해서만 ADL을 따른다. 

마지막으로, swap 호출 순서 제어에 대해 알아보자. <br>
swap 호출 순서 제어는 다음과 같은 순서로 이루어진다. 
1. T타입 전용 swap (전역 유효 범위 혹은 동일 네임스페이스 안에 있는 swap부터 찾는다)
2. std::swap의 특수화 버전을 호출한다.
3. std::swap 표준 버전을 호출한다.

```cpp

template<typename T>
void doSomething(T& obj1, T& obj2) {
  using std::swap; // std::swap을 이 함수 안으로 끌어올 수 있도록 한다.
  swap(obj1, obj2); // T타입 전용 swap을 호출한다. 
}

```

조심할 점은 멤버 swap은 **절대 예외를 던지면 안 된다.** swap을 쓸모있게 응용하는 방법들 중, 클래스 및 클래스템플릿이 강력한 예외 안전성 보장을 제공하도록 도움을 주는 방법이 전제이기 때문이다. <br>

### 4-25 정리.
  - std::swap이 여러분의 타입에 대해 느리게 동작할 여지가 있다면, swap 멤버 함수를 제공하고, 이 멤버 swap은 예외를 던지지 않도록 만들어야한다.
  - 멤버 swap을 제공했으면, 이 멤버를 호출하는 비멤버 swap도 제공하자. 클래스(템플릿 X)에 대해서는 std::swap도 특수화하자.
  - 사용자 입자엥서 swap을 호출할 때는, std::swap에 대한 using 선언을 넣어준 후에 네임스페이스 한정 없이 swap을 호출하자.
  - 사용자 정의 타입에 대한 std 템플릿을 완전 특수화 하는 것은 가능하다. 그러나 std에 어떤 것이든 새로 추가하려고 하지말자. 
