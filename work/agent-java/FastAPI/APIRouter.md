
**APIRouter 就像一个"路由模块"，用来组织和分组 API 路由。**

```python
# 没有 APIRouter（所有路由都堆在主文件）
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/user/register")  # 用户相关
async def register():
    pass

@app.get("/api/user/login")     # 用户相关
async def login():
    pass

@app.get("/api/product/list")   # 商品相关
async def list_products():
    pass

@app.get("/api/product/detail") # 商品相关
async def product_detail():
    pass
# ❌ 路由越来越多，文件越来越乱！
```

```python
# 有 APIRouter（按功能分组）
# user.py - 用户模块
from fastapi import APIRouter

router = APIRouter(prefix="/api/user", tags=["user"])

@router.post("/register")
async def register():
    pass

@router.post("/login")
async def login():
    pass

# product.py - 商品模块
from fastapi import APIRouter

router = APIRouter(prefix="/api/product", tags=["product"])

@router.get("/list")
async def list_products():
    pass

@router.get("/detail")
async def product_detail():
    pass

# main.py - 主文件
from fastapi import FastAPI
from user import router as user_router
from product import router as product_router

app = FastAPI()
app.include_router(user_router)    # 注册用户路由
app.include_router(product_router) # 注册商品路由
# ✅ 清晰！模块化！
```

## APIRouter 的参数详解

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/api/user",  # 所有路由的前缀
    tags=["user"],       # 文档分组标签
    responses={404: {"description": "Not found"}},  # 统一响应
    dependencies=[Depends(verify_token)]  # 统一依赖
)

@router.post("/register")
async def register_user(user_data: UserRegisterRequest):
    # 实际路径：/api/user/register
    pass

@router.get("/profile")
async def get_profile():
    # 实际路径：/api/user/profile
    pass

@router.put("/update")
async def update_user():
    # 实际路径：/api/user/update
    pass
```

### 参数的详细说明

| 参数 | 作用 | 示例 |
|------|------|------|
| `prefix` | 路由前缀，所有路由自动加上 | `/api/user` |
| `tags` | 在 API 文档中分组显示 | `["user"]` |
| `responses` | 统一响应状态码描述 | `{404: {...}}` |
| `dependencies` | 所有路由共享的依赖 | `[Depends(auth)]` |

## 完整的注册流程

### 步骤 1：创建路由文件

```python
# user.py - 用户模块
from fastapi import APIRouter, Depends
from pydantic import BaseModel

# 创建路由器
router = APIRouter(
    prefix="/api/user",
    tags=["用户管理"]
)

# 定义请求模型
class UserRegisterRequest(BaseModel):
    username: str
    password: str
    email: str

class UserLoginRequest(BaseModel):
    username: str
    password: str

# 定义路由
@router.post("/register")
async def register_user(user_data: UserRegisterRequest):
    """用户注册"""
    return {
        "message": "注册成功",
        "username": user_data.username,
        "email": user_data.email
    }

@router.post("/login")
async def login_user(user_data: UserLoginRequest):
    """用户登录"""
    return {
        "message": "登录成功",
        "username": user_data.username
    }

@router.get("/profile/{user_id}")
async def get_user_profile(user_id: int):
    """获取用户资料"""
    return {
        "user_id": user_id,
        "name": "Alice",
        "age": 25
    }
```

### 步骤 2：创建主文件并注册路由

```python
# main.py - 主应用
from fastapi import FastAPI
from user import router as user_router  # 导入路由器

app = FastAPI(title="我的API")

# 注册路由器
app.include_router(user_router)

# 也可以添加根路由
@app.get("/")
async def root():
    return {"message": "Hello World"}

# 启动应用：uvicorn main:app --reload
```

### 步骤 3：路由映射关系

```python
# 路由器中的路由
@router.post("/register")     # + prefix="/api/user"
# 实际 URL：/api/user/register

