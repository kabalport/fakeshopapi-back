# JWT, Spring Security, Spring Data JPA, Spring MVC를 이용한 Fake Shop API만들기

### 내가 만들 사이트를 고민한다.

- 메인페이지
  - 첫번째 페이지의 상품 목록을 보여준다.
  - 카테고리 목록을 보여준다.
  - 카테고리를 선택하면 해당 카테고리의 상품 목록을 보여준다. 페이징 처리를 한다.
- 로그인 페이지
  - 아이디(email)과 암호를 입력하여 로그인한다.
- 회원가입페이지
  - 이름, 이메일, 암호, 생년월일, 성별을 입력하여 회원가입한다.
- 회원가입 환영 페이지
  - 회원 가입 환영 메시지를 보여준다.
- 마이페이지
  - 현재 로그인한 사용자 정보를 보여준다.

### 내가 만들 페이지는 어떻게 고민할까?

- 종이 노트, 패드 등에 그림을 그린다. 프로토타이핑을 만든다.
- 되도록이면 디테일하게 그린다. 처음부터 사용자 동선에 맞춰서 그려나간다.

### 아키텍처를 고민한다.

- 아키텍처는 선택의 연속, 트레이드 오프
- 기술은 굉장히 다양하고, 각각의 기술은 장단점이 있다.
  - 예) 백엔드를 Node.js 로 만들 수도 있고 java로 만들수도 있다. 이때 우리에게 유리한 장점이 무엇인가? 단점이 큰가? 장점이 너무 큰가?
  - 회사는 놀이터가 아니다. 내가 하고 싶은 기술을 하는 곳이 아니라, 진짜 필요한 기술을 선택하여 사용하는 것이다.
- 우리는 학습 위해 Java, Spring Boot, MySQL을 사용한다.
  - 가장 많은 회가가 원하니깐, 백엔드 개발자로
  - SI 회사에서도 Java 개발자를 많이 구한다.

### 그렇다면? 어떻게 프로젝트를 만들고 개발을 진행할까?

- Rest API (Web API) 에 대한설계를 한다.
- URL 주소를 정의한다.

예)

상품목록을 조회한다.

url:/products

파라미터 : page(기본값 0), categoryId(기본값은 0)

응답: 응답코드: 200

Body: 상품목록, 전체페이지수, 현재페이지의 상품건수

상품목록- 상품(imageUrl, 상품명, 가격, 상품id) 목록

### 요청과 응답을 처리하는 Rest API를 작성한다.

가짜 api 가 만들어짐

db도 사용하지않고, 스프링 시큐리티도 사용하지않고 동작하는 가짜 Rest API를 미리 만든다.

- 프론트 개발자는 가짜 api로 개발할 수 있다.
- 백엔드 개발자는 프론트 개발자가 가짜 api로 개발하는 동안에 진짜 api를 만든다.

### 진짜 api는 어떻게 만들까?

- ProductService
  - Page<Product>getProducts(int categroyId, int page)
    - categoryId가 0일때 모든 상품을 리턴한다.
    - page가 0일때는 첫페이지
- 카테고리 엔티티가 필요하다.
- Product는 엔티티로 만들자. products 테이블에 저장하도록 하자. 테이블 이름은 복수형 or 단수형 → 상관없지만 일관성있게 만들자
- Database를 사용한다면 Spring

- 카테고리 목록을 저장할 insert 문장을 만들고, 상품정보를 저장할 insert 문장을 만든다.

### 로그인?
- Oauth2 인증으로 구글 로그인, Github 로그인등을 붙일 수 있다.
  - Spring Security 연동
- Oauth2 인증 + 추가 정보
  - 예) 구글로 로그인, 전화번호인증 회원가입…
- 가장 쉬운 방법
  - 쿠키, 세션을 직접 이용
- Spring Security 를 이용할 수 있다.
  - login api를 호출하도록한다.
  - JWT 토큰…

- Member - Role (N:M)
  - 다대다
  - Spring Security의 전반적인 이해가 필요하다.

- 로그인
  - /login


