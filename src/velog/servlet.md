# Servlet

## Servlet이란?

- servlet은 동적인 웹 페이지를 만들 때 사용되는 서버쪽에서 실행되는 프로그램입니다.

### Servlet 등장배경

servlet이 나오기 전에는 고정된 내용을 가진 정적 웹 페이지만을 제공했고, 사용자들의 요청을 정적 웹 페이지에서 표현하기에는 한계가 있었기 때문에 동적인 데이터를
제공하는 웹 페이지의 필요성을 느끼고 CGI(Common Gate Interface)가 나오게 됐습니다. 하지만 CGI는 새로운 프로세스를 계속 만들어야 한다는 단점
때문에 자바 진영에서 동적 데이터를 제공하기 위한 CGI 기반으로 servlet이 등장하게 됐습니다.

#### CGI

- CGI web server에서 사용자의 요청을 받아 CGI 구현체에 요청을 전달하고 CGI 구현체가 요청에 따른 데이터를 동적으로 생성해 응답해 동적인 웹 페이지를
  구현할 수 있습니다.
- 장점
    - 언어, 플랫폼에 독립적
    - CGI 코드를 수행하는데 특정라이브러리가 필요하지 않아 매우 가벼움
    - 재사용할 수 있는 CGI 코드 라이브러리가 풍부
- 단점
    - **느림** 요청이 올 때마다 Db에 연결을 새로해야하기 때문에
    - 잡아먹는 리소스가 많음 Http 요청이 들어올 때 마다 새로운 프로세스를 만들어야 하기 때문
    - 페이지 로드 사이에 데이터가 메모리에 캐시될 수 없음

<br>
<img src="https://velog.velcdn.com/images/malslapq/post/cdcc8ec6-959f-4d4d-9ff2-414c48abc758/image.png" style="width: 90%" alt=""/>

### Servlet 특징

- 서버쪽에서 실행되고 기능을 수행 (MVC패턴에서 Controller로 사용)
- 프로세스가 아닌 쓰레드로 로직을 실행
- 자바로 만들어졌기 때문에 자바 언어의 특징을 가짐
- `javax.servlet.Servlet` 인터페이스를 상속 받은 구현체를 구현해야 함
- 싱글톤 패턴을 사용
- Java 코드 안에 HTML 태그가 삽입돼있음

### Servlet 사용

#### Servlet의 생명주기

1. 사용자의 요청을 받은 서버는 servlet class가 로딩되어 있는지 먼저 확인한다.
    1. 없을 경우 servlet class를 로딩하고 `init()` 메소드를 수행한다.(처음 초기화시 init(); 수행)
2. servlet class가 로딩돼있는 경우 `service()` 메소드를 수행한다. (실제 서비스 로직, doGet(), doPost() 메소드 등을 호출)
3. 서버가 종료되거나 servlet 종료 요청이 있을 경우 `destroy()`메소드를 수행하여 만들었던 servlet을 제거한다.

즉 `main()` 메소드가 필요 없고 독립적으로 실행되지 않으며 오직 servlet container에 의해 생성, 실행되고 소멸됩니다.

### Servlet Container

Servlet Container는 servlet의 생명주기를 관리하는 컨테이너입니다. 예를들어 Servlet Container는 공장의 관리인이고
servlet은 공장의 기계라고 생각해보자면, Servlet Container는 어떤 물품에 대한 요청이 들어오면 
해당 물품을 만들기 위해 servlet을 가동해 물품을 만들어 내서 요청한 사람에게 넘겨주는 것을 생각하면 됩니다.

- 멀티쓰레드 지원 및 관리 : 요청이 들어올 때 마다 쓰레드를 생성하여 처리하기에 해당 요청이 응답을 받으면 쓰레드도 자동으로 소멸한다.
- 선언적인 보안 관리 : 보안에 관련된 내용을 서블릿이나 자바 클래스에 구현하지 않아도 됌
- 통신 지원 : 클라이언트의 Request을 받고 Reponse를 할 수 있도록 웹 서버와 소켓을 만들어 통신
- 동작 
  1. Servlet Container는 사용자로부터 Http Request (요청)이 들어오면 쓰레드, servlet, HttpServletRequest, 
    HttpServletResponse를 
    생성합니다. (servlet이 이미 생성된 경우 servlet은 제외)
  2. 서비스 로직 실행 후 HttpServlet Response에 응답을 보내고 요청 시 만들었던 객체인 HttpServletRequest, 
    HttpServletResponse를 소멸시킵니다.


<br>
<img src="https://velog.velcdn.com/images/malslapq/post/6829490d-1640-4459-b245-54ff91412551/image.png" style="width: 80%" alt=""/>

```java
public class SampleServlet extends HttpServlet {


    @Override
    public void init() throws ServletException {
        // 처음 초기화시 실행되어야 할 로직
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) {
        // 실제 서비스 로직 
    }

    @Override
    public void destroy() {
        // 종료시 실행되어야 할 로직
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {   
        //  get 요청 시 수행될 로직
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) {
        //  post 요청 시 수행될 로직
    }
}
```

위 코드처럼 기본적인 `init(), service(), destroy()` 메소드 뿐만 아니라 HttpServlet을 상속받아 들어오는 Request에 대해 처리할
`doGet(), doPost()` 메소드를 반드시 구현해줘야 합니다.
