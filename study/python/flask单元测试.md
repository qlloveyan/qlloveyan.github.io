[TOC]


### 1. python单元测试框架
####  1.1 unittest
- python自带测试框架
- 面向对象的方式实现测试流程

#### 1.2 nose2
- 扩展unittest库

#### 1.3 pytest
- 第三方框架，测试丰富程度和易用程度优于unittest
- 示例  
```python
def inc(x):
    return x +1
 
def test_answer():
    assert inc(3) == 5
```

### 2. unittest使用
- 示例  
```python
class SimpleTest(unittest.TestCase):
	
	def setUp(self):
        print("do before test")

    def tearDown(self):
        print("do after test")
    
    def inc(self, x):
    	return x +1
    
    def test_answer(self):
    	self.assertEqual(inc(3), 5)
    
if __name__ == "__main__":
    unittest.main()
```

### 3. unittest + 态势
#### 3.1 公共方法使用测试
- 需求：在国产化改造中，对达梦python兼容使用做全局测试
- 示例：  
```python
class DMUtilTest(unittest.TestCase):

    def setUp(self):
        self.engine = create_engine(PG_DB_URI, echo=True, encoding="utf-8", max_overflow=30, pool_size=10, connect_args={
            'connection_timeout': 60, 'autoCommit': True
        })
        DBSession = sessionmaker(bind=self.engine)
        self.db_session = DBSession()

    def tearDown(self):
        self.db_session.commit()
        self.db_session.close()

    def test_init_db(self):
        Base.metadata.create_all(self.engine)

    def test_add_data(self):
        user1 = Test(name="ww", domain="www.test.com", gender=0, extend_info={"age": 27}, label_ids=[1, 2])
        user2 = Test(name="ls", gender=0, extend_info={"age": 28, "child": {"age": 1, "gender": 1, "name": "ls2"}}, label_ids=[3, 2])
        user3 = Test(name="zs", gender=1)
        user1 = self.db_session.merge(user1)
        user2 = self.db_session.merge(user2)
        user3 = self.db_session.merge(user3)
        self.db_session.flush()
        assert user1.id > 0

if __name__ == "__main__":
    # 1. 不指定单独的测试方法, 直接全量执行
    # unittest.main()
	
	# 2. 只运行指定的测试方法
    suite = unittest.TestSuite()
    suite.addTest(DMUtilTest('test_init_db'))
    suite.addTest(DMUtilTest('test_add_data'))
    unittest.TextTestRunner().run(suite)
```

#### 3.2 view层测试示例
- 前置执行条件
	- login_required：进入方法前，需要在session中验证是否有username字段
	- csrf_protect：post请求中，需要将session中的csrf_token、csrf_timeout做验证
	- permission_required：需要验证用户是否有当前接口的权限
 - session操作接口文档：[测试 Flask 应用 — Flask 0.10.1 文档](http://docs.jinkan.org/docs/flask/testing.html)
- 示例  
```python
class ViewTest(unittest.TestCase):

    def setUp(self):
        self.app = init_test_app()

    def tearDown(self):
        close_session()

    def test_company_list(self):
        with self.app.test_client() as c:
            with c.session_transaction() as sess:
                sess['username'] = 'admin'
                sess['uid'] = 1
                sess['csrf_token'] = CSRF_TOKEN
                sess['csrf_timeout'] = generator_timeout()
            resp = c.get("/situation/assets_manager/service/company_list")
        self.assertEqual(resp.status_code, 200)
        data = json.loads(resp.data)
        logging.error(data)
        self.assertEqual(data.get('code'), 200)

if __name__ == "__main__":
    suite = unittest.TestSuite()
    suite.addTest(ViewTest('test_company_list'))
    unittest.TextTestRunner().run(suite)
```
- 公共方法层面  
```python
CSRF_TOKEN = "589c7ed15a4d63745501f27bc10723e8"

def init_test_app():
    app = create_app(name="testing")
    app.testing = True

    ctx = app.app_context()
    ctx.push()

    db = DbClass(app.config.get("PG_DB_URI"))
    db_session = db.make_session()
    g.pg_db = db_session
    return app

def close_session():
    db_session = g.pg_db
    db_session.commit()
    db_session.close()

def generator_timeout():
    current_timestamp = int(time.time())
    current_timestamp = current_timestamp + 3500
    return time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(current_timestamp))
```