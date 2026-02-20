## 5-27. 캐스팅은 절약, 또 절약! 잊지 말자 

어떤 일이 있어도 **type error**가 생기지 않도록 보장한다. 라는 것은 C++의 동작 규칙이다. 즉, 이론적으로는 컴파일만 잘 되면 그 이후엔 어떤 객체에서도 불완전한 연산이나 말도 안되는 연산등을 수행하지 않는다. 
하지만, C++에서는 **casting** 시스템 때문에 이러한 보장이 깨질 수 있다. 

  - 캐스트 방법
    + C 스타일의 캐스트 (구형 캐스트)
      - (T) 표현식
      - T(표현식) : 함수 방식의 캐스트
    + C++ 스타일의 캐스팅
      - **const_cast<T>(표현식)**
        + 객체의 상수성을 없애는 용도로, cosnt → Non-const로 바뀌는 용도
      - **dynamic_cast<T>(표현식)**
        + 안전한 다운 캐스팅을 할 때 사용, 상속에서 주로 쓰이며, 주어진 객체가 어떤 클래스 상속 계통에 속한 특정 타입인지 아닌지를 결정하는 작업에 사용한다.
        + 런타임 비용이 매우 높다. 
      - **static_cast<T>(표현식)**
        + 암시적 변환(비상수 객체 → 상수객체 or int → double 등)을 강제로 진행할 때 사용
        + 흔히 이루어지는 타입 변환을 거꾸로 수행하는 용도 (void* →  일반 타입의 포인터, 기본 클래스의 포인터 → 파생 클래스의 포인터 등)로도 사용
        + 상수 객체를 비상수 객체로 캐스팅하는 데는 사용할 수 없다.
       

C++ 스타일 캐스트를 쓰는 것이 바람직한 이유는 다음과 같다 <br>
1. 코드를 읽을 때 알아보기 쉽다
2. 소스 코드 어디에서 C++의 타입 시스템이 망가졌는지 찾아보는 작업이 편해진다.
3. 캐스트를 사용한 목적을 더 좁혀서 지정하기 때문에, 컴파일러 쪽에서 사용 에러를 진단할 수 있다.

-----------------------------------------------------------------------------------------

### 캐스팅으로 인해 런타임에 만들어지는 코드 

캐스팅은 그냥 어떤 타입을 다른 타입으로 처리하라고 컴파일러에게 알려주는 것으로만 알고 있는 경우가 많은데, 실상은 그렇지 않다. 
타입 변환시 런타임에서 실행되는 코드가 만들어지는 경우가 꽤 있어 런타임 비용이 필요하기 때문이다. 

```cpp
int x, y;
...
// 부동 소수점 나눗셈을 사용하여 x를 y로 나눈다
double d = static_cast<double>(x) / y;
```

int 타입인 x를 double 타입으로 캐스팅한 부분에서 코드가 만들어지는데, 이는 대부분의 컴퓨터 아키텍처에서 int의 표현 구조와 double의 표현 구조가 아예 다르기 때문이다. 

```cpp
clas Base{...};
clas Derived : public Base {...};

Derived d;
// Derived* -> Base*의 암시적 변환이 이루어진다.
Base *pb = &d;
```

