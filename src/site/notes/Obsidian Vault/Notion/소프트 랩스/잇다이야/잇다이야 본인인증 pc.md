---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///pc/"}
---



### 기본 흐름

#### 라우터
```
ex)
$route['Membership/join'](맵핑주소 사이트 경로) = 'Member/Join/join'; 컨트롤러 연결
var/www/itdia_www(pc)/application/controller(컨트롤러 기본경로)
/Member/join(파일명)/join(클레스명) 연결

```

#### 컨트롤러
```
기본 경로 
ex)
application/controoler/Member/join.php

뷰페이지 로드 , 로직 처리 
ex)
$this->layout->page('Member/Join/join')

```

#### 뷰
```
ex)
apllication/views
/pc/Member/join/join.php

본인 인증 ajax통신
```





### 작업내용

#### 라우터 설정
```

//  일회용 본인인증  Params 생성  
$route['User/Identity_Verification_Processing_Ones'] = 'Common/Identity_Verification_Processing_Ones/getParams';       

 //  본인인증 리턴 페이지
$route['User/IdentityVerificationResult_Ones'] = 'Common/Identity_Verification_Ones/IdentityVerificationResult';       
```
#### 시작페이지 
ar/www/itDia_www/application/views/PC/New/index.php (new 서브도메인 시작)

아래 Identity_Verification_Processing_Ones 값으로 팝업 본인인증 시작
완료시 아래에서 설정한 
$IVresultURL."/User/IdentityVerificationResult_Ones 이동

IdentityVerificationResult_Ones여기서 뷰페이지 로드
$this->layout->page(Fn_Device_View(), 'loadingView', "New/result", $resultDB, $resultData, $this->itDia_session)

뷰페이지에서 본인인증 성공시 
window.opener.postMessage({ verified: true }, "https://new.itdia.co.kr");
시작페이지로 응답값전송
시작페이지에서 응답이 성공적이라면 ajax 세션저장 요성 
set_verification_session.php
set_verification_session.php 페이지에서 세션값 저장

완료후 리로드 


```
세션 체크
세션값 있다면 본사이트로 리다이렉트

없다면 본인인증
ajax 요청 

  
IdentityVerification.prototype.DisposableCertification = function(sendData){  
    var params = {"pageType" : this.pageMainType, "data" : sendData};  
  
    console.log("DisposableCertification");  
    $.ajax({  
        type: "POST",  
        url: "https://www.itdia.co.kr/User/Identity_Verification_Processing_Ones",  
        data: params,  
        dataType: "json",  
        async: false,  
        success:function(response){  
  
            if (response.result){  
                var KMCobj = response.params;  
                console.log(KMCobj)  
                var form = document.createElement("form");  
                form.setAttribute("charset", "UTF-8");  
                form.setAttribute("method", "Post");  //Post 방식  
                form.setAttribute("action", "https://www.kmcert.com/kmcis/web/kmcisReq.jsp"); //요청 보낼 주소  
  
                var hiddenField = document.createElement("input");  
                hiddenField.setAttribute("type", "hidden");  
                hiddenField.setAttribute("name", "tr_cert");  
                hiddenField.setAttribute("value", KMCobj.enc_tr_cert);  
                form.appendChild(hiddenField);  
  
                hiddenField = document.createElement("input");  
                hiddenField.setAttribute("type", "hidden");  
                hiddenField.setAttribute("name", "tr_url");  
                hiddenField.setAttribute("value", KMCobj.tr_url);  
                form.appendChild(hiddenField);  
  
                hiddenField = document.createElement("input");  
                hiddenField.setAttribute("type", "hidden");  
                hiddenField.setAttribute("name", "tr_add");  
                hiddenField.setAttribute("value", KMCobj.tr_add);  
                form.appendChild(hiddenField);  
  
                document.body.appendChild(form);  
  
                if (response.ConnectionDevice.toUpperCase() == "pc".toUpperCase()){  
                    window.name = "itDiaMainWindow";  
                    KMCIS_window = window.open('', 'KMCISWindow', 'width=425, height=550, resizable=0, scrollbars=no, status=0, titlebar=0, toolbar=0, left=435, top=250' );  
                    form.target = 'KMCISWindow';  
                }  
  
                form.submit();  
            } else {  
                if (response.message != ""){  
                    alert(response.message);  
                } else {  
                    alert("죄송합니다.\n서버 오류로 인해 휴대폰 본인인증을 할수 없습니다.");  
                }  
            }  
        },  
        error:function(error){  
            alert("서버와 통신중 오류가 발생하였습니다.\n[" + error.status + "]" + error.statusText);  
        }  
    });  
}

```

#### Identity_Verification_Processing_Ones(본인인증 파라메터 생성)

