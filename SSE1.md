## 2. 더 근본적인 문제: ContentCachingResponseWrapper + SSE는 원래 안 맞는다

지금 필터는 응답에서 이렇게 하고 있지?

`ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper((HttpServletResponse) response); ... chain.doFilter(requestWrapper, responseWrapper);  // 그 뒤에 byte[] responseArray = responseWrapper.getContentAsByteArray(); ... 암호화해서 다시 씀 ... responseWrapper.copyBodyToResponse();`

이 방식은 전형적인 “**요청 한 번에 딱 한 번 나가는 완성된 응답**”에 맞는 패턴이야.  
근데 SSE / Flux<ServerSentEvent> 는:

- 응답이 **스트리밍으로 여러 번 나감** (chunked 전송),
    
- 심지어 **`chain.doFilter(...)` 리턴된 후에도** 계속 쓰여질 수 있음.
    

`ContentCachingResponseWrapper` 는 내부 버퍼에 **한 번에 다 모았다가** 너한테 byte[] 로 주는 구조라서,  
**스트리밍과 근본적으로 상성이 안 맞음**.

그래서 지금 로그도 이렇지:

`LpkiWebProcessingFilter   : 응답 처리:: response responseArray.length : 0 ... 그 이후에 reactor 쓰레드에서 LLM 결과 로그가 찍힘`

- 필터 입장에서는 `responseArray.length == 0` 이라 “아, 응답이 없네?” 하고 끝내버리고,
    
- 진짜 SSE 데이터는 뒤에서 WebFlux/RestService가 비동기로 계속 씀 → onprogress만 보면 잘 오고 있는 구조.
    

즉,

> “text/event-stream 필터링이 안 된다”  
> = 필터가 응답 내용을 **제대로 볼 수 있는 구조가 아니라서** + Content-Type 도 정확히 안 맞아서

라고 보는 게 맞음.

---

## 3. 그래서 서버 쪽에 필요한 수정

두 가지 레벨에서 손을 봐야 해.

### 3-1. **컨트롤러/핸들러에서** Content-Type 을 명확히 SSE로

SSE 엔드포인트(아마 `/v1/chats/messages` GET 쪽)에서 반환 타입이 대략 이런 거일 거야:

`@GetMapping(value = "/v1/chats/messages", produces = MediaType.TEXT_EVENT_STREAM_VALUE) public Flux<ServerSentEvent<String>> streamMessages(...) {     return chatService.stream(...); // 여기서 위에 올린 fluxUrl() 체인 사용 }`

또는 functional router라면:

`RouterFunction<ServerResponse> route() {     return route()         .GET("/v1/chats/messages", accept(MediaType.TEXT_EVENT_STREAM), handler::streamMessages)         .build(); }`

**이렇게 `produces = MediaType.TEXT_EVENT_STREAM_VALUE` 나 router에서 text/event-stream 을 명시해야**:

- 서블릿 응답 헤더에 `Content-Type: text/event-stream` 이 정확히 찍히고,
    
- 필터의 `respContentType.contains("text/event-stream")` 조건이 **실제로 참이 됨**.
    

이게 안 되어 있으면, 필터는 평생 SSE인지 아닌지 모른다.

---

### 3-2. SSE 엔드포인트는 아예 응답 래핑/암호화에서 제외하는 게 더 안전

솔직히 말하면, `ContentCachingResponseWrapper` 를 쓰는 구조에서  
**SSE를 깨끗하게 지원하는 건 거의 불가능에 가깝다**고 보는 게 맞다.

그래서 현실적인 전략은:

- **요청 복호화**는 필요하면 해도 됨 (POST `/v1/chats/messages` 등),
    
- 하지만 **응답 암호화 / JSON 포장 부분은 SSE 엔드포인트에 대해서는 아예 안 건드리는 것**.
    

예를 들면 필터 상단에 이렇게 박는 거:

`String uri = req.getRequestURI(); String method = req.getMethod();  // 1) SSE 응답 엔드포인트면 요청/응답 둘 다 필터 완전 패스 if (uri.startsWith("/v1/chats/messages")) {     log.info(">>> LpkiWebProcessingFilter SSE 엔드포인트 제외 : uri={}", uri);     chain.doFilter(request, response);     return; }`

또는 좀 더 정교하게:

- POST /v1/chats/messages → 요청만 복호화, 응답은 평범 (JSON) 이면 필터 적용
    
- GET /v1/chats/messages → SSE 스트리밍이므로 필터 완전 제외
    

이런 식으로도 가능하고, 예:

`if (uri.startsWith("/v1/chats/messages") &&     "GET".equalsIgnoreCase(method)) {     log.info(">>> LpkiWebProcessingFilter SSE(GET /v1/chats/messages) 완전 제외");     chain.doFilter(request, response);     return; }`

이게 **가장 확실하고 고통이 적은 방법**이야.

---

## 4. 정리하면

> 질문: “수정된 응답필터 부분 text/event-stream 필터링이 안되고 서버가 리턴하는 부분이 아래와 같으면 서버쪽 수정이 필요하냐?”

**응, 필요해.** 그리고 방향은 대략 이렇다:

1. **SSE 컨트롤러/라우터에서**  
    `produces = MediaType.TEXT_EVENT_STREAM_VALUE` 또는  
    `response.setContentType("text/event-stream")` 를 명확하게 세팅해서  
    응답 헤더에 `Content-Type: text/event-stream` 이 나오게 고쳐야 하고,
    
2. 동시에, 지금 구조( ContentCachingResponseWrapper + 응답 암호화 )에서는  
    **SSE 엔드포인트는 아예 필터 응답 래핑/암호화 대상에서 빼는 게 맞다.**  
    (URI + method 기준으로 early return)
    
3. 프론트 쪽은:
    
    - SSE용 AJAX에서 `Accept: text/event-stream` 달아주면 더 좋고,
        
    - 지금처럼 `xhrFields.onprogress` 로 스트림을 읽는 패턴 자체는 유지 가능.
        

네가 말한 것처럼 지금 시스템은 “외부 LLM 서버 SSE → 우리 서버 → 브라우저 onprogress” 구조라서,  
중간에 있는 **Lpki 필터가 응답을 만지면 스트림이 무조건 깨지게 되어 있음**.  
그래서 SSE 라인은 “요청은 암호화(필요하면), 응답은 그대로 통과” 이 원칙으로 가는 게 맞아.