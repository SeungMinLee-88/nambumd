좋은 포인트 찍었어. 결론부터 말하면:

> ✅ **멀티파트용으로 `LpkiMultipartResolver` / `LpkiMultipartHttpServletRequest` 따로 만든 방향은 맞는 방향이고, “멀티파트는 이쪽에서 처리한다”는 설계 자체는 잘한 거야.**  
> ❌ 근데 **그래도 지금 상태를 “잘 처리된 상태”라고 보기엔 부족한 부분이 아직 있어.**  
> (특히 `LpkiWebRequestWrapper`랑 어떻게 같이 돌리느냐가 핵심)

왜 그런지, 네가 올린 코드 기준으로 정리해볼게.

---

## 1. 지금 구조 요약

### 1) Multipart 설정

`@Bean public MultipartResolver multipartResolver() {     CommonsMultipartResolver multipartResolver = new LpkiMultipartResolver();     multipartResolver.setMaxUploadSize(209715200);     return multipartResolver; }`

→ **스프링의 멀티파트 처리는 전부 `LpkiMultipartResolver`를 통해 간다**는 뜻.  
즉, `DispatcherServlet` 입장에서는:

- 요청이 `multipart/form-data` 이면  
    → `LpkiMultipartResolver.resolveMultipart(request)` 호출  
    → 여기서 **(원래라면)** `MultipartHttpServletRequest` 로 감싸서 컨트롤러에 넘김  
    → 네 경우엔 `LpkiMultipartHttpServletRequest` 를 리턴하겠지.
    

그리고 `LpkiMultipartFile` 은 byte[] 기반으로 파일 하나를 감싸는 구현이고,  
커스텀 멀티파트 처리하려는 의도는 분명함. 여기까진 👍

### 2) Filter / Wrapper 쪽

`@Bean public FilterRegistrationBean<LpkiWebProcessingFilter> LpkiWebProcessingFilter() {     FilterRegistrationBean<LpkiWebProcessingFilter> registrationBean = new FilterRegistrationBean<>();     registrationBean.setFilter(new LpkiWebProcessingFilter());     registrationBean.addUrlPatterns("/v1/*");     return registrationBean; }`

→ `/v1/*` 으로 들어오는 **모든 요청**은  
`LpkiWebProcessingFilter` → `LpkiWebRequestWrapper` 를 반드시 탄다는 얘기.

그리고 `LpkiWebRequestWrapper` 생성자는 지금 이렇게 돼 있지:

`public LpkiWebRequestWrapper(HttpServletRequest request) throws IOException {     super(request);     ...     InputStream inputStream = request.getInputStream();     rawData = IOUtils.toByteArray(inputStream);   // 🔥 multipart 포함, 무조건 한 번 다 읽음      String contentType = request.getContentType();     if (contentType == null || !contentType.toLowerCase().contains("multipart/form-data")) {         rawData = AESUtil.decrypt(rawData, base64Key, base64Iv);     }      decodedParameters = decodeParameters(request);  // multipart 도 호출됨 (분기만 다름) }`

그리고 `getParameter*()` 들은 전부 `decodedParameters`만 봄:

`@Override public Map<String, String[]> getParameterMap() {     return Collections.unmodifiableMap(decodedParameters); }  @Override public String[] getParameterValues(String name) {     return decodedParameters.get(name); }`

---

## 2. 그래서 “멀티파트 잘 처리한 상태냐?”를 따져보면

### 👍 잘한 부분

- 멀티파트 전용으로
    
    - `LpkiMultipartResolver`
        
    - `LpkiMultipartHttpServletRequest`
        
    - `LpkiMultipartFile`  
        을 만든 건 **방향성 완전 맞음**이야.
        
- 즉, “파일 업로드/멀티파트는 일반 바디 암호화/복호화와 분리해서 처리하겠다”  
    라는 설계 자체는 맞고, 그걸 **Resolver/Request 레벨에서 처리**한 것도 좋은 선택.
    

### 하지만 👇

`MultipartResolver`가 있다고 해서 **filter/wrapper 가 멀티파트를 막 건드려도 자동으로 보호되는 건 아니야.**

지금 구조에선:

1. `/v1/fileupload` 같은 멀티파트 요청도  
    → `LpkiWebProcessingFilter` 를 먼저 탐 (`/v1/*` 매핑이라서)
    
2. 필터 안에서 `LpkiWebRequestWrapper` 를 new 해서 감쌈
    
3. 그 안에서 `request.getInputStream()` 으로 **스트림을 한 번 싹 다 읽음**
    
4. 비록 `contentType.contains("multipart/form-data")` 이면 `AESUtil.decrypt()` 는 안 돌지만,  
    **스트림은 이미 네가 한 번 소비한 상태**가 됨
    
5. 그 다음 DispatcherServlet 이 `LpkiMultipartResolver` 를 호출할 때는  
    이미 `LpkiWebRequestWrapper` 로 감싸진 request 가 넘어감
    

