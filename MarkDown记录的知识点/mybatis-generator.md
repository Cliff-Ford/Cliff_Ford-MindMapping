### 一、 MyBatis-Generator技术产生背景

​		随着mybatis的推广与使用，针对单个表的crud操作大多是重复的且没有新意的，为了提高编码，Mybatis-Generator这门技术应运而生，通过配置generatorConfig.xml文件和编写Generator类即可针对数据库快速生成所有表的crud操作代码，并会生成一个xxxExample类，该类可用于面向对象形式的查询。

### 二、环境搭建

* 建立一个maven工程，pom.xml文件添加generator的依赖

```xml
<!-- MyBatis 生成器 -->
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.3</version>
</dependency>
```

* 在resources文件夹下面建立一个generator.properties文件，该文件用于存放连接数据库的四大属性

```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
jdbc.userId=root
jdbc.password=1234
```

* 配置generatorConfig.xml文件

```xml
# 一些配置检验项
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
# 根标签
<generatorConfiguration>
    # 引入上一步定义的数据库资源属性文件
    <properties resource="generator.properties"/>
    # 可以配置多个数据源
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        # 设置开始分隔符，用于数据库名，表名，列名前的前缀
        <property name="beginningDelimiter" value="`"/>
        # 设置结束分隔符
        <property name="endingDelimiter" value="`"/>
        # 设置生成的java文件编码
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 为模型生成序列化方法-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--可以自定义生成model的代码注释-->
        <commentGenerator type="com.macro.mall.tiny.mbg.CommentGenerator">
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        <!--指定生成model的路径，该路径用于存放pojo类以及pojoExample类-->
        <javaModelGenerator targetPackage="com.macro.mall.tiny.mbg.model" targetProject="mall-tiny-01\src\main\java"/>
        <!--指定生成mapper.xml的路径，该xml包含单表的crud操作代码-->
        <sqlMapGenerator targetPackage="com.macro.mall.tiny.mbg.mapper" targetProject="mall-tiny-01\src\main\resources"/>
        <!--指定生成mapper接口的的路径-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.macro.mall.tiny.mbg.mapper"
                             targetProject="mall-tiny-01\src\main\java"/>
        <!--生成全部表tableName设为%，我的数据库只有一个user表，所以这里就只填了一个-->
        <table tableName="user">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

### 三、精简配置文件generatorConfig.xml

```xml
# 一些配置检验项
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
# 根标签
<generatorConfiguration>
    # 引入上一步定义的数据库资源属性文件
    <properties resource="generator.properties"/>
    # 可以配置多个数据源
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <!--告诉我怎么连接数据库-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        <!--告诉我生成的pojo类以及pojoExample类放在哪里-->
        <javaModelGenerator targetPackage="com.macro.mall.tiny.mbg.model" targetProject="mall-tiny-01\src\main\java"/>
        <!--告诉我生成的mapper接口放在哪里-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.macro.mall.tiny.mbg.mapper"
                             targetProject="mall-tiny-01\src\main\java"/>
        <!--告诉我生成的对应mapper的xml文件放在哪里-->
        <sqlMapGenerator targetPackage="com.macro.mall.tiny.mbg.mapper" targetProject="mall-tiny-01\src\main\resources"/>        
        <!--告诉我针对数据库的哪几张表自动生成代码-->
        <table tableName="user">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

### 四、详细的generatorConfig.xml配置

[简书链接](https://www.jianshu.com/p/e09d2370b796)

[mybatis-generator](http://www.mybatis.org/generator/)

### 五、编写生成器Generator

```java
/**
 * 用于生产MBG的代码
 * Created by macro on 2018/4/26.
 */
public class Generator {
    public static void main(String[] args) throws Exception {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}
```

该生成器运行之后，会产生如下结果

![1560604014516](C:\Users\Cliff_Ford\AppData\Roaming\Typora\typora-user-images\1560604014516.png)

产生一个pojo类User，一个UserExample，一个UserMapper接口，一个UserMapper.xml文件

### 六、看看自动生成的User类有什么

```java
public class User implements Serializable {
    private Long id;
    private String name;
    # 省略其他属性

    # 一个序列化参数
    private static final long serialVersionUID = 1L;

    # 省略一些getter、setter方法

    # 一个toString方法
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", id=").append(id);
        sb.append(", name=").append(name);
        sb.append(", serialVersionUID=").append(serialVersionUID);
        sb.append("]");
        return sb.toString();
    }
}
```

出奇的发现，生成的pojo类和我们手动编写的没有什么区别

### 七、看看自动生成的UserExample有什么

这个UserExample类的大小会根据你表字段的多少而不同

```java
public class UserExample{
    # 决定了User类根据哪一个字段排序
    protected String orderByClause;
    # 决定了查询结果是否重复
    protected boolean distinct;
	# Criteria是一组查询条件，每个Criteria都可以让User表返回一个结果集
    # oredCriteria表示一组Criteria，将每个Criteria查询条件对应的结果集取并集
    protected List<Criteria> oredCriteria;

    # 省略构造函数

