---
{"dg-publish":true,"permalink":"/obsidian-vault/notion///v2/admin/requests/"}
---


###  관리자 등급신청 메뉴 
 ejs 리턴(등급 신청관련 관리자 패널 )
유효성검사, 어드민 검사 
페이징 처리와 검색 

#### controller
```
findAll 호출


```


#### models
```
	findAll: function (req, data) {
		return new Promise((resolve, reject) => {
			const condition = [];
			let whereClause = " WHERE 1 = 1 ";

			if (data.searchType === 'email' && data.email) {
				whereClause += " AND users.email LIKE ? ";
				condition.push("%" + data.email + "%");
			}
			if (data.searchType === 'username' && data.username) {
				whereClause += " AND userdetails.username LIKE ? ";
				condition.push("%" + data.username + "%");
			}
			if (data.searchType === 'status' && data.status) {
				whereClause += " AND user_grade_requests.status = ? ";
				condition.push(data.status);
			}
			if (data.start_date) {
				whereClause += " AND user_grade_requests.request_date >= ? ";
				condition.push(data.start_date + " 00:00:00");
			}
			if (data.end_date) {
				whereClause += " AND user_grade_requests.request_date <= ? ";
				condition.push(data.end_date + " 23:59:59");
			}

			const countSql = `
          SELECT COUNT(*) as totalCount
          FROM user_grade_requests
                   JOIN users ON user_grade_requests.user_id = users.user_id
                   JOIN userdetails
                        ON user_grade_requests.user_id = userdetails.user_id
                   JOIN levels
                        ON user_grade_requests.requested_grade = levels.level_id
              ${whereClause}
			`;

			const currentPage = parseInt(data.page) || 1;
			const itemsPerPage = 10;
			const offset = (currentPage - 1) * itemsPerPage;

			const sql = `
          SELECT user_grade_requests.*,
                 users.email,
                 userdetails.username,
                 levels.title AS level_title
          FROM user_grade_requests
                   JOIN users ON user_grade_requests.user_id = users.user_id
                   JOIN userdetails
                        ON user_grade_requests.user_id = userdetails.user_id
                   JOIN levels
                        ON user_grade_requests.requested_grade = levels.level_id
              ${whereClause}
          ORDER BY request_date DESC
              LIMIT ?
          OFFSET ?
			`;

			req.getConnection((err, connection) => {
				if (err) {
					reject(err);
				}

				connection.query(countSql, condition, (err, countResults) => {
					if (err) {
						reject(err);
					}

					let totalCount = countResults[0].totalCount;

					connection.query(sql, [
						...condition, itemsPerPage, offset], (err, results) => {
						if (err) {
							reject(err);
						}
						resolve({
							results   : results,
							totalCount: totalCount
						});
					});
				});
			});

		});
	},

```