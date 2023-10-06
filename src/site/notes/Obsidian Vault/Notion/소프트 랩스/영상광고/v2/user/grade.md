---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///v2/user/grade/"}
---


### 아래 링크 접속시  등급메뉴 
[http://localhost:3011/video-admin/levels](http://localhost:3011/video-admin/levels)


### 기본 등급
	기본 4개의 등급
	어드민, 운영자, 디폴트, 퍼블릭
	회원가입시 기본 →디폴트(level_id = 4)
	운영자(level_id = 3)사용자 측 콘텐츠를 편집할 수 있습니다.
	퍼블릭(level_id =2 ) 비로그인자

### 관련 테이블
	user table 
	 level_id


  

#### 테이블생성 

```
CREATE TABLE user_grade_requests (     
id INT AUTO_INCREMENT PRIMARY KEY COMMENT '요청 ID',     
user_id INT UNSIGNED NOT NULL COMMENT '유저 ID (users 테이블의 user_id와 연관)',     
requested_grade VARCHAR(255) NOT NULL COMMENT '요청한 등급',     
request_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '요청 날짜 및 시간',     
status ENUM('PENDING', 'APPROVED', 'REJECTED') DEFAULT 'PENDING' 
COMMENT '요청 상태',    
admin_comment TEXT COMMENT '관리자의 코멘트',     
FOREIGN KEY (user_id) REFERENCES users(user_id)) 
ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci 
COMMENT '사용자의 등급 요청을 기록하는 테이블';
```

  