```
본인인증 파라메터 생성
 /*  
 description : 일회용 본인인증 > 파라메터 만들기  
 Date of creation : 2023.10.04 
 Writer : Fall return value : json  
 Record ========================================== [Date, Writer] description */ public function getParams(){  
     $userSessionData = $this->itDia_session;  
     $result = array(  
         "result" => FALSE,  
         "message" => "",  
         "params" => array(  
             "enc_tr_cert" => $userSessionData,  
             "tr_url" => "",  
             "tr_add" => ""  
         ),  
         "ConnectionDevice" => Fn_Device_Connection()  
     );  
  
     $getBackData = $pageType = $this->input->post_get("pageType", true) ? trim($this->input->post_get("pageType", true)) : "";  
     $data = $this->input->post_get("data", true);  
     $VerificationCode = "";  
  
     if ($pageType != ""){  
         //  Start : KMC url 인증 코드  
         $domain = "";  
         $IVresultURL = PROTOCOL_TYPE."://";  
  
         if (Fn_Device_Connection() == Fn_Device_View()){  
             //  접속 디바이스와 보여주는 웹페이지가 같음  
             $domain = HTTP_HOST;  
             $IVresultURL .= HTTP_HOST;  
         } else {  
             //  접속 디바이스와 다른 웹페이지임.  
             if (Fn_Device_Connection() == strtoupper("PC")){  
                 //  1. PC > Mobile  
                 $domain = PC_DOMAIN;  
                 $IVresultURL .= PC_DOMAIN;  
             } else if (Fn_Device_Connection() == strtoupper("MOBILE")){  
                 //  2. Mobile > PC  
                 $domain = MOBILE_DOMAIN;  
                 $IVresultURL .= MOBILE_DOMAIN;  
             }  
         }  
  
         $VerificationCode = Fn_IdentityVerificationCode($domain, $pageType);  
         //  End : KMC url 인증 코드  
$VerificationCode = "005001";  
  
         if ($VerificationCode != ""){  
             if (strtoupper($pageType) == strtoupper("ONE_JOIN")){  
                 //  회원가입 > 쿠키로 입력된 정보 저장 (모바일 기기일때만)  
                 if (Fn_Device_Connection() == strtoupper("MOBILE")){  
                     $joinCookie = serialize($data);  
  
                     if (get_cookie(TEMP_COOKIE_JOIN)) {  
                         delete_cookie(TEMP_COOKIE_JOIN);  
                     }  
  
                     set_cookie(TEMP_COOKIE_JOIN, base64_encode($joinCookie), 0);  
                 }  
             }  
  
             try{  
                 //요청 번호 생성  
                 $CurTime = date('YmdHis');  
                 $RandNo = rand(100000, 999999);  
  
                 $reqNum = $CurTime.$RandNo;  
  
                 //01.입력값 변수로 받기  
                 $cpId = "IDAM1001";        // 회원사ID  
                 $urlCode = $VerificationCode;     // URL 코드  
                 $certNum = $reqNum;     // 요청번호  
                 $date = $CurTime;        // 요청일시  
                 $certMet = "M";     // 본인인증방법 “M”:휴대폰본인확인▪ “P”:공인인증서  
  
                 $birthDay = "";   // 생년월일  
                 $gender = "";    // 성별  
                 $name = "";        // 성명  
                 $phoneNo = "";       // 휴대폰번호  
                 $phoneCorp = "";  // 이동통신사  
                 $nation = "";      // 내외국인 구분  
  
                 $plusInfo   = $getBackData;   // 추가DATA정보  
                 $extendVar  = "0000000000000000";       // 확장변수  
  
                 $tr_url     = $IVresultURL."/User/IdentityVerificationResult_Ones";      // 본인인증 결과수신 POPUP URL                 $tr_add     = "N";      // IFrame사용여부  
  
                 //02. tr_cert 데이터변수 조합 (서버로 전송할 데이터 "/"로 조합)  
                 $tr_cert = $cpId . "/" . $urlCode . "/" . $certNum . "/" . $date . "/" . $certMet . "/" . $birthDay . "/" . $gender . "/" . $name . "/" . $phoneNo . "/" . $phoneCorp . "/" . $nation . "/" . $plusInfo . "/" . $extendVar;  
  
                 if (extension_loaded('ICERTSecu')) {  
                     //03. 1차암호화  
                     $enc_tr_cert = ICertSeed(1,0,'',$tr_cert);  
  
                     //04. 변조검증값 생성  
                     $enc_tr_cert_hash = "abcdabcdabcdabcdabcdabcdabcdabcdabcdabcd";  
                     //$enc_tr_cert_hash = ICertHMac($enc_tr_cert);  
  
                     $enc_tr_cert = $enc_tr_cert . "/" . $enc_tr_cert_hash . "/" . "0000000000000000";  
                     $enc_tr_cert = ICertSeed(1,0,'',$enc_tr_cert);  
  
                     $result["result"] = TRUE;  
                     $result["params"]["enc_tr_cert"] = $enc_tr_cert;  
                     $result["params"]["tr_url"] = $tr_url;  
                     $result["params"]["tr_add"] = $tr_add;  
                 } else {  
                     $result["message"] = "죄송합니다.\n서버 오류로 인해 휴대폰 본인인증을 할수 없습니다.";  
                 }  
             } catch(Exception $e){  
                 $s = $e->getMessage().'(오류코드:'.$e->getCode().')';  
                 $result["message"] = "죄송합니다.\n서버 오류로 인해 휴대폰 본인인증을 할수 없습니다.\n[".$s."]";  
             }  
         } else {  
             $result["message"] = "잘못된 요청입니다.지속적으로 발생시 관리자에게 문의하시기 바랍니다.[001]";  
         }  
     } else {  
         $result["message"] = "잘못된 요청입니다.지속적으로 발생시 관리자에게 문의하시기 바랍니다.[000]";  
     }  
  
     echo json_encode($result);  
 }


```

