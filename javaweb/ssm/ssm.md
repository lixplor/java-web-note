# SSM整合

## Spring整合MyBatis

* 整合要求
    - SqlSessionFactory对象放到Spring容器中作为单例存在
    - 传统DAO方式中, 应从Spring容器中获取SqlSession对象
    - Mapper代理形式中, 应从Spring容器中获取Mapper对象
    - 数据库连接池和配置都应交给Spring容器处理
