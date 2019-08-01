1.首先创建一个Maven工程,pom依赖中添加
```
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.44</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.5</version>
        </dependency>

    </dependencies>
```

2.直接在项目目录下新建配置文件
![](https://upload-images.jianshu.io/upload_images/5660329-b36fe5a1a9f842b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置文件的内容为:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="testTables" targetRuntime="MyBatis3">

        <!-- 配置pojo的序列化 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql:///pinyougoudb" userId="root"
                        password="2237">
        </jdbcConnection>
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
            NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- targetProject:生成PO类的位置 -->
        <javaModelGenerator targetPackage="com.logoxiang.pojo"
                            targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
        <sqlMapGenerator targetPackage="com.logoxiang.mapper"
                         targetProject=".\src\main\resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.logoxiang.mapper"
                             targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
        <!-- 指定数据库表 -->
        <table schema="" tableName="tb_address"/>
        <table schema="" tableName="tb_areas"/>
        <table schema="" tableName="tb_brand"/>
        <table schema="" tableName="tb_cities"/>
        <table schema="" tableName="tb_content"/>
        <table schema="" tableName="tb_content_category"/>
        <table schema="" tableName="tb_freight_template"/>
        <table schema="" tableName="tb_goods"/>
        <table schema="" tableName="tb_goods_desc"/>
        <table schema="" tableName="tb_item"/>
        <table schema="" tableName="tb_item_cat"/>
        <table schema="" tableName="tb_order"/>
        <table schema="" tableName="tb_order_item"/>
        <table schema="" tableName="tb_pay_log"/>
        <table schema="" tableName="tb_provinces"/>
        <table schema="" tableName="tb_seckill_goods"/>
        <table schema="" tableName="tb_seckill_order"/>
        <table schema="" tableName="tb_seller"/>
        <table schema="" tableName="tb_specification"/>
        <table schema="" tableName="tb_specification_option"/>
        <table schema="" tableName="tb_type_template"/>
        <table schema="" tableName="tb_user"/>
     </context>
</generatorConfiguration>
```

3.创建运行类:33
```
/**
 * @Author: logoxiang
 * @Date: 2019/1/26 13:52
 */
public class GeneratorSqlMap {

    public void generator() throws Exception{

        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        //指定 逆向工程配置文件
        File configFile = new File("generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                callback, warnings);
        myBatisGenerator.generate(null);

    }
    public static void main(String[] args) throws Exception {
        try {
            GeneratorSqlMap generatorSqlmap = new GeneratorSqlMap();
            generatorSqlmap.generator();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}

99
4.注意我们的逆向工程是依赖数据库中的表,所以咱们首先要把表建起来,在运行逆向工程//