자식 클래스 객체에 대한 부모 클래스 포인터를 만드는 코드이다. 근데 두 포인터 값이 같지 않을 때는 포인터의 Offset를 Derived* 포인터에 적용하여 실제 Base* 포인터 값을 구하는 동작이 런타임에 이루어진다.
위의 예제는 객체 하나가 가질수 있는 주소가 한 개가 아니라 그 이상을 될 수 있음을 보여주는 예시이다. (Base* 포인터로 가리킬 때의 주소와 Derived* 포인터로 가리킬 때의 주소때문에) <br>
다른 프로그래밍 언어(C, Java, C#)에서는 생길 수 없으나 C++에서는 생긴다. 다중 상속이 사용되면 이런 일이 항상 생기겠지만, 단일 상속 인데도 생기는 경우가 있다. <br>
객체의 메모리 배치 구조를 결정하는 방법과 객체의 주소를 계산하는 방법은 컴파일러마다 다 다르기 때문에, 캐스팅을 아주 조심히 써야 한다.  


--------------------------------------------------------------------------------------------

### 캐스팅으로 인해 맞게 보이지만, 틀린 코드 

일단 밑의 예제 코드를 보자. 

```cpp
// 부모 클래스
class Window {

  public:
  // 부모 클래스의 OnResize 구현 결과 
  virtual void OnResize() {...}
  ...
};

// 자식 클래스
class SpecialWindow : public Window {

  public:
  // 자식 클래스의 OnResize 구현 결과 *this를 Window로 캐스팅하고, 그것에 대해 OnResize를 호출한다.
  // 하지만 동작이 안되어서 문제이다. 
  virtual void OnResize() {
    static_cast<Window>(*this).OnResize();
    // SpecialWindow에서만 필요한 작업은 여기서 수해ㅔㅇ한다.
    ...
  }
};

```

Window 클래스에서 가상 함수를 정의하고 있고, 그 가상 함수를 자식 클래스인 SpecialWindow에서 재정의하는데, 재정의할 때 가상 함수에서 Window의 OnResize()함수를 호출하는 부분이다. 
이때 자기 자신 SpecialWindow 객체를 Window로 캐스팅하여 부모 클래스에 있는 Window의 OnResize()함수를 부르고 있다. <br>
이러한 코드는 잘 못 되었는데, 그 이유는 *this의 사본이 임시 객체로 형성되어 Window::OnResize를 호출하기 때문이다. 현재 객체가 호출하는 것이 아닌 임시 객체가 호출하는 방식이기 때문에, 
혹시라도 Window::OnResize에서 객체를 수정하도록 되어있다면 반영되지 않을 것이다. 따라서 **캐스팅을 사용하지말고 부모 클래스 버전을 호출**하도록 만들면 된다.  

```cpp

// 자식 클래스
class SpecialWindow : public Window {

  public:
  // 자식 클래스의 OnResize 구현 결과 *this를 Window로 캐스팅하고, 그것에 대해 OnResize를 호출한다.
  // 하지만 동작이 안되어서 문제이다. 
  virtual void OnResize() {
    Window::OnResize();
    ...
  }
  ...
};

```

--------------------------------------------------------------------------------------------

### Dynamic_cast 연산자의 비용과 대안

상당수의 구현 환경에서 dynamic_cast 연산자는 느리다, 특히 어떤 구현 환경에서는 클래스 이름을 비교함으로써 이 연산자가 구현었기 때문에 깊이가 4인 단일 상속 계통에 속한 어떤 객체에 적용할 때 strcmp가 최대 4번 불리며, 
다중 상속일 경우는 그 정도가 더 심해진다. 따라서 수행 성능이 중요한 코드라면 dynamic_cast 사용에 주의하여야 한다. <br>
하지만 dynamic_cast 연산자를 사용하고 싶을 때가 있을 건데, 자식 클래스 객체라는 것에 확실한 객체가 있어, 이에 대한 자식 클래스의 함수를 호출하고 싶은데, 
그 객체를 조작할 수 있는 수단으로 기초 클래스의 포인터 밖에 없을 경우가 있다. 이럴 경우 문제를 피해갈 수 있는 일반적인 방법 2 가지가 있다.

  - 첫 번째 방법
    + 자식 클래스 객체에 대한 포인터(or 스마트 포인터)를 컨테이너에 담아둠으로써 각 객체를 기본 클래스 인터페이스를 통해 조작할 필요를 아예 없애 버리는 경우.

```cpp
// 하지 말 것! 
class Window {...};

// 자식 클래스
class SpecialWindow : public Window {

  public:
  void blink();
  ...
};

typedef std::vector<std::tr1::shared_ptr<Window>> VPW;
VPW winPtrs;
...

for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
  if(SpecialWindow *psw = dynamic_cast<SpecialWindow*>(iter->get())) {
    psw->blink();
  }
}
```
위의 코드처럼 Window 및 SpecialWindow 상속 계통에서 blink 기능을 SpecialWindow 객체만 지원하게 되어있다면 이렇게 하지말고 
밑의 코드처럼 해보란 것이다. 

```cpp
typedef std::vector<std::tr1::shared_ptr<SpecialWindow>> VPSW;
VPSW winPtrs;
...

// 다이나믹 캐스트를 안해도 된다! 
for (VPSW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
    (*iter)->blink();
  
}
```

이 방법으로는 Window에서 파생될 수 있는 모든 객체들에 대한 포인터를 똑같은 컨테이너에 저장할 수 없다. 즉, 다른 타입의 포인터를 담으려면 타입 안전성을 갖춘 컨테이너 여러 개가 필요할 것이다.

  - 두 번째 방법
    + 원하는 조작을 가상 함수 집합으로 정리해서 기본 클래스에 넣어두면 Window에서 파생한 자식 클래스들 번부 기본 클래스 인터페이스를 통해 조작할 수 있다.
    + Blink 함수가 SpecialWindow에서만 가능하지만, 그렇다고 기본 클래스에 넣을 수 없는 것도 아니다. 즉, 아무것도 안하는 기본 blink 함수를 구현해서 가상 함수를 제공하는 방법이 있다.
   
```cpp
// 부모 클래스 
class Window {

  public:
  // 기본 구현은 아무 동작 안하기
  // 나중에 항목 34에서 가상함수의 기본 구현이 왜 안 좋은건지 확인할 수 있다.
  virtual void blink() {}
  ...
};

// 자식 클래스
class SpecialWindow : public Window {

  public:
  // 이 클래스에서는 blink 함수가 특정한 동작 수행
  virtual void blink() {...}
  ...
};

// 이 컨테이너에서는 Windows에서 파생된 모든 타입의 객체 혹은 포인터를 담는다. 
typedef std::vector<std::tr1::shared_ptr<Window>> VPW;
VPW winPtrs;
...

for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
    (*iter)->blink();
 
}
```

하지만 **폭포식 dynamic_cast는 피하자!**

```cpp
// 부모 클래스 
class Window {...};
... // 자식 클래스들 정의 

// 이 컨테이너에서는 Windows에서 파생된 모든 타입의 객체 혹은 포인터를 담는다. 
typedef std::vector<std::tr1::shared_ptr<Window>> VPW;
VPW winPtrs;
...

for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
  if(SpecialWindow1 *psw1 = dynamic_cast<SpecialWindow1*>(iter->get())) {
  ...
  }
  else if(SpecialWindow2 *psw2 = dynamic_cast<SpecialWindow2*>(iter->get())) {
  ....
  }
  else if(SpecialWindow3 *psw3 = dynamic_cast<SpecialWindow3*>(iter->get())) {
  .....
  }

}
```

자식 클래스가 하나 추가할 때 마다 폭포식 코드에 계속해서 조건 분기문를 추가해줘야 하기 때문에 이러한 형태의 코드를 보면 가상 함수 호출에 기반을 둔 어떤 방법이든 써서 소스 코드를 바꿔야 한다. 

**정말 잘 짜여진 C++ 코드는 캐스팅을 거의 사용하지 않는다** <br>
캐스팅 역시, 그냥 막 쓰기에 꺼림칙한 문법 기능을 써야할 때 흔히 쓰이는 수단을 활용해서 처리하는 것이 좋고, 최대한 격리 시키자. 
캐스팅을 해야 하는 코드는 내부 함수 속에 몰아 놓고, 그 안에서 일어나는 일들은 이 함수를 호출하는 외부에서 알 수 없도록 인터페이스로 막아두는 식으로 해결하면 된다. 

--------------------------------------------------------------------------------------------

### 5-27 정리
1. 다른 방법이 가능하다면, 캐스팅을 피하자! 특히 수행 성능에 예민한 코드에서는 dynamic_cast는 몇 번이고 다시 생각하자, 설계 중에 캐스팅이 필요해졌다면, 캐스팅을 쓰지 않는 다른 방법을 생각해보고 시도해보자.
2. 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길 수 있도록 하자, 이렇게 하면 최소한 사용자는 자신의 코드에 캐스팅을 넣지 않고 이 함수를 호출할 수 있게 된다.
3. 구형 스타일의 캐스트를 쓰려거든 C++ 스타일의 캐스트를 선호하자!, 발견하기도 쉽고, 설계자가 어떤 역할로 의도했는지가 더 자세하게 드러난다.