## 인증공부
1. 사용자는 http Request로 id(username)과 password를 가진 상태로 접근한다.
2. Authentication Filter는 사용자로부터 받은 id(username)과 password를 이용하여 UsernamePasswordAuthenticationtoken을 만든다.
3. 그렇게 형성된 Token을 Authentication Manager로 보내주게 되면 Authentication Manager는 Token에 저장된 username을 4,5,6 번 과정을 거쳐 DB에 존재하는 지 확인하게 된다.
4. DB의 user 존재여부를 확인한 Authentication Manager는 username 해당 유저가 있다면 UsernamePAsswordAuthenticationToken에서 password를 db에 저장된 형태와 같은 해쉬함수를 사용하여 암호화 시킨후 db의 user객체의 password와 비교한다.
5. 모든 비교가 성공하면, Authentication Manager는 Authentication이라는 객체를 형성하여 SecurityContextHolder 내부의 SecurityContext에 저장하여 세션값을 유지시킨다.
### AuthenticationManager
- AuthenticationManager는 인증을 처리하는 방법을 정의한 api이다.
  - AuthenticationFilter에 의해 AuthenticationManager가 동작한다.
  - 인증을 처리하면 SecurityContextHolder에 Authentication 값이 세팅된다.
- JWT 인증을 위해 위의 기능을 하는 JWTAuthenticationFilter를 만들고, 해당 필터 안에서 AuthenticationManager를 사용한다.
### AuthenticationProvider
- ProviderManager는 AuthenticationManager의 가장 일반적인 구현체이다.
- ProviderManager는 AuthenticationProvider 목록을 위임 받는다. 각 AuthenticationProvider는 인증 성공, 실패, 결정할 수 없음을 나타낼 수 있고 나머지 AuthenticationProvider가 결정을 할 수 있도록 전달한다.
- 각각의 AuthenticationProvider는 특정 유형의 인증을 수행한다.
- JWTAuthenticationProvider를 만들고 JWT인증을 수행하도록 한다.

# jwt
- jwt(json web token)는 양측이 정보를 json개체로 안전하게 전송하기 위한 간결하고 독립적인방법을
정의하는 개방형 표준(RFC 75519)이다.
- 이 정보는 디지털 서명되어 있으므로 확인하고 신뢰할 수 있다.
jwt는 비밀(hmac 알고리즘 포함) 또는 RSA 또는 ECDSA 를 사용하는 공개/개인 키 쌍을 사용 하여 서명할 수 있다.
- JWT를 암호화하여 당사자 간에 비밀성을 제공할 수도 있지만 여기서는 서명 된 토큰에 중점을 둘 것이다.
서명된 토큰은 그안에 포함된 클레임의 무결성을 확인할 수 있는 반면 암호화된 토큰은 다른 당사자로부터 해당 클레임을 숨긴다.
공개/개인 키 쌍을 사용하여 토큰에 서명할 때 서명은 개인 키를 보유한 당사자만이 서명한당사자임을 인증한다.
# json web token은 언제 사용하는가?
- 권한부여: jwt를 사용하는 가장 일반적인 시나리오이다. 
  - 사용자가 로그인하면 이후의 각 요청에는 jwt가 포함되어 사용자가 해당 토큰으로
  - 허용된 경로, 서비스 및 리소스에 엑세스할 수 있는 Single sign on은 오버헤드가 적고 다양한 도메인에서 쉽게 사용할 수 있기 때문에
  - 현재jwt를 널리 사용하는 기능이다.
- 정보교환: json 웹 토큰은 당사자 간에 정보를 안전하게 전송하는 좋은 방법이다.
  - 예를 들어 공개/개인 키 쌍을 사용하여 jwt에 서명할 수 있으므로 보낸 사람이 누구인지 확인할 수 있다.
  - 또한 헤더와 페이로드를 사용하여 서명을 계산하므로 콘텐츠가 변조되지 않았음을 확인할 수 도 있다.

# jwt의 구조
- Header 헤더
- Payload 내용
- Signature 서명

- header
  - 헤더는 일반적으 토큰 유형(JWT)과 사용중인 서명 알고리즘(예: HMAC SHA256 또는 RSA)의 두 부분으로 구성된다.
```json
{
  "alg": "HS256",
  "typ": "JWT"
        }
```
- Payload
  - 토큰의 두 번째 부분은 클레임(claims)을 포함하는 페이로드이다. 클레임은 엔티티(일반적으로 사용자) 및 추가 데이터에 대한 설명이다.
  - 클레임에는 등록된 클레임, 공개 클레임 및 비공개 클레임의 세 가지 유형이 있다.


