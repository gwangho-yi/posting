FTP는 통신할 때, 두 개의 연결을 사용한다.

HTTP 처럼 요청, 응답 패턴이지만 요청과 데이터를 한꺼번에 보내는 HTTP와는 다르게 명령어 연결과 데이터 연결이라는 두 가지 방식을 사용한다.

### 1. 명령어 연결 ( Control Connection )

- 명령어 포트를 사용한다. 주로 21번 사용 → 다른 포트로 변경 가능하다.
- 클라이언트가 서버에 명령 내릴 때 사용
- 클라이언트의 명령과 서버의 응답만 주고 받는 통로
- 아래는 주로 사용 되는 명령
    - 서버 로그인
    - 디렉토리 리스트 요청
    - 파일 업로드/다운로드 명령
    - 접속 종료 등
- HTTP 는 한번의 전송으로 TCP 연결은 종료되지만, FTP는 명령어 연결 시 사용했던 TCP 연결이 계속 유지된다.
- 즉, FTP 연결은 Keep-Alive 하다.

### 2. 데이터 연결

- 실제 파일을 주고 받는 통로
- 서버가 클라이언트로 파일 전송을 위해 연결하는 포트
- 아래 두 가지 모드 일 때, 데이터 연결 방향이 달라진다.
    - Active 모드
        
        서버 → 클라이언트
        
    - Passive 모드
        
        클라이언트 → 서버
        

## Active 모드, Passive 모드

### 1. Active 모드

- FTP의 전통적 방식
- 명령어 연결 : 클라이언트 → 서버
- 데이터 연결 : 서버 → 클라이언트
- 연결 순서는 아래와 같다.
    1. 클라이언트가 데이터 전송 요청
    2. 클라이언트가 자신의 랜덤 포트 번호(1024번 이상) 를 서버 알려준다.
    3. 서버가 클라이언트의 랜덤 포트에 접속하여 연결된다.
- 문제점
    - 클라이언트의 방화벽, NAT 뒤에 있을 때는 서버 → 클라이언트 연결이 차단될 수 있다.
    - 요즘은 잘 쓰이지 않는다.

### 2. Passive 모드

- 방화벽/NAT 환경에서 주로 사용
- 명령어 연결 : 클라이언트 → 서버
- 데이터 연결 : 클라이언트 → 서버
- 연결 순서는 아래와 같다.
    1. 클라이언트가 데이터 전송 요청
    2. 클라이언트가 Passive 모드 요청 → 서버가 데이터 포트를 클라이언트에 전송
    3. 클라이언트가 서버의 데이터 포트에 접속하여 연결된다.
- 장점
    - Active 모드보다 방화벽 환경에서 연결이 자유롭고 안정적이다.



#FTP #네트워크