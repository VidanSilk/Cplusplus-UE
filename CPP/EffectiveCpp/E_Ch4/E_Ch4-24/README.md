# 4-24. 타입 변환이 모든 매개변수에 대해 적용되어야 한다면, 비멤버 함수를 선언하자.

암시전 변환 타입을 지원하는 것은 좋지 않다고 한적이 있을거다. <br>
하지만 예외 케이스도 있는데 대표적으로 숫자 타입을 만들 때 이다.
유리수를 나타내는 클래스를 만들고 있다면, 정수에서 유리수로의 암시적 변환은 허용해도 크게 이상하지 않다. 

```cpp
class Rational{
public:
  Rational(int num = 0, int denmoinator = 1); // 암시적 변환을 위 explict를 붙이지않음, 

  int num() const
  int denmoinator () const;

private:
  ....

};

```

int에서 Rational로 암시적 변환은 허용한다. 그럼, 기본 연산을 지원하기 위해서 멤버 함수, 비멤버 함수, 비멤버 프렌드 함수 중 어떤 것이 좋을까? 

일단 operator*를 멤버함수로 선언했을 때를 생각하자.

```cpp
class Rational{
public:
  ...
  const Rational operator*(const Rational& Rhs) const;
};

Rational oneEnglish(1, 8);
Rational oneHalf(1,2);

Rational result = oneHalf * oneEnglish; //Ok
result = result * oneEnglish; // Ok

result = oneHalf * 2; // Ok 
result = 2 * oneHalf; // Error
/*
result = oneHalf * 2;
result = 2 * oneHalf;
를 다시 풀어서 작성해보면 
*/

result = oneHalf.operator*(2);
result = 2.operator*(oneHalf);
```

가장 마지막 부분을 보면 result = 2.operator*(oneHalf);에서 2는 클래스가 연관되지 않았기 때문에 operator를 호출할 수 없다. <br>
즉, 혼합형 수치 연산을 지원 못하게 된다. <br>
그러나 result = oneHalf.operator*(2);는 왜 에러가 발생하지 않고, 될까? 분명 operator*연산의 매개변수로는 Rational을 받고 있는데, 2를 대입해서 넣어줘도 제대로 동작하고 있기 때문이다.  <br>
이를 암시적 변환 타입에 의해서 일어나는데, 컴파일러가 함수에 int를 넘겨받았지만, Rational을 요구한다는걸 알고 int를 Rational 클래스 생성자에 넣고 호출하여 Rational으로 둔갑 시킨 것이다. 

```cpp
const Rational temp(2);
result = oneHalf * temp; 
```

물론 생성자에 명시적 호출(explicit)를 했다면, oneHalf * 2도 에러가 떴을거다. <br>
결과적으로, 암시적 타입 변환에 대해 매개변수가 먹히려면 매개변수 리스트가 있어야하며, 호출되는 멤버 함수를 갖고 있는(this가 가리키는)객체에 해당 암시적 매개변수에는 암시적 변환이 먹히지 않는다. <br>
이 문제를 해결하기 위해, operator*를 비멤버 함수로 만들어 컴파일러 쪽에서 모든 인자에 대한 암시적 타입 변환을 수행하도록 하면 혼합형 수치 연산을 지원할 수 있다.

```cpp
const Rational operator*(const Rational& lhs, const Rational& rhs){
  return Rational(lhs.num() * rhs.num(), lhs.denmoinator() * rhs. denmoinator());
}

Rational oneFourth(1,4);
Rational result;

result = oneFourth * 2; //Ok
result = 2 * oneFourth; //Ok
```

그리고 **멤버 함수의 반대는 프렌드 함수가 아니라, 비멤버 함수**라는걸 꼭 기억하자.

4장 24절 정리
  - 어떤 함수가 들어가는 모든 매개변수(this가 가리키는 객체도 포함해서)에 대해 타입 변환을 해줄 필요가 있다면, 그 함수는 비멤버여야 한다. 