- JWT의 구조
  - 등록된 클레임(Registered claims): 필수는 아니지만 유용하고 상호 운용 가능한 클레임을 제공하기 위해
  - 권장되는 미리 정의된 클레임 집합이다. 그들 중 일부는 iss(발급자), exp(만료 시간), sub(주제), aud(대상) 및 기반(others) 이다.
  - 비공개 클레임 : 사용에 동의한 당사자 간에 정보를 공유하기 위해 생성된  사용자 정의 클레임이며 등록 또는 공개 클레임이 아니다.
  - 페이로드의 예
```json
{
  "sub": "12344567890",
  "name": "John Doe",
  "admin": true
}
```
- 그런 다음 페이로드는 Base64Url 로 인코딩되어 JSON 웹 토큰의 두 번째 부분을 형성한다.
# 서명(Signature)
- 서명 부분을 생성하려면 인코딩된 헤더, 인코딩된 페이로드, a secret, 헤더에 지정된 알고리즘을 가져와서 서명해야한다.
- 예를 들어 HMAC SHA256 알고리즘을 사용하려는 경우 서명은 다음과 같은 방식으로 생성된다.
- 서명은 도중에 메시지가 변경되지 않았는지 확인하는 데 사용되며 개인 키로 서명된 토큰의 경우 jwt 발신자가 누구인지 확인할 수도 있다.

## role테이블에 다음의 정보를 미리 insert해야 합니다. (Spring Boot 애플리케이션이 실행될때 자동으로 추가되도록 수정됨)

```
insert into role(name) values('ROLE_USER');
insert into role(name) values('ROLE_ADMIN');
```




## 회원 가입

```
curl --location --request POST 'localhost:8080/members/signup' \
--header 'Content-Type: application/json' \
--data-raw '{
"name":"이름",
"email":"이메일",
"password":"암호"
}'
```

---

## 로그인

```
curl --location --request POST 'localhost:8080/members/login' \
--header 'Content-Type: application/json' \
--data-raw '{
    "email":"email",
    "password":"암호"
}'
```

응답 형식

```
{
    "accessToken": "JWT토큰",
    "refreshToken": "JWT토큰",
    "memberId": memberId,
    "nickname": "이름"
}
```

---

## 로그아웃

```
curl --location --request DELETE 'http://localhost:8080/members/logout' \
--header 'Authorization: Bearer accessToken' \
--header 'Content-Type: application/json' \
--data '{
    "refreshToken" : "리프래시토큰"
}'
```
---

## 리프래시 토큰

```
curl --location 'http://localhost:8080/members/refreshToken' \
--header 'Content-Type: application/json' \
--data '{
"refreshToken" : "리프래시토큰"
}'
```

응답값

```
{
    "accessToken": "JWT토큰",
    "refreshToken": "JWT토큰",
    "memberId": memberId,
    "nickname": "이름"
}
```


## sample data (직접 입력하세요.)

insert into category (id, name) values (1, '상의');
insert into category (id, name) values (2, '하의');
insert into category (id, name) values (3, '아우터');
insert into category (id, name) values (4, '신발');
insert into category (id, name) values (5, '가방');
insert into category (id, name) values (6, '악세서리');
insert into category (id, name) values (7, '언더웨어');
insert into category (id, name) values (8, '수영복');
insert into category (id, name) values (9, '잠옷');
insert into category (id, name) values (10, '스포츠웨어');



-- 상의
INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('면 반팔 티셔츠', 10000, '편안한 면 소재의 반팔 티셔츠', 1, 'https://via.placeholder.com/400x600', 4.5, 100);

INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('린넨 셔츠', 20000, '시원한 린넨 소재의 셔츠', 1, 'https://via.placeholder.com/400x600', 4.0, 50);

-- 하의
INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('청바지', 30000, '편안한 착용감의 청바지', 2, 'https://via.placeholder.com/400x600', 4.2, 80);

INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('면 슬랙스', 25000, '면 소재의 슬랙스', 2, 'https://via.placeholder.com/400x600', 4.5, 60);

-- 아우터
INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('트렌치코트', 90000, '고급스러운 트렌치코트', 3, 'https://via.placeholder.com/400x600', 4.7, 30);

INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('가디건', 40000, '편안한 가디건', 3, 'https://via.placeholder.com/400x600', 4.0, 40);

-- 신발
INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('런닝화', 60000, '편안한 착용감의 런닝화', 4, 'https://via.placeholder.com/400x600', 4.3, 120);

INSERT INTO product (title, price, description, category_id, image_url, rate, count)
VALUES ('로퍼', 80000, '멋스러운 로퍼', 4, 'https://via.placeholder.com/400x600', 4.6, 70);



