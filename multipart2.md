> **아니야. 저 상태 그대로면 “multipart 잘 처리했다”고 보기 힘들다.**  
> `MultipartResolver`를 등록해놔도, 지금 `LpkiWebRequestWrapper`가 multipart 요청을 중간에서 건드리고 있어서 구조상 위험해.

왜 그런지 차근차근 짚어볼게.

---

## 1. 지금 LpkiWebRequestWrapper 가 multipart 에서 하는 일

현재 생성자:

`public LpkiWebRequestWrapper(HttpServletRequest request) throws IOException {     super(request);     base64Key = (String) request.getSession().getAttribute("l_pki_base64_key");     base64Iv = (String) request.getSession().getAttribute("l_pki_base64_iv");      String characterEncoding = request.getCharacterEncoding();     if (StringUtils.isBlank(characterEncoding)) {         characterEncoding = StandardCharsets.UTF_8.name();     }     encoding = Charset.forName(characterEncoding);      log.info("encoding::{}", encoding);     log.info("base64Key::{}", base64Key);     log.info("base64IV::{}", base64Iv);      try {         InputStream inputStream = request.getInputStream();         rawData = IOUtils.toByteArray(inputStream);   // 🔥 여기서 무조건 body 전체 읽음          String contentType = request.getContentType();         if (contentType == null || !contentType.toLowerCase().contains("multipart/form-data")) {             rawData = AESUtil.decrypt(rawData, base64Key, base64Iv);  // multipart면 decrypt만 스킵         }          decodedParameters = decodeParameters(request); // multipart일 때도 호출됨      } catch (IOException e) {         throw e;     } }`

`decodeParameters`도 multipart 에서 이렇게 동작해:

`private Map<String, String[]> decodeParameters(HttpServletRequest request) {      String reqType       = request.getMethod().toLowerCase();     String contentType   = request.getContentType();     String urlSearchParams = request.getQueryString();      Map<String, String[]> decodedParams = new HashMap<>();      try {         String seachParameters = null;         log.info("##1");         if (urlSearchParams != null && ("get".equals(reqType) || "delete".equals(reqType))) {             log.info("##2");             seachParameters = AESUtil.decrypt(urlSearchParams, base64Key, base64Iv);          } else if (contentType != null && contentType.toLowerCase().contains("multipart/form-data")) {             log.info("multipart/form-data 처리");             // ❌ 하지만 decodedParams 에 아무것도 안 넣고 넘어감          } else if (contentType != null && !contentType.toLowerCase().contains("application/json")) {             log.info("##3");             seachParameters = new String(rawData);         }          log.info("##4");         if (seachParameters != null && !seachParameters.isEmpty()) {             log.info("##5");             processEncodedString(seachParameters, decodedParams);         }         log.info("##6");      } catch (Exception e) {         log.info("##7");         log.error("Error decoding parameters", e);         return new HashMap<>();     }     log.info("##8");     return decodedParams; }`

그리고 파라미터 관련 메서드 전부 `decodedParameters`만 보고 있음:

`@Override public Map<String, String[]> getParameterMap() {     log.info("decodedParameters::" + decodedParameters);     return Collections.unmodifiableMap(decodedParameters); }  @Override public Enumeration<String> getParameterNames() {     log.info("decodedParameters.keySet()::{}", decodedParameters.keySet());     return Collections.enumeration(decodedParameters.keySet()); }  @Override public String[] getParameterValues(String name) {     log.info("name::{}", name);     return decodedParameters.get(name);      // ❌ super.getParameterValues 안 씀 }`

### 결과적으로 multipart 에서는

1. **InputStream 을 한 번 싹 다 읽어버리고(rawData)**
    
    - MultipartResolver가 나중에 읽으려고 하면 이미 EOF 일 수도 있음 (환경에 따라 문제 발생 가능)
        
2. `decodeParameters()` 에서 multipart 는
    
    `log.info("multipart/form-data 처리");`
    
    만 찍고 **decodedParams 에 아무 값도 안 넣음**
    
3. 그래서 `decodedParameters` 는 빈 Map `{}` 인 상태
    
4. 그 상태로
    
    - `getParameterMap()`, `getParameterNames()`, `getParameterValues()` 가 전부 `decodedParameters` 를 통해서만 리턴
        
    - `super.getParameter*()` 로 fallback 하지 않음
        

즉, **multipart 요청일 때 파라미터를 전부 직접 막으면서**,  
“잘 처리했다”고 말하기는 어려운 구조야.

---

## 2. MultipartResolver 등록했다고 자동으로 안전한 게 아니다

`LpkiFilterConfig`:

`@Bean public MultipartResolver multipartResolver() {     CommonsMultipartResolver multipartResolver = new LpkiMultipartResolver();     multipartResolver.setMaxUploadSize(209715200);     return multipartResolver; }`

이건 **DispatcherServlet 레벨에서만** 동작하는 거야:

- DispatcherServlet 이 들어온 `HttpServletRequest` 를 보고
    
    - Content-Type 이 `multipart/form-data; boundary=...` 이면
        
    - `MultipartResolver` 를 호출해서 request body 를 파싱
        
    - 그 결과로 `MultipartHttpServletRequest` 로 감싼 뒤 컨트롤러로 넘김
        

그런데 그 전에 **filter + wrapper** 가:

- `request.getInputStream()` 다 읽고
    
- rawData 를 이상하게 변경하거나
    
