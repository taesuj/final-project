ICIA 최종프로젝트 Ouir31
---
프로젝트 기간 : 2022.12 ~ 2023.01 <br><br>
프로젝트 인원 : 3명<br><br>
사용 프로그램 : visual studio, intellj, Mysql<br><br>
사용 언어 : Html, CSS, JavaScript, React, Java<br>
## 환경 구성
IDE(통합개발환경) : IntelliJ Ultimate(유료 버전), Visual Studio Code<br><br>
프레임워크 : Spring Boot<br><br>
데이터베이스 : Mysql<br><br>
DB 접근 기술 : JPA<br><br>
View 템플릿 : React<br>
## 프로젝트 설명<br>
실제로 카페 운영중인 지인에게 MD 배달과 음료 주문 서비스를 포함한 페이지 개설 요청으로 실행<br> 

- #### 메인페이지 화면<br><br>
![배경](https://user-images.githubusercontent.com/117873818/224891979-151b7c22-9630-4b7c-90c9-ae14e9c2df99.jpg)

## 제가 구현한 기능으로는<br>
- 로그인<br>
- 회원가입/회원탈퇴<br>
- 아이디/비밀번호 찾기<br>


※ 1. 로그인

## Back_EstimateController
```java
@PostMapping("/user/login")
    public Map<Object, Object> login(@RequestBody User user, HttpSession session){
        log.info("login()");
        return uServ.loginProc(user, session);
    }

    @PostMapping("/user/logout")
    @ResponseBody
    public ReturnMsg userLogout(HttpSession session){
        log.info("userLogout()");
        return uServ.userLogout(session);
    }
```
## Back_EstimateService
```java
public Map<Object, Object> loginProc(User user, HttpSession session){
        log.info("loginProc()");
        Map<Object, Object> result = new HashMap<>();

        User uData = null;
        uData = uRepo.findByUid(user.getUid());

        try {

            if(uData != null || user.getUid().equals("Admin")){
                if(user.getUid().equals("Admin")){
                    adminlogin(user,session);

                    result.put("msg" , "관리자계정으로 로그인되었습니다");
                    result.put("uid","Admin");
                    result.put("success", true);

                    redirect();
                    return result;
                }

                String cPwd = uData.getUpwd();

                if(encoder.matches(user.getUpwd(),cPwd)){
                    session.setAttribute("loginName",uData.getUname());
                    session.setAttribute("loginId",uData.getUid());
                    log.info((String)session.getAttribute("loginName"));
                    session.setMaxInactiveInterval(30*60);
                    result.put("msg" , "로그인 성공");
                    result.put("success", true);
                    result.put("uid", uData.getUid());

                    return result;
                }else {
                    result.put("msg" , "비밀번호를 확인해주세요");
                    result.put("success", false);

                }
            }else {
                result.put("msg" , "아이디를 찾을 수 없습니다");
                result.put("success", false);

            }

        } catch (Exception e) {
            e.printStackTrace();
            result.put("msg" , "로그인 실패");
            result.put("success", false);

        }

        return result;
    }
```

    {
    public ReturnMsg userLogout(HttpSession session){
        ReturnMsg rm = new ReturnMsg();
        session.invalidate();
        rm.setFlag(true);
        rm.setMsg("로그아웃 되었습니다.");
        return rm;
    }
}

<br><br>
- #### 로그인 화면<br><br>

![image](https://user-images.githubusercontent.com/117873818/225042607-04c467a4-7699-42ff-9837-4d57d6ef9317.png)


※ 2. 회원가입/회원탈퇴

## Back_EstimateController
```java

```
## Back_EstimateService
```java
public Map<Object, Object> loginProc(User user, HttpSession session){
        log.info("loginProc()");
        Map<Object, Object> result = new HashMap<>();

        User uData = null;
        uData = uRepo.findByUid(user.getUid());

        try {

            if(uData != null || user.getUid().equals("Admin")){
                if(user.getUid().equals("Admin")){
                    adminlogin(user,session);

                    result.put("msg" , "관리자계정으로 로그인되었습니다");
                    result.put("uid","Admin");
                    result.put("success", true);

                    redirect();
                    return result;
                }

                String cPwd = uData.getUpwd();

                if(encoder.matches(user.getUpwd(),cPwd)){
                    session.setAttribute("loginName",uData.getUname());
                    session.setAttribute("loginId",uData.getUid());
                    log.info((String)session.getAttribute("loginName"));
                    session.setMaxInactiveInterval(30*60);
                    result.put("msg" , "로그인 성공");
                    result.put("success", true);
                    result.put("uid", uData.getUid());

                    return result;
                }else {
                    result.put("msg" , "비밀번호를 확인해주세요");
                    result.put("success", false);

                }
            }else {
                result.put("msg" , "아이디를 찾을 수 없습니다");
                result.put("success", false);

            }

        } catch (Exception e) {
            e.printStackTrace();
            result.put("msg" , "로그인 실패");
            result.put("success", false);

        }

        return result;
    }
```

    {
    public ReturnMsg userLogout(HttpSession session){
        ReturnMsg rm = new ReturnMsg();
        session.invalidate();
        rm.setFlag(true);
        rm.setMsg("로그아웃 되었습니다.");
        return rm;
    }
}

<br><br>
- #### 로그인 화면<br><br>


