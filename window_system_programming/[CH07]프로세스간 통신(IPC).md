# [CH07][CH08] 프로세스간 통신(IPC)

일반적으로 1program 당 1process이지만, 1:n 관계 일때도 많다. 프로그램에 속한 여러 프로세스들은 통신을 통해서 소통해야 한다.

## IPC
- 프로세스간 데이터 송/수신
    - Shared Memory
    - Msg Passing
    - 
- 안전성을 위해서  프로세스 간 메모리 참조는 이뤄질 수 없다 (OS가 block)
- 이를 해결하기 위해서는 OS를 통하여 통신이 이루어져야 한다.


## 메일슬롯
- `sender`프로세스와 `Receiver`프로세스 사이에 `우체통`을 만들어 둔다.
- 우체통은 함수호출을 통해서 간접 호출만 가능하다.
- 단방향 특성을 지닌다.(sender가 receive할 수없다.)
- sender : receiver = n:n
- 윈도우, 리눅스등 모든 os에 구현되어있다.
- 대부분의 운영체제에서 메일슬롯을 file을 활용하여 구현해둔다.
    - 이로 인하여, mail slot file을 생성하고 나면 os API를 fil API를 통해서 전송/수신/연결 등을 처리한다.
- 로컬 이외에도 사용할 수 있다. 그러나 잘 사용하진 않는다. TCP/IP를 쓰는게 훨씬 명확하고 다양한 기능을 사용할 수 있기 때문에


- 소스코드 7-2
```c
// 메일슬롯 Receiver
#define SLOT_NAME _T("PC이름")
int _tmain(int argc, LPTSTR argv[])
{
    HANDLE hMailSlot; //mailslot  핸들
    TCHAR messageBox[50];
    hMailSlot = CreateMailslot(
        SLOT_NAME,    // 슬롯 이름
        0,            // 메일슬롯 버퍼크기, 0은 허용최대치
        MAILSLOT_WAIT_FOREVER    //ReadFile함수 특성
        NULL);    // 보안 설정
    ReadFile(
        hMailSlot,     //슬롯지정
        messageBox,    //데이터 수신버퍼(배열지정)
        sizeof(TCHAR)*50    // 읽어 들일 데이터 최대크기
        &bytesRead        //읽어들인 데이터 크기
        NULL);        // 일단 null
}
```

- 소스코드 7-3
```c
// 메일슬롯 Sender
#define SLOT_NAME _T("PC이름")
int _tmain(int argc, LPTSTR argv[])
{
    HANDLE hMailSlot;    //mailslot핸들
    hMailSlot = CreateFile(
        SLOT_NAME,    //연결 슬롯 이름
        GENERIC_WRITE,    //메일슬롯 용도
        FILE_SHARE_READ,
        NULL,
        OPEN_EXISTING,//생성방식(메일박스가 있는데 새로생성하면 큰일난다)
        FILE_ATTRIBUTE_NORMAL,
        NULL);
    WriteFile(
        hMailSlot,
        message,    //전송데이터 버퍼
        _tcslen(message)*sizeof(TCHAR)    //전송크기
        &bytesWritten,    //전송결과
        NULL);
CloseHandle(hMailSlot);
}
```

## c.f) 커널 오브젝트의 상태

- 커널 오브젝트는 boolean 상태값을 지닌다.
    - 프로세스를 예로들면
        - True (`signaled`): 프로세스가 종료됨
        - False(`non-signaled`): 프로세스가 현재 실행중
    - 이렇듯 각각의 리소스별로 커널 오브젝트의 상태값은 다른 유의미한 의미를 지닌다. ( 모두 익혀야 한다 )

- `커널 오브젝트 상태` 확인방법
    - 상태관찰 시나리오
    - `waitForSingleObject()`: 싱글/non싱글  여부 확인
        - single 즉 프로세스가 종료되었는지 물어보고,  종룓되기까지 polling하는 함수

- 커널 오브젝트의 상태는 waitForSigleobject()를 통하여 프로세스 진행을 control하기 위해서 필요하고, 만약 타 프로세스가 끝나지 않은 상태라면 polling 하면서 blocking 되기 때문에, 멀티 스레딩 환경에서도 동시성 흐름 control이 가능하다.


## 프로세스 환경변수
- 각 프로세스는 고유한 메모리 블록을 가지고 있고 그 안에 환경변수 테이블이 존재한다.
- 부모/자식간 sharing이 가능하긴 하지만, 주 관점은 프로세스마다 자기만의 configuration을 저장하고 뺐다 하기 위한 데이터 블록
- 하지만 이번시간에 IPC 관점이 부가적인 기능이지만, 다뤄보도록 하겠다


