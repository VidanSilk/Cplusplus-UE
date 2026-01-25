# Ch4 설계 및 선언 

**<개요>**

소프트웨어 설계하는 챕터 <br>
어떻게 하면 좋은 C++ 인터페이스를 설계하고 선언할 수 있을까? 이번 챕터에는 바로 이 문제에 대해 생각하고 알아보는 것이다. <br>
일단 시작은 어떤 인퍼페이스를 설계하든지 막론하고, 가장 중요할 것 같은 지침인 4-18항목을 바탕으로 **정확성, 효율성, 캡슐화, 
유지보수성, 확장성, 규약 준수**에 이르는 인터페이스 설계에 대한 필요한 모든 것을 제공할 수 없겠지만, 적어도 가장 중요한 몇 가지 고려 사항, 잘못 
사용하고 있는 것에 대해 다시 알려주고, 클래스, 함수, 템플릿을 설계하는데 있어 종종 신경을 건드리는 문제들을 해결하는 방법이 있는 챕터이다. 

**<목차>**
  - [4-18 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-18)
  - [4-19 클래스 설계는 타입 설계와 똑같이 취급하자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-19)
  - [4-20 값에 의한 전달(Call by Value or Pass by Value)보다는 상수객체 참조자에 의한 전달 방식(Pass-by-reference-to-const)을 택하는게 대개 낫다](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-20)
  - [4-21 함수에서 객체를 반환해야 할 경우에 참조자를 반환(return by reference)하려고 들지 말자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-21)
  - [4-22 데이터 멤버가 선언될 곳은 private 영역임을 명심하자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-22)
  - [4-23 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-23)
  - [4-24 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-24)
  - [4-25 예외를 던지지 않는 swap에 대한 지원도 생각해 보자](https://github.com/VidanSilk/Cplusplus-UE/tree/main/CPP/EffectiveCpp/E_Ch4/E_Ch4-25)