- `getParameter*()` 를 override 해서 빈 값만 리턴하고 있으면
    

**MultipartResolver 의 정상 동작이 깨질 수밖에 없어.**

그래서:

> ✅ **MultipartResolver 등록 +  
> ❌ 필터/래퍼가 multipart 요청을 건드리는 구조**  
> → 이건 안전하다고 볼 수 없음.

---

## 3. “multipart 잘 처리한 상태”가 되려면 어떻게 해야 하냐

### 선택지 A – 필터에서 아예 multipart 제외 (가장 깔끔)

`@Override public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)         throws IOException, ServletException {      HttpServletRequest httpReq = (HttpServletRequest) request;     String uri         = httpReq.getRequestURI();     String contentType = httpReq.getContentType();      // 1) multipart면 암복호화 / 래핑 전부 제외     if (contentType != null &&         contentType.toLowerCase().contains("multipart/form-data")) {          log.info(">>> LpkiWebProcessingFilter 적용제외 (multipart/form-data) : uri={}, contentType={}",                  uri, contentType);         chain.doFilter(request, response);         return;     }      // 2) 나머지에만 LpkiWebRequestWrapper 적용     ... }`

이렇게 하면:

- `/v1/fileupload` 같은 multipart는
    
    - `LpkiWebRequestWrapper` 도 안 태우고
        
    - `AESUtil.decrypt(rawData, ...)` 도 안 돌리고
        
    - `decodedParameters` 도 안 건드림
        
- 그냥 **스프링 기본 multipart + LpkiMultipartResolver 조합으로만 동작**
    

이게 사실 제일 안정적이야.

---

### 선택지 B – 래퍼에서 multipart 를 완전히 패스시키기

필터는 그대로 두고, `LpkiWebRequestWrapper` 안에서만 이렇게 처리할 수도 있어:

`public LpkiWebRequestWrapper(HttpServletRequest request) throws IOException {     super(request);      base64Key = (String) request.getSession().getAttribute("l_pki_base64_key");     base64Iv  = (String) request.getSession().getAttribute("l_pki_base64_iv");      String characterEncoding = request.getCharacterEncoding();     if (StringUtils.isBlank(characterEncoding)) {         characterEncoding = StandardCharsets.UTF_8.name();     }     encoding = Charset.forName(characterEncoding);      String contentType = request.getContentType();     log.info("encoding::{}", encoding);     log.info("base64Key::{}", base64Key);     log.info("base64IV::{}", base64Iv);     log.info("request.getContentType() = {}", contentType);      // 🔥 1) multipart면: body 안 건드리고, 파라미터도 super 그대로 쓰게 설정     if (contentType != null &&         contentType.toLowerCase().contains("multipart/form-data")) {          log.info("LpkiWebRequestWrapper: multipart/form-data -> 암복호화/파라미터 래핑 스킵");          this.rawData           = new byte[0];                // 의미 없는 값         this.decodedParameters = null;                       // null 표시해두고          return;  // 생성자 여기서 끝     }      // 🔥 2) multipart가 아닌 경우만 body 읽고 복호화 + decodeParameters     try {         InputStream inputStream = request.getInputStream();         rawData = IOUtils.toByteArray(inputStream);          log.info("rawData before decrypt = [{}]", new String(rawData, encoding));          // 여기서는 multipart 아님이 확실         rawData = AESUtil.decrypt(rawData, base64Key, base64Iv);          log.info("rawData after decrypt = [{}]", new String(rawData, encoding));          decodedParameters = decodeParameters(request);     } catch (IOException e) {         throw e;     } }`

그리고 `getParameter*` 쪽을 이렇게 바꿈:

`@Override public Map<String, String[]> getParameterMap() {     if (decodedParameters == null) {         // 🔥 multipart 등: wrapper에서 파싱 안 했으니, 원본 서블릿 값 사용         return super.getParameterMap();     }     return Collections.unmodifiableMap(decodedParameters); }  @Override public Enumeration<String> getParameterNames() {     if (decodedParameters == null) {         return super.getParameterNames();     }     return Collections.enumeration(decodedParameters.keySet()); }  @Override public String[] getParameterValues(String name) {     if (decodedParameters == null) {         return super.getParameterValues(name);     }     return decodedParameters.get(name); }  @Override public String getParameter(String name) {     String[] values = getParameterValues(name);     return (values != null && values.length > 0) ? values[0] : null; }`

이러면:

- multipart 요청은:
    
    - body 복호화/건드리는 작업 전부 스킵
        
    - 파라미터는 `super.getParameter*()` 를 그대로 사용 → MultipartResolver 가 채워준 값 사용
        
- 나머지 요청만 `decodedParameters` 기반으로 동작
    

---

## 4. 질문에 대한 딱 한 줄 답변

> **Q.** “그럼 multipart 잘 처리 한거 아니냐?”  
> **A.** 지금 코드 그대로라면, `MultipartResolver` 만 믿기에는 wrapper 가 multipart 요청을 중간에서 건드리고 있어서 “완전히 잘 처리했다”고 보긴 어렵다. 최소한
> 
> - **필터에서 multipart 를 아예 제외**하거나,
>     
> - **래퍼에서 multipart 면 `rawData` / `decodedParameters`를 건드리지 않고 `super.getParameter*()` 로 fallback** 하는 수정이 한 번 더 들어가야 안전하다.
>