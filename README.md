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
![인증](https://user-images.githubusercontent.com/117873818/225111641-69d54eef-3b26-43d2-b695-48675f4a5677.png)

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

```
<br>
- #### 임시패스워드 <br><br>

비밀번호 찾기 기능에서 이메일 인증에 성공하면, 임시패스워드를 암호화 처리하여 발급하게 되며 자동으로 update하는 기능을 구현하였습니다.

![image](https://user-images.githubusercontent.com/117873818/225114883-7157d83f-d219-4aaf-8f76-f44dc312432c.png)

## React.join.js
```java
const Join = () => {
  const nav = useNavigate();

  const [chEmail, setChEmail] = useState(false)

  const [form, setForm] = useState({
    uid: "",
    upwd: "",
    uname: "",
    uadd: "",
    uemail: "",
    uphone: ""
  });
  const { uid, upwd, uname, uadd, uemail,uphone } = form;

    //이름, 이메일, 비밀번호, 비밀번호 확인
    const [id, setId] = useState('')
    const [password, setPassword] = useState('')
    const [name, setName] = useState('')
    const [add, setAdd] = useState('')
    const [email, setEmail] = useState('')
    const [phone, setPhone] = useState('')
  
    //오류메시지 상태저장
    const [idMessage, setIdMessage] = useState('')
    const [passwordMessage, setPasswordMessage] = useState('')
    const [nameMessage, setNameMessage] = useState('')
    const [addMessage, setAddMessage] = useState('')
    const [emailMessage, setEmailMessage] = useState('')
    const [phoneMessage, setPhoneMessage] = useState('')
  
    // 유효성 검사
    const [isId, setIsId] = useState(false)
    const [isPassword, setIsPassword] = useState(false)
    const [isName, setIsName] = useState('')
    const [isadd, setIsAdd] = useState('')
    const [isEmail, setIsEmail] = useState(false)
    const [isphone, setIsPhone] = useState('')
  
    // 아이디
    const onChangeId = useCallback((e) => {
      const formObj = {
        ...form,
        [e.target.name]: e.target.value,
      };
      const idRegExp = /^[a-zA-z0-9]{4,12}$/;    

      if (!idRegExp.test(e.target.value)) {
        setIdMessage('4-12사이 대소문자 또는 숫자만 입력해 주세요!');
        setIsId(false);
      } else {
        setIdMessage('사용가능한 아이디 입니다.');
        setIsId(true);
      }
      setForm(formObj);
    }, [form])

     // 비밀번호
    const onChangePassword = useCallback((e) => {
      const formObj = {
        ...form,
        [e.target.name]: e.target.value,
      };      
      const passwordRegex = /^(?=.*[a-zA-Z])(?=.*[!@#$%^*+=-])(?=.*[0-9]).{8,25}$/
      setPassword(e.target.value)
  
      if (!passwordRegex.test(e.target.value)) {
        setPasswordMessage('숫자+영문자+특수문자 조합으로 8자리 이상 입력해주세요!')
        setIsPassword(false)
      } else {
        setPasswordMessage('비밀번호 안정성 높음')
        setIsPassword(true)
      }
      setForm(formObj);

    }, [form])


    //이름
    const onChangeName = useCallback((e) => {
      const formObj = {
        ...form,
        [e.target.name]: e.target.value,
      };
   
      const nameRegex = /^[ㄱ-ㅎ|가-힣|a-z|A-Z|]{2,5}$/;
      
      if (!nameRegex.test(e.target.value)) {
        setNameMessage('2글자 이상 5글자 미만으로 입력해주세요.')
        setIsName(false)
      } else {
        setNameMessage('올바른 이름 형식입니다.')
        setIsName(true)
      }
      setForm(formObj);
    }, [form])
  
    // 이메일
    const onChangeEmail = useCallback((e) => {
      const formObj = {
        ...form,
        [e.target.name]: e.target.value,
      };
      const emailRegex =
        /([\w-.]+)@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.)|(([\w-]+\.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(\]?)$/
    
  
      if (!emailRegex.test(e.target.value)) {
        setEmailMessage('이메일 형식을 확인해주세요.')
        setIsEmail(false)
      } else {
        setEmailMessage('올바른 이메일 형식입니다.')
        setIsEmail(true)
      }
      setForm(formObj);
    }, [form])

    //핸드폰
    const onChangePhone = useCallback((e) => {
     
      // const phoneRegex = /^[a-zA-z0-9]{4,12}$/;  
      const phonenumber = e.target.value
      .replace(/[^0-9]/g, '')
      .replace(/^(\d{0,3})(\d{0,4})(\d{0,4})$/g, "$1-$2-$3").replace(/(\-{1,2})$/g, "");
      const formObj = {
        ...form,
        [e.target.name]: phonenumber,
      };
      setForm(formObj);
    }, [form])


  const emailcheck = () => {
   
    axios
        .get('/mail/check', {params:{uemail: form.uemail}})
        .then((rm) =>{
            console.log(rm.data);
            if(isEmail===true && rm.data.flag ===true){
              if(rm.data.flag === true){
                var cmsg = prompt("인증번호를 입력해주세요.")
                if(rm.data.msg === cmsg){
                  alert("이메일 인증 완료")
                  setChEmail(true)                  
                }else{
                    alert("인증번호를 확인해주세요.")
                    setChEmail(false)
                }
              }           
            }else if(rm.data.flag === false && isEmail ===true){
              alert("이미 가입된 이메일 입니다.")
              setChEmail(false)
            }
            else{
                alert("이메일을 형식에 맞게 입력해 주세요")
                setChEmail(false)
            }
        })
  };

  const sendJoin = (e) => {
    e.preventDefault();
    console.log(form)
    if(chEmail===false){
      alert('이메일 인증을 진행해주세요.')
      // e.stopPropagation()
      // e.preventdefault()
      return;}      
    axios
      .post('/user/join', form)
      .then((rm) => {
        if (rm.data.flag === true) {
          alert("가입 성공");
          nav("/home");
        }
      })
      .catch((error) => console.log(error));
  };

  // const onChange = useCallback(
  //   (e) => {
  //     const formObj = {
  //       ...form,
  //       [e.target.name]: e.target.value,
  //     };
  //     setForm(formObj);
  //   },[form]);

    
  return (
    <>
    <div className="Join">
      <form className="Content" onSubmit={sendJoin}>
        <h2 className="main_title">회원가입</h2>
        <input
          className="Input"
          name="uid"
          value={uid}
          placeholder="아이디"
          onChange={onChangeId}
          autoFocus
          required
        />        
        {uid.length > 0 && <span className={`message ${isId ? 'success' : 'error'}`}>{idMessage }</span>}        
        
        <input
          type="password"
          className="Input"
          name="upwd"
          value={upwd}
          placeholder="비밀번호"
          onChange={onChangePassword}
          required
        />
        {upwd.length > 0 && (<span className={`message ${isPassword ? 'success' : 'error'}`}>{passwordMessage}</span>)}
       
        <input
          className="Input"
          name="uname"
          value={uname}
          placeholder="이름"
          onChange={onChangeName}
          required
        />
        {uname.length > 0 && <span className={`message ${isName ? 'success' : 'error'}`}>{nameMessage}</span>}

          {/* <input
          className="Input"
          name="uadd"
          value={uadd}
          placeholder="주소"
          onChange={onChange}
          required
        /> */}
          <input
          className="Input"
          name="uemail"
          value={uemail}
          placeholder="이메일"
          onChange={onChangeEmail}
          required
        />
         {uemail.length > 0 && <span className={`message ${isEmail ? 'success' : 'error'}`}>{emailMessage}</span>}

         <Button color="ouir1" onClick={emailcheck}>
          이메일 인증
        </Button>
        <input
          maxlength="13"
          className="Input"
          name="uphone"
          value={uphone}
          placeholder="연락처"
          onChange={onChangePhone}
          required
        />      
        <Button type="submit">
          가입
        </Button>    
      </form>
    </div>
    </>
  );
};

export default Join;

```

## React.login.js
```java
import axios from "axios";
import React, { useCallback, useState } from "react";
import { useNavigate } from "react-router-dom";
import Button from "./etc/Button";
import "./etc/Button.scss";
import "./Login.scss";

const Login = ({ sucLogin }) => {
  const nav = useNavigate();
  const [form, setForm] = useState({
    uid: "",
    upwd: "",
  });
  const { uid, upwd } = form;

  const sendLogin = (e) => {
    e.preventDefault();

    axios
      .post("/user/login", form)
      .then((res) => {
        if (res.data.success === true) {
          const uid = res.data.uid;
          sucLogin(uid);
          //로그인 상태 유지(세션)
          sessionStorage.setItem("uid", uid);
          nav("/home");
        } else {
          alert("아이디나 비밀번호가 틀립니다.");
          const formObj = {
            uid: "",
            upwd: "",
          };
          setForm(formObj);
        }
      })
      .catch((err) => console.log(err));
  };

  const onChange = useCallback(
    (e) => {
      const formObj = {
        ...form,
        [e.target.name]: e.target.value,
      };
      setForm(formObj);
    },
    [form]
  );

  return (
    <div className="Login">
      <form className="Content" onSubmit={sendLogin}>
        <h2 className="main_title">로그인</h2>
        <input
          className="Input"
          name="uid"
          value={uid}
          placeholder="아이디"
          onChange={onChange}
          autoFocus
          required
        />
        <input
          type="password"
          className="Input"
          name="upwd"
          value={upwd}
          placeholder="비밀번호"
          onChange={onChange}
          required
        />
        <Button color="ouir1" type="submit" size="large">
          로그인
        </Button>
        <div className="U1">
        <li>아이디 찾기</li>
        <li>비밀번호 찾기</li>
        </div>
      </form>
    </div>
  );
};

export default Login;

```




마치며
---
#### 소감<br><br>
프로젝트를 시작하기 전 리액트를 3~4일 정도만 배웠어서 코드 자체가 번잡한 부분이 많은것 같습니다. 더 열심히 해서 점점 나아지는 모습 기대해주세요. 이 프로젝트는 저에게는 많은 경험이 되었고, 저의 부족한 부분과 제가 할 수 있는 부분을 구별할 수 있는 능력을 가지게 해주었다고 생각합니다. 그렇기에 부족한 부분은 더 공부하고, 할 수 있는 부분은 까먹지 않게 노력해야 될것 같습니다.

