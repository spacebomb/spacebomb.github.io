#禧云设备管理平台

基于Spring Boot的太空桥后台服务接口工程开发说明v.1.0
===============================

* 我们使用当前最新稳定版[Spring Boot 1.5.4.RELEASE](http://projects.spring.io/spring-boot/)来开发太空桥后端服务接口

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
* 服务程序中用到的和您想要用的资源示例都可在[https://spring.io/guides](https://spring.io/guides)中找到，请自行熟悉
![](media/15004768104904/15004775371747.jpg)

* 学习Spring Boot的第一个开始工程配置地址：[https://start.spring.io/](https://start.spring.io/)

太空桥服务端目录结构
-------------------
基于Maven 3+ 配置生成的多模块工程

```
spacebradge                         工程主目录
    spacebradge-dao/                数据层：用于进行数据库操作的模块
      - java/                       源文件存放目录
        - config/                   数据库规则配置
        - entity/                   数据库映射模型实体
        - mapper/                   基于mybatis的数据库操作接口类
      - resource
        - mybatis/             
          - mapper/                 mybatis数据表映射配置文件
          - mybatis-config.xml      mybatis数据库操作的通用配置  
          
    spacebradge-model/              数据模型：用户自定义的业务数据Bean
      - java/                       源文件存放目录
      
    spacebradge-service/            业务处理模块：业务逻辑封装
      - java/                       源文件存放目录
      
    spacebradge-util/               通用工具
      - java/                       源文件存放目录
    spacebradge-web/                用于对外暴露的可发布模块，如：API、页面
      - java
        - controller                对外暴露的可访问控制器
      - resource/                   资源目录
        - application.yml           Spring boot 特有的配置文件
        - logback.xml               日志规则
```
工程代码示例
-------------------
* 日志记录方式

```java
import lombok.extern.slf4j.Slf4j;
    
/**
* 类注释需要写清楚
*/
@Slf4j
public class Test {
   /**
    * 函数说明
    */
   public void egLogger(){
       log.info("日志输出内容");
   }
}
```
* 事务操作

注意函数上的`@Transactional`注解，其使用方式自行关注

```java
/**
* 事务管理
*/
@Transactional
public void transTest() {
   log.info("------Begin a transaction execute.----------------");
   TestEntity entity1 = new TestEntity();
   entity1.setName("test");
   testMapper.insertRow(entity1);
   supplierMapper.findAll();
   TestEntity entity2 = new TestEntity();
   entity2.setName("test-test-test");
   testMapper.insertRow(entity2);
}
```
* 数据库操作（目前提供mybatis、JdbcTemplate两种方式）
    * Mybatis操作

    1、在Dao层`com/xiyunerp/mapper`中新建一个操作接口类，用于与对应的XML映射
    
    ```
    /**
     * 类说明
     */
    @Component(value = "supplierMapper")    //为当前操作类取一个别名，可以避免多个相同类名在Spring容器实例化时冲突
    public interface SupplierMapper {
        List<SupplierEntity> findAll();
    }
    ```
    2、在Dao层资源文件夹`resources/mabatis/mapper`中新建一个mybatis对应的数据表操作XML
    
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xiyunerp.mapper.SupplierMapper">
    <resultMap id="BaseResultMap" type="com.xiyunerp.entity.SupplierEntity">
        <id column="supplier_id" property="supplierId"/>
        <result column="supplier_name" property="supplierName"/>
        <result column="level_1" property="level1"/>
        <result column="level_2" property="level2"/>
        <result column="level_3" property="level3"/>
        <result column="level_4" property="level4"/>
        <result column="level_5" property="level5"/>
        <result column="level_6" property="level6"/>
        <result column="level_7" property="level7"/>
        <result column="level_8" property="level8"/>
        <result column="manage_eara_id" property="manageEaraId"/>
        <result column="create_time" property="createTime"/>
        <result column="update_time" property="updateTime"/>
    </resultMap>


    <select id="findAll" resultMap="BaseResultMap">
        SELECT * FROM qy_supplier
    </select>
</mapper>
    ```
    3、使用实例
    
    ```java
    @Autowired
    private SupplierMapper supplierMapper;

    /**
     * mybatis操作数据方式
     */
    public void findTest() {
        System.out.println(supplierMapper.findAll());
    }

    ```
    
    
    * JdbcTemplate操作

    ```java
    /**
     * 使用从库的DataSource实例,有两种：1、primaryJdbcTemplate 2、secondaryJdbcTemplate，
     * 详情关注Dao层的数据源配置文件com.xiyunerp.config.DataSourceConfig
     */
    @Autowired
    @Qualifier("secondaryJdbcTemplate")
    private JdbcTemplate secondaryJdbcTemplate;


    /**
     * JdbcTemplate操作数据方式
     */
    public void transTemplate() {
        primaryJdbcTemplate.update("insert into qy_test(name) values(?)", "liukui-a");
    }

    ```
* 分页操作

```
/**
* 分页获取数据方式
*/
public void beginPageData() {
   int currentPage = 1, pageSize = 3;
   boolean needCountNum = true;

   //一定要，只需要在数据库查询之前加上这一句即可实现分页效果
   PageHelper.startPage(currentPage, pageSize, needCountNum);
   List<SupplierEntity> supplierEntityList = supplierMapper.findAll();
   System.out.println("当前获取数据条数：" + supplierEntityList.size());

   //如果要获取分页先关的统计数据，如：总行数、总页数等等，只需要强转为Page对象即可
   Page page = (Page) supplierEntityList;
   System.out.println("当前所有数据条数：" + page.getTotal());
}
```
项目构建方式
-------------------
* 微服务 .jar 发布方式

``` js
> maven clean package -Pdev     #开发环境打包
> maven clean package -Ptest    #测试环境打包
> maven clean package -Pprod    #生产环境打包
```
* 基于容器发布方式 .war



