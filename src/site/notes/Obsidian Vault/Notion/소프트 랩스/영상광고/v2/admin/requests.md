---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///v2/admin/requests/"}
---


###  관리자 등급신청 메뉴 
페이징, 검색 컨트롤러 연결
```
exports.index = async function(req, res) {  
  /**  
   #swagger.tags = ['v2/admin']   
   #swagger.description = "관리자 페이지 권한요청 페이지"  
   #swagger.security = [{'sessionAuth': []}]   
   #swagger.parameters['obj'] = {  
		in: 'query',        
		description: `            
			searchType: \n            
			- type: string \n            
			- description: email,level_title,username,status,date            
			- example: email \n            
			- \n        
		`,        
		required: true,        
		schema: {  
		  searchType:emai          
		  email: ""          
		  username: ""          
		  level_title: ""          
		  status: ""          
		  start_date: ""          
		  end_date: ""        
	  }  
	}   
   */  
  
  
  const url = req.originalUrl.replace(process.env.ADMIN_SLUG, '');  
  
  try {  
    const result = await adminModel.findAll(req, req.query);  
    const gradeRequests = result.results;  
    const totalCount = result.totalCount;  
    const currentPage = parseInt(req.query.page) || 1;  
  
   console.log(gradeRequests);  
    res.render('admin/grades/index', {  
      nav: url,  
      title: "등급 요청 관리",  
      gradeRequests: gradeRequests,  
      query: req.query,  
      itemsPerPage: 10,  
      currentPage: currentPage,  
      totalCount: totalCount  
    });  
  
  
  } catch (error) {  
    return res.status(500).send({ error: fieldErrors.errors([{ msg: constant.grade.DB_ERROR }], true) }).end();  
  }  
  
};


```

랜더링 admin/grades/index.ejs 파일 생성

```


```