####  Identity_Verification_Ones.php 본인인증 결과 처리
```
 /*  
 description : 일회용 본인인증 > 결과값  
 Date of creation : 2023.10.04 
 Writer : Fall return value : html  
 Record ========================================== [Date, Writer] description */ public function IdentityVerificationResult(){  
     $resultDB = array();  
     $resultData = array(  
         "header" => array(  
             "title" => "휴대폰 본인인증 처리"  
         ),  
         "data" => array(  
             "result" => false,  
             "userName" => "",  
             "ivResultIndex" => 0,  
             "userMobileNumber" => "",  
             "returnPageCode" => "",  
             "returnPageSubCode" => "",  
             "returnPageURL" => "/",  
             "message" => "",  
             "memIndex" => 0  
         )  
     );  
  
     $userSession = $this->itDia_session;  
  
     $rec_cert = $this->input->get_post("rec_cert", true);  
     $iv = $cookieCertNum  = $this->input->get_post("certNum", true);  
  
     if (extension_loaded('ICERTSecu')){  
         //01.인증결과 1차 복호화  
$rec_cert = ICertSeed(2,0,$iv,$rec_cert);  
  
//02.복호화 데이터 Split (rec_cert 1차암호화데이터 / 위변조 검증값 / 암복화확장변수)  
$decStr_Split = explode("/", $rec_cert);  
  
$encPara  = $decStr_Split[0];     //rec_cert 1차 암호화데이터  
$encMsg   = $decStr_Split[1];     //위변조 검증값  
  
//03.인증결과 2차 복호화  
$rec_cert = ICertSeed(2,0,$iv,$encPara);  
  
         //04. 복호화 된 결과자료 "/"로 Split 하기  
         $decStr_Split = explode("/", $rec_cert);  
  
         $certNum    = $decStr_Split[0];  
         $date       = $decStr_Split[1];  
         $CI         = $decStr_Split[2];  
         $phoneNo    = $decStr_Split[3];  
         $phoneCorp  = $decStr_Split[4];  
         $birthDay   = $decStr_Split[5];  
         $gender     = $decStr_Split[6];  
         $nation     = $decStr_Split[7];  
         $name       = iconv("EUC-KR","UTF-8", $decStr_Split[8]);  
         $result     = $decStr_Split[9];  
         $certMet    = $decStr_Split[10];  
         $ip         = $decStr_Split[11];  
         $M_name     = $decStr_Split[12];  
         $M_birthDay = $decStr_Split[13];  
         $M_Gender   = $decStr_Split[14];  
         $M_nation   = $decStr_Split[15];  
         $plusInfo   = $decStr_Split[16];  
         $DI         = $decStr_Split[17];  
  
         //05. CI,DI 복호화  
         if(strlen($CI) > 0){  
             $CI = ICertSeed(2,0,$iv,$CI);  
         }  
  
         if(strlen($DI) > 0){  
             $DI = ICertSeed(2,0,$iv,$DI);  
         }  
  
         $tempPlusInfo = explode(",", $plusInfo);  
         $resultData["data"]["returnPageCode"] = $pageCode = isset($tempPlusInfo[0]) ? strtoupper($tempPlusInfo[0]) : "";  
         $resultData["data"]["returnPageSubCode"] = $pageSubCode = isset($tempPlusInfo[1]) ? strtoupper($tempPlusInfo[1]) : "";  
         $resultData["data"]["pageTimeCode"] = $pageTimeCode = isset($tempPlusInfo[2]) ? $tempPlusInfo[2] : "";  
         $resultData["data"]["userMobileNumber"] = Fn_mobileNumberFormat($phoneNo);  
         $resultData["data"]["userName"] = $name;  
  
  
         //  14세 미만 사용 차단  
         $userAge = Fn_UserAgeChecking(Fn_setDataEncryption($birthDay));     //  연령 확인  
  
         if (strtoupper($pageCode) == strtoupper("ONE_JOIN")){  
             if ((int)$userAge >= 19){  
                 if ($pageCode == strtoupper("ONE_JOIN")){  
                     $_SESSION['age_verified'] = true;  
                     $resultData["data"]["result"] = TRUE;  
                 }  
             } else {  
                 //  14세 미만 제한.  
                 $resultData["data"]["message"] = "만 19세 미만은 이용하실수 없습니다.";  
             }  
         }  
  
  
     } else {  
         $resultData["data"]["message"] = "본인인증 진행중 오류가 발생하였습니다.\n관리자에게 문의하시기 바랍니다.[IV00]";  
     }  
  
     $this->layout->page(Fn_Device_View(), 'loadingView', "New/result", $resultDB, $resultData, $this->itDia_session);  
 }

```