→ 이 상황에서 `LpkiMultipartResolver` 가 기대하는 “원본 서블릿 request” 상태하고는 달라.  
케이스에 따라:

- 환경/서블릿 구현에 따라:
    
    - stream 한 번 읽고 나면 다시 못 읽을 수 있음 → 멀티파트 파싱 실패 가능
        
- 설계에 따라:
    
    - `LpkiMultipartHttpServletRequest` 가 내부에서 `originalRequest.getParameter(...)` 를 쓸 때,  
        그 `originalRequest` 가 이미 wrapper + 빈 `decodedParameters` 상태일 수도 있음
        

그래서:

> **멀티파트 전용 Resolver 를 직접 만들었다 = 설계 방향은 OK**  
> 하지만  
> **지금 LpkiWebRequestWrapper 가 멀티파트까지 같이 훑고 있다는 점 때문에 “이미 잘 처리됐다”고 말하긴 어렵다.**

---

## 3. 정말 “멀티파트 잘 처리된 상태”로 만들려면

이제 해야 할 건 **역할 분리**야:

- **멀티파트(파일 업로드)**  
    → `LpkiMultipartResolver` + `LpkiMultipartHttpServletRequest` 만 담당  
    → 필터/래퍼는 **손 안댐**
    
- **그 외 JSON/form/x-www-form-urlencoded**  
    → `LpkiWebRequestWrapper` 에서 암복호화 + `decodedParameters` 처리
    

### 방법 1 – 필터에서 아예 멀티파트 제외 (추천)

`LpkiWebProcessingFilter` 에서 제일 먼저:

`String contentType = request.getContentType(); String uri = httpReq.getRequestURI();  if (contentType != null &&     contentType.toLowerCase().contains("multipart/form-data")) {      log.info(">>> LpkiWebProcessingFilter 적용제외 (multipart/form-data) : uri={}, contentType={}",              uri, contentType);     chain.doFilter(request, response);     return; }`

그 다음에만 `"req"`, `"res"`, 기본 분기 타서 `LpkiWebRequestWrapper` 를 적용.  
이러면:

- 멀티파트는 **무조건**:
    
    - 원본 HttpServletRequest → DispatcherServlet → `LpkiMultipartResolver`  
        라인만 타고,
        
- 암호화/복호화 래퍼는 멀티파트에 손 안 댐
    

### 방법 2 – 래퍼에서 멀티파트 완전 패스

필터는 그대로 두고, `LpkiWebRequestWrapper` 생성자에서:

`String contentType = request.getContentType(); if (contentType != null &&     contentType.toLowerCase().contains("multipart/form-data")) {      log.info("LpkiWebRequestWrapper: multipart/form-data -> 암복호화/디코드 스킵");      this.rawData           = new byte[0];     this.decodedParameters = null;   // “내가 파싱 안 했다”는 표시      return; // 아래 decrypt/ decodeParameters 안 타게 여기서 끝 }`

그리고 `getParameter*()` 들에서:

`@Override public Map<String, String[]> getParameterMap() {     if (decodedParameters == null) {         // multipart 같은 경우: 스프링/MultipartResolver가 채운 값 그대로 쓰기         return super.getParameterMap();     }     return Collections.unmodifiableMap(decodedParameters); }  @Override public String[] getParameterValues(String name) {     if (decodedParameters == null) {         return super.getParameterValues(name);     }     return decodedParameters.get(name); }`

이렇게 해두면:

- multipart 요청  
    → `LpkiWebRequestWrapper` 는 실질적으로 “투명 패스스루”  
    → `MultipartResolver` + `LpkiMultipartHttpServletRequest` 가 본인 로직대로 처리
    
- 나머지 요청  
    → 기존처럼 `rawData` 복호화 + `decodedParameters` 사용
    

---

## 4. 정리

> ❓ _“첨부된 파일로 보아 multipartResolver를 직접 만들어서 처리 한거 같은데 저러면 잘 처리 한거 아니냐?”_

- **구조/의도 자체는 잘 잡아놓은 거 맞아.**
    
    - 멀티파트 전용 Resolver/Request/File 구현 따로 둔 거 → Good.
        
- 하지만 **현재 `LpkiWebRequestWrapper`가 멀티파트 요청의 InputStream 과 파라미터까지도 건드리고 있기 때문에**,  
    `LpkiMultipartResolver` 하나만 보고 “이제 멀티파트는 잘 처리된다” 라고 말하긴 힘들어.
    
- 진짜 안정적으로 만들려면:
    
    - **필터 단계에서 멀티파트를 암복호화 대상에서 완전히 제외**하거나,
        
    - 아니면 __래퍼 내부에서 multipart 면 decrypt/decodedParameters 를 아예 건드리지 않고 super._ 로 fallback_* 하도록 한 번 더 손봐줘야 한다.
        

이 두 개만 확실히 정리하면,  
지금 만들어놓은 `LpkiMultipartResolver` 세트랑 아주 깔끔하게 맞물리게 될 거야.