# [CH17]구조적 예외처리(SEH)

- `시스템` = 내부(CPU + OS) + 외부(APP)

## 예외
CPU를 디자인할 때,몇가지 규칙들이 있다. 예를 들면  8byte / 4byte 연산은 불가능한데, 왜냐면 CPU는 동일한 바이트의 크기만 연산하기 떄문이다. 

만일 CPU가 32bit라면, 기본 연산 단위는 4byte이다. 그 내부적으로 32비트로 변환해서 연산한다. 또 다른 예시로는 0으로 나누라는 예외 적인 상황이다. 이 같은 경우 무한정 나누기를 반복하다 전자회로에 손상을 입게 될 것이기 때문에 cpu가 자체적으로 연산을 불가능하게 규정지었다. 

이처럼 하드웨어 상에서 예외로 미리 정해둔 특정 연산들이 존재한다. 이와 마찬가지로 운영체제 관점에서 예외는 하드웨어 예외를 감싼 것 + 운영체제가 정해 놓은 예외가 존재하며, 마지막으로 application단에서도 low level에서 제공하는 예외들과 + application이 규정한 예외상황들이 존재하게 된다.

## SEH(Structured Exception Handling)
> 구조적 예외 처리 기법, 코드 단위에서 예외처리 영역과 실제 코드 흐름 영역을 분리할 수 있다.
- 특징
	- `try ~ except()` 느낌 ( 실제 코드와 예외처리 영역을 나눌 수 있다.)
		- c에는 예외처리 기법이 없다. 이런 코드 블록 나누는 기법이 가능한 이유는 윈도우가 해당 기능을 제공하기 때문이다. 소프트 웨어 예외 발생시 windows는 예외처리 메커니즘을 동작시키고, 하드웨어 예외가 발생할 때도 window에 의해서 예외가 핸들링된다. ( 모든 예외를 windows가 중간에서 처리)
	- `종료 핸들러`
		- try {}  ~ finally {}
		- 한번이라도 try 안 코드블록에 진입하면 finally 코드블록에 접근한다. 
	- `예외 핸들러`(__except)
		- try {} ~ except {}

하드웨어에서 나오는 예외 신호를 os가 받아들이고, 그것을 app에 전달하는데 바로 그 예외를 전달하는 부분(OS와 APP 사이에 예외에 대한 접점)에 `SEH`가 존재합니다.

