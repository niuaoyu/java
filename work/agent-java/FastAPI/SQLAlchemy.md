pip install "sqlalchemy[asyncio]" aiomysql

```python
from sqlalchemy.ext.asyncio import async_sessionmaker, AsyncSession, creat e_async_engine

# 数据库 URL 
ASYNC_DATABASE_URL = "mysql+aiomysql://root:123456@localhost:3306/news_ap p?charset=utf8mb4" 

创建异步引擎async_engine = create_async_engine( 
	ASYNC_DATABASE_URL, echo=True, # 可选：输出 SQL ⽇志 
	pool_size=10, # 设置连接池中保持的持久连接数 
	max_overflow=20 # 设置连接池允许创建的额外连接数
)

# 创建异步会话⼯⼚
AsyncSessionLocal = async_sessionmaker( 
	bind=async_engine, 
	class_=AsyncSession, 
	expire_on_commit=False 
)
 # 依赖项，⽤于获取数据库会话
async def get_db(): 
	async with AsyncSessionLocal() as session: 
		try: 
			yield session 
			await session.commit() 
		except Exception: 
			await session.rollback() 
			raise 
		finally: 
			await session.close()

class News(Base): 
	__tablename__ = "news" # 创建索引：提升查询速度 
	__table_args__ = ( Index('fk_news_category_idx', 'category_id'), Index('idx_publish_time', 'publish_time') )

```