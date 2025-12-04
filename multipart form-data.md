좋은 질문이다.  
**“못 하는 게 아니라, 지금 네 구조(필터 + 래퍼 + AES)로는 하면 100% 깨지게 돼 있어서”** 안 하는 거야.

차근히 정리해볼게.

---

## 1. multipart/form-data 가 뭔지부터

`multipart/form-data` 는 이런 식으로 생긴 **복잡한 포맷**이야:

`POST /upload HTTP/1.1 Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryABC123  ------WebKitFormBoundaryABC123 Content-Disposition: form-data; name="chunks" 1 ------WebKitFormBoundaryABC123 Content-Disposition: form-data; name="uuid" 7952adde-ba70-45eb-9627-3b12bab5f727 ------WebKitFormBoundaryABC123 Content-Disposition: form-data; name="file"; filename="test.xlsx" Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet  (binary file bytes....) ------WebKitFormBoundaryABC123--`

여기서 중요한 건:

- **boundary 문자열** (`----WebKitFormBoundaryABC123`)
    
- 각 파트의 헤더 (`Content-Disposition`, `Content-Type`)
    
- 파일의 바이너리 바이트들
    

Spring / Servlet 컨테이너(Tomcat)는 이걸 보고:

1. boundary 기준으로 잘라서
    
2. 각 파트 헤더 읽고
    
3. `MultipartFile`, `request.getParameter("uuid")` 같은 걸 만들어 줌
    

---

## 2. 네가 지금 하는 AES 암복호화는 “이런 전제”를 깔고 있음

현재 AES 처리 흐름을 보면:

- **POST/PUT/DELETE 바디**는
    
    - 평문이 `키=값&키=값` 형태 (x-www-form-urlencoded) 이거나
        
    - JSON 텍스트 ( `{ ... }` ) 라는 걸 가정하고
        
- **통째로 문자열 → AES 암호화 → Base64 문자열**로 만듦
    
- 서버에서:
    
    - `rawData` 를 바이트 배열로 읽고
        
    - `AESUtil.decrypt(rawData, key, iv)` 해서 다시 **“그냥 텍스트”**로 돌려놓은 다음
        
    - 그걸 다시 **쿼리스트링처럼 파싱**하거나, JSON으로 파싱하려고 함
        

즉, 이 구조는:

> **“바디 전체가 하나의 문자열이다”**  
> 라는 가정 위에서 돌아가는 암·복호화 로직이야.

---

## 3. 그런데 multipart/form-data 는 “하나의 문자열”이 아님

`multipart/form-data` 는:

- 중간에 헤더 섞여 있고
    
- 중간에 boundary 명시적으로 들어가 있고
    
- 파일은 **바이너리** 그대로 들어있고,
    
- 전체 구조가 깨지면 **스프링의 MultipartResolver가 이걸 더 이상 multipart로 인식 못 함**
    

이걸 네 방식대로 **바디 전체를 AES 암호화했다가** 다시 복호화하면:

- 암호화 후 바디는 그냥 의미 없는 Base64 문자열
    
- 복호화 후에도 “똑같은 multipart 포맷”으로 되돌아와야 하는데,  
    그걸 제대로 하려면 **암복호화 과정에서 단 1바이트도 손실/추가/공백 변환이 일어나면 안 됨**
    
- 게다가 네쪽 래퍼는 **복호화 후 그 내용을 다시 가공(파싱/재조합)** 하기도 함  
    (`decodeParameters`에서 `rawData`를 `new String()`으로 바꾸고, 다시 쪼개고 등등)
    

조금이라도 잘못 만지면:

- boundary 문자열 깨짐
    
- `\r\n` ↔ `\n` 변환, charset 문제, 공백/플러스 치환 같은 거 생김
    
- 그러면 Spring 이:
    
    > "이거 multipart/form-data 맞는 형식 아닌데?"  
    > → `MultipartFile` 비어 있음  
    > → `uuid` 같은 텍스트 필드도 못 읽음
    

