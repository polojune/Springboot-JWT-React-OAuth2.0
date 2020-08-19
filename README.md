# 스프링부트 시큐리티 + JWT

# Spring JWT 서버로 OAuth까지 React와 연동

## 주의사항

- 시큐리티가 동작할 때만 CORS가 작동하여 차단함.
- SecurityConfig.java에 addFilter()로 cors 설정을 걸어줘야 함.

## Client Credentails Grant Type 방식 사용

- https://cheese10yun.github.io/spring-oauth2-provider/

- 기본구성은 Spingboot서버는 JWT토큰 생성기
- OAuth2.0 로그인은 React에서 함. https://www.npmjs.com/package/react-google-login
- React에서 로그인되면 그 정보를 그대로 스프링으로 던짐
- 스프링에서 그걸 받아서 회원가입 시키고 회원정보를 세션에 담음
- 그리고 JWT 토큰 만들어서 response의 body에 담아서 전송
- React에서 JWT 토큰 받아서 localStorage에 저장.
- 인증이 필요한 요청시마다 header에 JWT토큰 담아서 요청.

![blog](https://postfiles.pstatic.net/MjAyMDA4MTlfMTA0/MDAxNTk3NzY2MzA0Nzk2.lP9wYniN_9lQrAnkG5gmPWlDS1cNhLr8JUgk5cXwnAUg.ALy_HtvN3xvy4KAC14PNOq0H0I7-yZT78E3L1ciRilQg.JPEG.getinthere/Screenshot_2.jpg?type=w773)

## 라이브러리

- yarn add axios
- npm install react-google-login

## 설치 및 환경설정

- git에서 내려 받으면 node-module 폴더가 없음.
- npm install 명령어 입력하여 node-module 폴더 생성해서 테스트 해야함.

## 참고

- https://velog.io/@seize/React-%EC%B9%B4%EC%B9%B4%EC%98%A4-%EC%86%8C%EC%85%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8#8-kakaoauthlogin-%ED%95%A8%EC%88%98%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EC%B9%B4%EC%B9%B4%EC%98%A4-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%ED%8C%9D%EC%97%85%EC%B0%BD-%EC%B6%9C%EB%A0%A5-%EB%B0%8F-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%B2%98%EB%A6%AC


## 리엑트 연동 참고
https://bezkoder.com/spring-boot-react-jwt-auth/

## 인증

### UsernamePasswordAuthenticationFilter 등록

- attemptAuthentication() 함수를 오버라이딩 하고 아래와 같이 구현한다.
- request의 username과 password를 ObjectMapper로 받는다.
- 해당 username과 password로 UsernamePasswordAuthenticationToken을 생성한다.
- UsernamePasswordAuthenticationToken으로 Authentication 객체를 만든다.
- Authentication객체를 만들때 자동으로 UserDetailsService가 호출된다.
- 그렇기 때문에 UserDetailsService를 상속하여 직접 서비스를 구현한다.
- UserDetailsService를 통해서 리턴될 UserDetails을 커스텀해서 구현한다.

### AuthenticationProvider 관련 팁

- authenticate() 함수가 호출 되면 인증 프로바이더가 유저 디테일 서비스의
- loadUserByUsername(토큰의 첫번째 파라메터) 를 호출하고
- UserDetails를 리턴받아서 토큰의 두번째 파라메터(credential)과
- UserDetails(DB값)의 getPassword()함수로 비교해서 동일하면
- Authentication 객체를 만들어서 필터체인으로 리턴해준다.
- Tip: 인증 프로바이더의 디폴트 서비스는 UserDetailsService 타입
- Tip: 인증 프로바이더의 디폴트 암호화 방식은 BCryptPasswordEncoder
- 결론은 인증 프로바이더에게 알려줄 필요가 없음.

### AuthenticationProvder 커스터 마이징 방법

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(authenticationProvider());
    }

    @Bean
    DaoAuthenticationProvider authenticationProvider(){
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setPasswordEncoder(passwordEncoder());
        daoAuthenticationProvider.setUserDetailsService(userPrincipalDetailsService);

        return daoAuthenticationProvider;
    }
```

## 인가

- Tip : JWT를 사용하면 UserDetailsService를 호출하지 않기 때문에 @AuthenticationPrincipal 사용 불가능.왜냐하면 @AuthenticationPrincipal은 UserDetailsService에서 리턴될 때 만들어지기 때문이다.

- Tip : 토큰 검증 (이게 인증이기 때문에 AuthenticationManager도 필요 없음)

- Tip : 스프링 시큐리티가 수행해주는 권한 처리를 위해 아래와 같이 토큰을 만들어서 Authentication 객체를 강제로 만들고 그걸 세션에 저장!

```java
    PrincipalDetails principalDetails = new PrincipalDetails(user);
    Authentication authentication =
            new UsernamePasswordAuthenticationToken(
                    principalDetails, //나중에 컨트롤러에서 DI해서 쓸 때 사용하기 편함.
                    null, // 패스워드는 모르니까 null 처리, 어차피 지금 인증하는게 아니니까!!
                    principalDetails.getAuthorities());

    // 강제로 시큐리티의 세션에 접근하여 값 저장
    SecurityContextHolder.getContext().setAuthentication(authentication);
```

![blog](https://postfiles.pstatic.net/MjAyMDA4MTBfMzQg/MDAxNTk3MDM2OTc1NjQ0.3bgXzd_Bf7JoS1fsYIyGP1DAl9kQZ8IA-_WW74GyaFcg.Vtp4R4c4X1zakxFzEk212VqkTsQhI0bRmPZft9ZQ92og.PNG.getinthere/Screenshot_31.png?type=w773)

![blog](https://postfiles.pstatic.net/MjAyMDA4MTBfMjMy/MDAxNTk3MDM2OTc1NjM2.vXqNYRrbfievaF0YrELs8Rj-QW5gMmkoXRmIor3VDrEg.VR5lD5t-6T6FiFXd5bEopgLPR02oSuvzCjYNVFPlqaYg.PNG.getinthere/Screenshot_32.png?type=w773)
