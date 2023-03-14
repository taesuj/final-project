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
 @PostMapping("/user/join")
    public ReturnMsg userWrite(@RequestBody User user){
        log.info("userWrite()");
        return uServ.insertUser(user);
    }

```
## Back_EstimateService
```java
public ReturnMsg insertUser(User user) {
        log.info("insertUser()");
        ReturnMsg rm = new ReturnMsg();
        rm.setFlag(false);

        User udata = null;
        udata = uRepo.findByUid(user.getUid());
        User uemail = null;
        uemail = uRepo.findByUemail(user.getUemail());


        try{
            if(udata == null && uemail == null){
                user.setUpwd(encoder.encode(user.getUpwd()));
                uRepo.save(user);
                rm.setFlag(true);
            }else{
                rm.setFlag(false);
            }
        }catch (Exception e){
            e.printStackTrace();
            rm.setFlag(false);
        }
        return rm;
    }
```

    {
    public ReturnMsg userDelete(HttpSession session,String dCheck){
        User ddata = uRepo.findByUid(session.getId());
        ReturnMsg rm = new ReturnMsg();
        try {
            if(encoder.matches(dCheck,ddata.getUpwd())){
                uRepo.delete(ddata);
                session.invalidate();
                rm.setMsg("회원 탈퇴하였습니다.");
                rm.setFlag(true);
            }
        }catch (Exception e){
            rm.setMsg("회원 탈퇴 실패");
        }
        return rm;
        }
    }
}
## MailBack_EstimateController
```java
@PostMapping("/check/mailch")
    @ResponseBody
    public Map<String, Boolean> pw_find(String userEmail) {
        Map<String, Boolean> json = new HashMap<>();
        boolean pwFindCheck = mServ.userEmailCheck(userEmail);

        System.out.println(pwFindCheck);
        json.put("check", pwFindCheck);
        return json;
    }
    
 @GetMapping  ("/mail/check")
    @ResponseBody
    public ReturnMsg mailcheck(@RequestParam("uemail") String uemail) {
        ReturnMsg rm = new ReturnMsg();
        return mServ.createMailAndChangePassword(uemail);
    }
```
## MailBack_EstimateService
```java
public class MailService {


    @Autowired
    private UserRepository uRepo;
    @Autowired
    private JavaMailSender mailSender;

    private BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    private static final String FROM_ADDRESS = "ouir31@naver.com";
    public boolean userEmailCheck(String userEmail) {

        User user = null;
        user = uRepo.findByUemail(userEmail);

        try {
            if (user != null) {
                return true;
            } else {
                return false;
            }
        }catch (Exception e){
            return  false;
        }
    }

//////////////////////////////////////////////////////////////////////////

    public void mailSend(Mail maildto){
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(maildto.getAddress());
        message.setFrom(FROM_ADDRESS);
        message.setSubject(maildto.getTitle());
        message.setText(maildto.getMessage());

        mailSender.send(message);
    }
////////////////////////////////////////////////////////////////////

    public ReturnMsg createMailAndChangePassword(String uemail){
        ReturnMsg rm = new ReturnMsg();

        try {
            if (uRepo.findByUemail(uemail)==null){
                String str = getTempPassword();
                Mail maildto = new Mail();
                maildto.setAddress(uemail);
                maildto.setTitle("위어31 회원님의 회원가입 안내 이메일 입니다.");
                maildto.setMessage("안녕하세요. 위어31 회원님의 회원가입 안내 관련 이메일 입니다." + "회원님의 인증코드는 "
                        + str + " 입니다.");
                mailSend(maildto);
                rm.setMsg(str);
                rm.setFlag(true);
            }else {
                rm.setFlag(false);
            }
        }catch (Exception e){
            rm.setFlag(false);
        }
        return rm;
    }

    public void newpasswordupdate(String str, String uid){
        User user = uRepo.findByUid(uid);
        String pwd = encoder.encode(str);
        user.setUpwd(pwd);
        uRepo.save(user);
    }