지금 실제로 네 로그에서 보는 것처럼:

- `rawData before decrypt = []`
    
- `decodedParameters = {}`
    
- `getParameter("uuid")` → null
    

이런 현상이 딱 그 결과.

---

## 4. 요약하면, 왜 “지금 구조에선” multipart 에 암복호화 적용하면 안 되냐

1. **multipart 는 “프로토콜/포맷” 자체라서**  
    → boundary / 헤더 / 바이너리가 그대로 살아 있어야 함  
    → 중간에 AES 한 번 태우고, Base64 인코딩/디코딩, String 변환, URLDecoder 등 거치면  
    그 구조가 깨질 위험이 너무 큼.
    
2. **지금 네 암복호화 레이어는 “문자열 바디”만 상정하고 설계됨**
    
    - `키=값&키=값` / JSON 텍스트에는 잘 맞지만
        
    - multipart 같은 “컨테이너 포맷 + 바이너리”에는 안 맞음
        
3. **Spring 의 multipart 처리 시점과도 충돌**
    
    - 보통 MultipartResolver가 **필터/인터셉터 전에** request inputStream을 파싱하는데
        
    - 네가 filter/wrapper에서 inputStream을 먼저 다 읽고 재가공해 버리면  
        multipart 파서가 더 이상 원본 스트림을 볼 수 없음 → 꼬임
        

그래서 업계에서도 보통:

- **multipart/form-data 전체를 암호화**하는 식으로는 잘 안 하고
    
- 대신:
    
    - 파일 내용 자체를 따로 암호화 (예: 파일을 미리 AES 암호화해서 업로드)
        
    - 또는
        
    - 메타데이터(JSON)만 암호화하고, 파일은 그대로 multipart로 보냄
        

이런 식으로 설계함.

---

## 5. 그러면 “원칙적으로” multipart 암복호화는 완전 불가능이냐?

**원칙적으로는 가능은 해.**  
단, 조건이 빡셈:

1. 클라이언트와 서버가 **multipart 프로토콜 바깥에서** 암호화 해야 함
    
    - 예:
        
        - 파일을 미리 AES로 암호화한 파일로 바꾸고
            
        - 그걸 multipart로 업로드
            
    - 또는
        
        - multipart 전체를 하나의 바이너리 blob 으로 보고
            
        - **네가 직접** multipart 파서를 구현해서  
            암복호화 전/후에 정확히 동일 바이트 시퀀스로 복원
            
2. Spring 의 기본 MultipartResolver를 안 쓰고,
    
    - inputStream 직접 읽어서
        
    - 암복호화하고
        
    - boundary 기준으로 직접 파싱하는 커스텀 컴포넌트를 써야 함
        

==> 현실적으로 지금 프로젝트에서 이걸 하는 건 **과하다**고 봐야지.

---

## 6. 결론 (진짜 핵심만)

> ❌ **지금 네 구조(필터에서 request.getInputStream 읽어서 AES 복호화 + decodedParameters로 파라미터 재구성)는 multipart/form-data에 절대 맞지 않는 방식**이라서,
> 
> - multipart에 동일 로직을 적용하면  
>     → boundary/헤더/바이너리 구조가 깨져서  
>     → 파일/uuid/chunks 정보가 다 날아간다.
>     
> 
> ✅ 그래서 정답은:
> 
> - **multipart/form-data 요청은 암복호화 필터/래퍼 대상에서 제외하고,**
>     
> - 나머지 JSON / x-www-form-urlencoded / GET 쿼리 등만 암복호화하는 것.
>     

지금 네가 “multipart 는 필터에서 제외하자” 쪽으로 가고 있는 방향이 **정석**이야.  
그렇게 분리하는 게 맞고, 괜히 multipart까지 한 번에 처리하려고 하면 지금처럼 계속 지뢰 밟는 구조가 돼.


https://velog.io/@csh0034/Spring-MVC-MultipartResolver-CVE-2023-24998