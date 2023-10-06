---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///mobile/","tags":["gardenEntry"]}
---



# 이슈
	new -> m -> 본인인증 -> m -> new 이동시
	m-> new로 응답을 보낼수 없음 

#### 시도
	 1. javascript postMessage 시도 opner 값(부모창 값을 잃어버림)
	 2. 세션 공유
```
config.php
서브도메인에서는 세션 공유를 위한 설정

$config['cookie_domain']    = '.itdia.co.kr';
	
```

```
index.php
여기서는 코드이그나이터의 세션을 사용할 수 없음
기본 설정된값과 같게 설정

ini_set('session.cookie_domain', '.itdia.co.kr');  
session_name('itDia');  
session_save_path('/var/lib/php/session');  
session_start();

```

문제 m사이트에서는 세션 라이브러리를 사용해서 세션을 관리
쿠키에 저장된 아이디값으로는 접근이 불가함.
세션 파일에서 쿠키 아이디값을 가지고 있는 세션파일을 파일내용중 
본인인증 결과 가져와 새로운 세션에 저장