@router.post("/login")        # + prefix="/api/user"
# 实际 URL：/api/user/login

@router.get("/profile/{user_id}")  # + prefix="/api/user"
# 实际 URL：/api/user/profile/123
```

## APIRouter 的高级用法

### 1. 路由依赖（统一鉴权）

```python
from fastapi import APIRouter, Depends, Header, HTTPException

async def verify_token(authorization: str = Header(...)):
    if authorization != "Bearer secret_token":
        raise HTTPException(401, "Invalid token")
    return {"user_id": 1}

# 所有路由都需要验证 token
router = APIRouter(
    prefix="/api/user",
    tags=["user"],
    dependencies=[Depends(verify_token)]  # 统一依赖
)

@router.get("/profile")
async def get_profile():
    # 不需要再手动验证 token
    return {"name": "Alice"}
```

### 2. 嵌套路由器

```python
# admin.py
from fastapi import APIRouter

admin_router = APIRouter(prefix="/admin", tags=["admin"])

@admin_router.get("/users")
async def admin_users():
    return {"admin": "users"}

# user.py
from fastapi import APIRouter
from admin import admin_router

router = APIRouter(prefix="/api/user", tags=["user"])

# 嵌套路由
router.include_router(admin_router)  # 实际路径：/api/user/admin/users

@router.get("/profile")
async def profile():
    return {"name": "Alice"}
```

### 3. 自定义响应

```python
from fastapi import APIRouter
from fastapi.responses import JSONResponse

router = APIRouter(prefix="/api/user")

@router.post("/register", 
             status_code=201,  # 201 Created
             response_model=UserResponse)  # 响应模型
async def register():
    return {"id": 1, "name": "Alice"}

@router.get("/{user_id}", 
            responses={
                200: {"description": "成功"},
                404: {"description": "用户不存在"}
            })
async def get_user(user_id: int):
    if user_id > 100:
        return JSONResponse(404, {"message": "User not found"})
    return {"id": user_id, "name": "Alice"}
```

## 七、对比总结

| 特性 | 直接在 app 上装饰 | 使用 APIRouter |
|------|------------------|----------------|
| 组织方式 | 所有路由都在主文件 | 按功能模块分文件 |
| 路径前缀 | 手动写完整路径 | 使用 prefix 自动添加 |
| 文档分组 | 需要手动设置 tags | 统一设置 tags |
| 复用性 | 差，无法移植 | 好，可以导入使用 |
| 依赖管理 | 每个路由单独配置 | 可以统一配置 |
| 适合场景 | 小型项目 | 中大型项目 |

## 八、完整可用示例

```python
# 文件结构
project/
├── main.py
├── routers/
│   ├── __init__.py
│   ├── user.py
│   └── product.py
└── models/
    └── __init__.py

# routers/user.py
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/api/user", tags=["user"])

class UserRegisterRequest(BaseModel):
    username: str
    password: str

@router.post("/register")
async def register_user(user_data: UserRegisterRequest):
    return {"message": f"注册成功: {user_data.username}"}

@router.get("/{user_id}")
async def get_user(user_id: int):
    return {"id": user_id, "name": "Alice"}

# main.py
from fastapi import FastAPI
from routers.user import router as user_router

app = FastAPI(title="API 示例")
app.include_router(user_router)

# 访问 http://localhost:8000/docs 查看 API 文档
# POST http://localhost:8000/api/user/register
# GET  http://localhost:8000/api/user/1
```

## 核心总结

1. **APIRouter = 路由分组工具**，让代码更有组织性
2. **prefix = 自动添加路径前缀**，不用重复写 `/api/user`
3. **tags = API 文档分组**，在 Swagger 中清晰显示
4. **注册方式 = `app.include_router(router)`**，将路由挂载到主应用
5. **你的理解完全正确**：前端访问 `/api/user/register` 时，就会执行 `register_user` 函数！

现在明白了吗？APIRouter 就是 FastAPI 的"模块化"武器！🎯