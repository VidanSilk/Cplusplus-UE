## 5-26. 변수 정의는 늦출 수 있는데 까지 늦추는 근성을 발휘하자! 

생성자 혹은 소멸자를 끌고 다니는 타입으로 변수를 정의하면 반드시 물게 되는 비용(Cost)이 두 개가 있다. <br>
1. 프로그램 제어 흐름이 변수의 정의에 닿을 때 생성자가 호출되는 비용
2. 그 변수가 유효범위를 벗어날 때 소멸자가 호출되는 비용

변수가 정의됐으나 사용되지 않은 경우에도 비용이 부과되는데, 이런 비용은 웬만한 경우가 아니면 물고 싶은 생각이 안들 것이다.

일단, 비용은 변수가 **정의만** 되어도 부과된다. 예제코드를 보자.

```cpp
// 이 함수는 encrypted 변수를 너무 일찍 정의한 것이다.
std::string encryptPassword(const std::string& password){
  using namespace std;

  string encrypted;
  if(password.length() < MinmumPasswordLenth){
    throw logic_error("Password is too short");
  }
  return encrypted;
}
```

encrypted를 안 쓴다고 할 수 없지만, 예외가 발생되면 이 변수는 사용되지 않지만 encrypted 객체의 생성과 소멸에 대한 비용은 부과된다. <br>
그럼 encrypted의 위치를 옮겨보자.

```cpp
// 이 함수는 encrypted 변수가 진짜로 필요할 때까지 정의를 미룬다. 하지만 여전히 비효율적이다. 
std::string encryptPassword(const std::string& password) {
  using namespace std;

  if(password.length() < MinmumPasswordLenth) {
    throw logic_error("Password is too short");
  }

  string encrypted;
  encrypted = password;

  encrypt(encrypted); //비밀번호 암호화

  return encrypted;
}
```

위의 코드를 보면 예외가 발생할 때는 사용하지 않게 되어, 생성자와 소멸자에 대한 자원 낭비는 사라지지만 encrypted 변수가 생성 될 때 기본 생성자가 호출되고, password를 대입하게 되므로 생성할 때 초기화하는 것보다 효율이 떨어진다. <br>
그럼 어찌하면 성능를 높히고, 프로그램을 깔끔하게 만들 수 있을까? 다음 코드를 보자

```cpp
// 이 함수는 encrypted 변수를 정의하고 초기화하는 가장 좋은 방법
std::string encryptPassword(const std::string& password) {

  if(password.length() < MinmumPasswordLenth) {
    throw logic_error("Password is too short");
  }

  using namespace std;
  string encrypted(password); // 정의와 동시에 초기화, 복사 생성자가 사용

  encrypt(encrypted); //비밀번호 암호화

  return encrypted;
}
```

이렇게 정의와 동시에 복사 생성자를 통해 초기화 함으로써 성능을 높일 수 있고 프로그램도 깔끔해 짐을 알 수 있다.

---------------------------------------------------------------------------------------

그럼 루프에서는? 어떤 변수가 루프에서만 쓰이는 경우라면, 해당 변수를 루프 바깥에서 미리 정의하고 루프 안에서 대입하는게 좋을까? 아니면 루프 안에 변수를 정의하는 방법이 좋을까?... 어떤 구조가 좋은지에 대해 알아보겠습니다. <br>
일단, 방법 A은 루프 바깥쪽에서 정의 한것이다. 

```cpp
// 방법 A, 루프 바깥쪽에서 정의
Widget w;
for (int i = 0; i < n; i++){
  w = i에 따라 달라지는 값; 
}
```

비용은 생성자 1번, 소멸자 1번, 대입 n번이다. <br>
그럼 방법 B는? 루프 안쪽에 정의 하는 것이다. 

```cpp
// 방법 B, 루프 안쪽에 정의

for (int i = 0; i < n; i++){
  Widget w(i에 따라 달라지는 값);
}
```

비용은 생성자 n번, 소멸자 n번이다. 

결과를 보면, 일반적인 경우에는 바깥쪽에 정의하는 것보다 안쪽에 정의하는 것이 변수를 볼 수 있는 유효범위가 좁아지기 때문에, 프로그램 이해도와 유지보수성이 증가하여 안쪽에 정의하는 것이 좋다. 하지만 대입에 들어가는 비용이 생성자, 소멸자 쌍보다 적게 나오고 이러한 성능 하나하나를 민감하게 따져야하는 경우라면 바깥쪽에 정의하는 것도 고려해볼만 하다. <br>
이러한 위의 전제 조건이 아니라면, 앞뒤 볼 것 없이 방법 B(루프 안쪽에 정의) 하는 방법 좋다.  

  - **5-26 정리**
    + 변수 정의는 늦출 수 있을 때까지 늦추자! 프로그램이 더 깔끔해지고 효율이 좋아진다.
