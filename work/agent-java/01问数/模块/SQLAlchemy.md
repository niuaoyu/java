
SQLAlchemy是Python的SQL工具包和对象关系映射(ORM)框架，ORM全称为Object Relational Mapping(对象关系映射)

作用：将数据库表及字段与Python实体类及类属性进行映射，通过面向实体类进行增删改查操作、提供企业级持久化模式、支持高效数据库访问

步骤：
1. 建立实体类class
2. 创立engine引擎（管理数据库的连接，把数据库的连接传过来）：传统方式问题，每次操作前创建连接，操作后立即销毁，频繁创建销毁导致性能低下。连接池优势，预先创建多个连接存储在"池"中，操作时从池中获取可用连接，使用完毕后归还而非销毁，避免了重复创建连接的开销，显著提高数据库操作效率

详细步骤：先在yaml配置文件里，把所有的数据库的ip，password，name，都配置好。然后开始创建一个create_async_engine（），通过omegaconf导入这些配置到create_async_engine()这个函数，然后通过这个en'gi'nie获取数据库的连接，然后通过session处理所有的crud！