- 환경변수의 구성
    - key,value string으로 구성된 lookup table
    - `SettEvironmentVariable("키","값")

- 환경변수 IPC 통신법
    - 부모 프로세스에서 SetEnvironmentVVariable()
    - CreateProcess의 6번째 인자를 null로 전달
    - 이는 부모 프로세스의 환경변수를 등록한다는 뜻
    - 자식 프로세스에서 GetEnvironmentVVariable()


## 핸들 테이블과 오브젝트 핸들의 상속

- Handle table
    - handle
    - 주소
    - 상속여부

부모의 프로세스는 자식 프로세스에게 핸들테이블을 상속할 수 있는데, 이때 조건은 부모 프로세스의 핸들 테이블에 상속여부 칼럼의 값이 True일 때만 가능하다. 부모 핸들테이블에 존재하는 수많은 커널 오브젝트에 대한 참조중에서 상속엽가 True로 처리 되어있는 필드만 자식 핸들테이블에 상속 된다.


사실 위의 과정을 거치기 이전에, CreateProcess의 5번째 인자를 통하여 자식을 생성할때 핸들테이블 상속 여부를 체크할 수 있다. 만약 이게 False라면 비어진 핸들 테이블에서 출발할 것이다.

receiver는 mail slot을 생성한다. 


- Pseudo 핸들과 핸들의 중복
    - 가짜 핸들
    - 핸들테이블에 등록되지 않은 핸들은 모두 가짜 핸들이다.
    - 자기 자신을 가리키는 핸들은 상수값으로 가짜 핸들이다.
    - 상황: 가짜 핸들을 진짜 핸들로 생성해야 하는 경우가 있다.
    - 구체적으로는 자식 프로세스를 하나 생성하고, 부모 프로세스에서 자식에게 자신의 핸들값을 전달해주고자 할때. (시나리오)
    - 위의 상황은 자신의 핸들테이블을 상속하여 해결 할 수 있다.
    - 하지만 자기 자신의 핸들값은 자기 핸들테이블에 존재하지 않기 때문에, 전달이 되지 않는다.
    - 해당 시나리오가 필요한 이유: 자식 프로세스에서 부모 프로세스를 waitForSingleObject() 시켜서 끝날때까지 감시를 해주기 위해
    - 해결방법: 
        - `DuplicateHandle(핸들1,256,핸들2,&val)`
        - 핸들1에 핸들테이블에 존재하는 256라는 핸들을 핸들2의 핸들테이블에 &val값으로 복사하고 싶다. (만약 val이 364이면 핸들테이블의 핸들값은 364: 커널 오브젝트가 된다.)

        ```c
        DuplicateHandle (
            GetCurrentProcess(),    //자신의 프로세스 핸들테이블에서
            GetCurrentProcess(),    // 자기자신을 가리키는 가짜 핸들
            GetCurrentProcess(),    // 자기 자신의 프로세스 핸들테이블에
            &hProcess,            // 실제 프로세스 주소를
            0,
            TRUE,
            DUPLICATE_SAME_ACCESS
        );
        ```

## 파이프 방식의 IPC

- IPC (3)
    - 메일 슬롯: 단방향, 브로드캐스팅/ 통신범위 제한 없음, (sender,receiver)
    - 이름없는(컴퓨터이름) 파이프:    단방향, 부모잣식 프로세스으로 통신범위 제한
        - 이름이 없기 때문에, 외부에서 접근이 불가하다.
    - 이름있는 파이프: 양방향, 통신범위 제한 없음
    - 파이프라이닝과는 다른 용어인듯하다.

- 이름없는 파이프
    - `CreatePipe(&hReadPipe,&hWritePipe,NULL,0);`
    - read 입구와 write입구 핸들을 받는다.
    - 단방향
    - 핸들을 통해서 읽고 쓰고를 한다.
    - 부모 자식프로세스간 통신
        - 핸들정보는 상속이 가능하다.
        - 파이프 입구와 출구 핸들을 상속받는다.
        - 이로 인하여 IPC 통신이 가능하다.
        - 부모가 write handle을 통해 데이터를 제공하면 자식이 이를 read Handle을 통하여 읽어들인다.


- 이름있는 파이프
    - 서버, 클라이언트
    - 양방향 통신
    - 진행 순서도
        1. 서버 내에서 `CreateNamedPipe()` 를 통하여 파이프를 생성한다. 단 해당 파이프는 아직 외부와 연결이 되지 않았다.
            - 최대 인스턴스갯수 인자: 파이프 하나당 연결 가능한 인스턴스  갯수
            - whil(1) { CreateNamedPipe() } 이유는 파이프는 동시에 연결을 할 순없지만, 동시에 여러개의 파이프를 가지고는 있다. 그러므로 파이프를 loop를 통핸 연결시켜준다.
        2. `ConnectNamedPipe()`를 통해 서버의 adapter를 추상화 시켜서 외부에 노출시킨다. (누구나 연결이 가능, 연결 대기 상태로 전환)
        3. client의 `CreateFile()`(메일 슬롯과 같은 함수)을 호출하여 파이프 오픈(연결)

    - cliet 코드
        - while(1) { CreateFile() }
            - 동시접속은 안되지만 여러개의 파이프를 가질 수 있다.
        - WaitNamedPipe()
        - SetNamedPipeHandleState(파이프핸들, 변경할 모드 정보, NULL,NULL)
        - FlushFileBuffers(hPipe);
            - 버퍼는 모아서 데이터를 보낸다.
            - 만약 일정 임계치까지 모이지 않은 상태여서 전송이 안된 데이터가 혹시라도 남아있다면 이를 비워준다.
        - DisconnectNamedPipe(hPipe);
            - 파이프 파괴
        - CloseHandle(hPipe);
            - Handle Usage Count -=1

