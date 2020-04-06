# [CH21] Dynamic Linking Library

## Static Library vs DLL
DLL은 가상메모리에서 필요할때 DLL파일을 VM에 매핑을 시키는 형태이고, Static Library는 VM에 .exe파일을 올릴때 Static Lib(정적 라이브러리) 영역이 VM에 올라간다.  

참고로 가상 메모리중 몇개의 page만 메인 메모리에 올라간다.


## DLL의 특성
1. 여러 .exe파일에서, .dll 파일끼리 **서로서로 linking** 시킬 수 있다.
2. 프로그램 실행중에(**runtime**, not compile time) 필요하다면 DLL파일을 요청하여 해당 프로세스의 Virtual Memory공간에서 해당 Thread Heap 영역(한 프로세스 내에서 스레드끼리는 Heap영역을 공유하지만, 엄밀하게는 같은 heap 영역에 각자의 쓰레드힙 영역을 구획을 두고 할당시켜준다)에 .dll파일을 매핑 시켜준다. 이후 page algorithm에 따라서 VM에서 RAM으로 PAGE 단위로 올라갈 수도 올라가지 않을 수도 있다.
3. **Context Switching**때 공유될 수도 있다.
	-  시나리오1 (static lib): a.exe와 b.exe는 같은 static lib를 사용하고 있다. 현재 RAM에는 a.exe 프로세스가 돌아가고 있고, 일부 페이지에 a.exe가 참조하는 static lib가 올라가 있다. 이때 context switching 이 일어나는데, 만약 b.exe에서 RAM에 올라간 static lib와같은 page를 필요로 하더라도 정적 라이브러리 영역에서는 context switching시 모두 다 변경 시킨다.
	- 시나리오 2( DLL ): 시나리오1과 같은 상황에서 라이브러리 파일만 다르다면, VM 메모리의 dll영역은 HDD의 별도의 DLL파일에 존재하는 파일 내용을 mapping 시켜서 사용하는 것이다. 그렇기 때문에,  같은 내용을 참조하고 있다면 context switching 시점에 cpu가 VM에서 해당 dll page를 내릴 이유가 전혀 없다.  (시나리오1에서는 cpu가 알지 못한다. a.exe의 dll과 b.exe의 dll이 같은 page를 참조하고 ram에 올라가 있는지를 )
4. OS가 한번 DLL을 load하��� RAM / HDD 사이에서 페이지 단위로 왔다갔다 한다.
	- os가 물리메모리로 .dll을 한번 로드하고 나면, 해당 데이터는 여러 프로세스들의 가상 메모리에 페이지 단위로 매핑된다.
	- 해당 매핑 정보가 RAM에 올라갔다, 다시 VM의 HDD로 내려왔다. 하면서 cycle을 돌긴하지만 os로부터 메모리로 로딩되는 것은 1번 뿐이다. 


## c.f) 물리주소, 물리 메모리
- 물리주소: RAM의 실제 주소
- 물리 메모리: RAM + HDD 까지 확장(VM과 연관되는 물리 메모리)
- 