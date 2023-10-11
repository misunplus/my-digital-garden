---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///v2/admin/"}
---

# v2/admin/grade

#### 쿼리 유효성 검사
server/middleware/verifyUserIdentity.js
```
  
const validateGradeRequestInput = (req, res, next) => {  
  const data = req.query;  
  
  if (!req.query.page) {  
    req.query.page = "1";  
  }  
  const page = parseInt(req.query.page);  
  if (isNaN(page) || page <= 0) {  
  
    return res.status(403).render('admin/error',  
        { error: fieldErrors.errors([{ msg: constant.grade.INVALID_PAGE_NUM }], true) }  
    );  
  }  
  
  const validStatuses = ["pending", "approved", "rejected"];  
  if (data.searchType === 'status' && data.status && !validStatuses.includes(data.status)) {  
    return res.status(403).render('admin/error',  
        { error: fieldErrors.errors([{ msg: constant.grade.INVALID_STATUS_VALUE }], true) }  
    );  
  }  
  
  const maxLengths = {  
    email: 100,  
    username: 50  
  };  
  
  if (data.email && data.email.length > maxLengths.email) {  
    // return res.status(400).send({ error: fieldErrors.errors([{ msg: constant.grade.EMAIL_LONG }], true) }).end();  
    return res.status(400).render('admin/error',  
        { error: fieldErrors.errors([{ msg: constant.grade.EMAIL_LONG }], true) }  
    );  
  }  
  
  if (data.username && data.username.length > maxLengths.username) {  
    return res.status(400).render('admin/error',  
        { error: fieldErrors.errors([{ msg: constant.grade.USER_NAME_LONG }], true) }  
    );  
  }  
  
  if (req.query.email && typeof req.query.email !== 'string') {  
    return res.status(400).render('admin/error',  
        { error: fieldErrors.errors([{ msg: constant.grade.INVALID_EMAIL_FORMAT }], true) }  
    );  
  }  
  
  next();  
}

```

##### 에러 렌더링 페이지 생성
include시 필수값 title, nav 
```

<%- include('./includes/header.ejs', { title: 'Error Page' }) %>  
<%- include('./includes/navigation.ejs', { nav: '' }) %>  
  
<div class="content-wrapper">  
    <section class="content-header">  
        <h1>  
            Error Page  
        </h1>  
  
    </section>  
  
    <!-- Main content -->  
    <section class="content">  
        <div class="error-page">  
            <div class="error-content">  
                <h3><i class="fa fa-warning text-yellow"></i>  
                    <% error.forEach(err => { %>  
                        <%= err.message %>  
                    <% }); %>  
                </h3>  
  
                <p>                    에러가 발생하였습니다.  
                    <a href="<%= process.env.ADMIN_SLUG %>">대시보드로 돌아가기</a>  
                </p>  
            </div>  
            <!-- /.error-content -->  
        </div>  
        <!-- /.error-page -->  
    </section>  
    <!-- /.content -->  
</div>  
<%- include('./includes/scripts.ejs') %>  
<%- include('./includes/footer.ejs') %>
```



router.get('/requests', validateGradeRequestInput, isAdmin, controller.index);
[[Obsidian Vault/Notion/소프트 랩스/영상광고/v2/admin/requests\|requests]]