    # 省略orderByClause, distinct, oredCriteria三个属性的getter，setter方法

    # oredCriteria添加一个criteria，根据不同情况，oredCriteria有不同的添加方法，但该criteria的查询结果都将作为结果集的一部分
    
    # 清楚查询条件
    public void clear() {
        oredCriteria.clear();
        orderByClause = null;
        distinct = false;
    }

    # 一个抽象的生成查询条件的类
    protected abstract static class GeneratedCriteria {
        # criteria是一组查询条件，criterion是单个查询条件，criterion是一个静态内部类，在下面
        protected List<Criterion> criteria;

        # 省略构造方法
        # 省略criteria属性的getter，setter方法
        # 省略criteria不同情况下添加criterion的方法
            
        # 举User表中的一个字段id为例
        # 判断了User表查询条件中id是否为null，=，>，>=，<，<=，in，not in，between，not between等查询条件
    }

    public static class Criteria extends GeneratedCriteria {

        protected Criteria() {
            super();
        }
    }

    # 单个查询条件，用于sql语句的拼凑
    public static class Criterion {
        # 字段+判断操作符，如 id is null
        private String condition;
		# 第一个值
        private Object value;
		# 范围取值的第二个值
        private Object secondValue;

        # null条件判断
        private boolean noValue;
		# = 条件判断
        private boolean singleValue;
		# 范围查询条件判断
        private boolean betweenValue;
		# in 查询条件判断
        private boolean listValue;
		# 类型处理器
        private String typeHandler;

        # 省略上面属性的getter，setter方法

        # 不同条件下的构造方法
}
```

### 八、看看自动生成的UserMapper接口有什么

```java
public interface UserMapper {
    /**
     * 统计指定条件下User的数目，example为空时全表统计
     * @param example 面向对象形式的查询条件
     * @return int 条目数
     */
    int countByExample(UserExample example);

    /**
     * 删除指定条件User,example为空时删除表中所有数据，保留表结构
     * @param example criteria
     * @return 影响条数
     */
    int deleteByExample(UserExample example);

    /**
     * 删除User
     * @param id 主键
     * @return 影响条数
     */
    int deleteByPrimaryKey(Long id);

    /**
     * 添加用户，记录包含null值
     * @param record user对象
     * @return 影响条数
     */
    int insert(User record);

    /**
     * 添加user,记录不包含null值
     * @param record user对象
    * @return 影响条数
     */
    int insertSelective(User record);

    /**
     * 根据查询标准查找用户，example为空时查找所有user
     * @param example 查询标准
     * @return List<User>
     */
    List<User> selectByExample(UserExample example);

    /**
     * id查找
     * @param id 主键
     * @return User
     */
    User selectByPrimaryKey(Long id);

    /**
     * 修改User
     * @param record 新值
     * @param example 查询条件
     * @return 影响条数
     */
    int updateByExampleSelective(@Param("record") User record, @Param("example") UserExample example);

    /**
     * 修改User
     * @param record 新值
     * @param example 查询条件
     * @return 影响条数
     */
    int updateByExample(@Param("record") User record, @Param("example") UserExample example);

    /**
     * 修改User
     * @param record 带id
     * @return 影响条数
     */
    int updateByPrimaryKeySelective(User record);

    /**
     * 修改User
     * @param record 带主键的user对象
     * @return 影响条数
     */
    int updateByPrimaryKey(User record);
}
```

该Mapper接口是单表的crud操作，注意带Selective的是做了null值排除，没带Selective的则没有做null值排除

### 九、看看自动生成的UserMapper.xml文件有什么

```xml
<!--省略配置头信息-->
<mapper namespace="com.macro.mall.tiny.mbg.mapper.UserMapper">
  # 指定了一个结果映射，该映射包含User表的全部字段
  <resultMap id="BaseResultMap" type="com.macro.mall.tiny.mbg.model.User">
    <id column="id" jdbcType="BIGINT" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
  </resultMap>
  # 指定了一个动态sql片段，最终拼接成这样的sql语句，where (clause1 and clasue2) or (clause3 or clause4)
  <sql id="Example_Where_Clause">
    <where>
      # 利用了oredCriteria，每一个criteria之间用or将结果并起来
      <foreach collection="oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          # criteria内部是一组criterion，每个criterion是一个查询条件，需要用and拼接起来，组成一个完成的where查询条件
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              # choose选择下面四种查询操作符的一种
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
  # 和查询类似，但这里是set
  <sql id="Update_By_Example_Where_Clause">
    
  </sql>
  # User表全部的字段
  <sql id="Base_Column_List">
    id, name
  </sql>
  
  <select id="selectByExample" parameterType="com.macro.mall.tiny.mbg.model.UserExample" resultMap="BaseResultMap">
    select
    <if test="distinct">
      distinct
    </if>
    <include refid="Base_Column_List" />
    from user
    # 注意这里的where条件拼接方式，说明selectByExample这类方法，可以加参数，可以不加参数
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null">
      order by ${orderByClause}
    </if>
  </select>
  # 省略其他的方法
</mapper>
```













