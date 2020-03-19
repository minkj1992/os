# [ch02] 유니코드와 WBCS

## 유니코드(WBCS)
> Wide Byte Char Set

1. `wchar_t` 2바이트(char -> wchar_t) 
2. L"ABC" 유니코드로 명시
3. 흔한 에러
```c
int main(void)
{
    wchar_t str[] = L"ABC";
    int size = sizeof(str);
    int len = strlen(str);   --> wcslen(str)
... 
}
```

4. main -> wmain( wide main )을 사용
이때 char *argv[] ->wchar_t *argv[]

## MBCS WBCS 동시지원
> 기존 legacy들 때문에 WBCS만 쓰는 경우는 희귀하다.

- 버전이 2가지 형식으로 나눠진다는 것은 유지보수의 어려움을 말한다.
- 윈도우에서는 이를 헤결하기 위해서 도움을 준다.

<windows.h>