    //임시 비밀번호 암호화 처리
    public static String getTempPassword() {
        char[] charSet = new char[]{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F',
                'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'};

        String str = "";

        int idx = 0;
        for (int i = 0; i < 10; i++) {
            idx = (int) (charSet.length * Math.random());
            str += charSet[idx];
        }
        return str;
    }

    public int checkMail(String uemail){
        return uRepo.countUserByUemail(uemail);
    }

}
```

<br><br>
- #### 회원가입/탈퇴<br><br>

회원가입시 이메일 중복 체크를 통해 다량의 아이디 생성 방지와 회원관리에 편의성을 추가하고자 하였습니다.
이메일 인증 시 임시코드를 임의로 메일로 발급하여 인증에 성공할 시 회원가입 버튼이 활성화 되도록 코드를 작성하였습니다.

![image](https://user-images.githubusercontent.com/117873818/225107022-e94685b3-61c6-4517-9f9f-c8e4226ee9e2.png)

※ 3. 아이디찾기/패스워드 

## Back_EstimateController
```java
@PostMapping("/user/idsearch")
    @ResponseBody
    public ReturnMsg userIdsearch(User uemail){
        log.info("userIdsearch()");
        return uServ.userIdsearch(uemail);
    }

    @PostMapping("/user/pwdsearch")
    @ResponseBody
    public ReturnMsg userPwdsearch(String email){
        return mServ.userpwdsearch(email);
    }
```
## Back_EstimateService
```java
public ReturnMsg userIdsearch(User uemail){
        ReturnMsg rm = new ReturnMsg();
        User sUemail = null;
        sUemail = uRepo.findByUemail(uemail.getUemail());
        log.info("sUemail");

        try {
            if (sUemail != null){
                String sId = sUemail.getUid();
                rm.setMsg(sId);
                rm.setFlag(true);

            }else {
                rm.setMsg("해당 email은 존재하지 않습니다.");
            }
        }catch (Exception e){
            rm.setMsg("Error");
        }

        return rm;
    }
```

 {
 
   public ReturnMsg userpwdsearch(String email) {
        ReturnMsg rm = new ReturnMsg();

        User user = null;
        user = uRepo.findByUemail(email);

        try {
            if (uRepo.findByUemail(email)!=null){
                String str = getTempPassword();
                Mail mail = new Mail();
                mail.setAddress(email);
                mail.setTitle("위어31"+user.getUname()+"님의 개인정보 안내 메일입니다.");
                mail.setMessage("안녕하세요. 위어31"+user.getUname()+"님의 임시패스워드 안내 이메일 입니다." + "회원님의 인증코드는 "
                        + str + " 입니다.");
                mailSend(mail);
                newpasswordupdate(str,user.getUid());
                rm.setMsg("임시패스워드 발급 성공");
            }
        }catch (Exception e){
            rm.setMsg("실패하였습니다. 다시 시도하여 주십시오.");
        }
        return rm;
    }

    public void newpasswordupdate(String str, String uid){
        User user = uRepo.findByUid(uid);
        String pwd = encoder.encode(str);
        user.setUpwd(pwd);
        uRepo.save(user);
    }

    //임시 비밀번호 암호화 처리
    public static String getTempPassword() {
        char[] charSet = new char[]{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F',
                'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'};

        String str = "";

        int idx = 0;
        for (int i = 0; i < 10; i++) {
            idx = (int) (charSet.length * Math.random());
            str += charSet[idx];
        }
        return str;
    }
}

<br><br>
- #### 임시패스워드 <br><br>



마치며
---
#### 소감<br><br>
프로젝트를 시작하기 전 리액트를 3~4일 정도만 배웠어서 코드 자체가 번잡한 부분이 많은것 같습니다. 더 열심히 해서 점점 나아지는 모습 기대해주세요. 이 프로젝트는 저에게는 많은 경험이 되었고, 저의 부족한 부분과 제가 할 수 있는 부분을 구별할 수 있는 능력을 가지게 해주었다고 생각합니다. 그렇기에 부족한 부분은 더 공부하고, 할 수 있는 부분은 까먹지 않게 노력해야 될것 같습니다.

