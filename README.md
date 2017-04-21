# 这是 MyBatis Generator 插件的拓展插件包
应该说使用Mybatis就一定离不开[MyBatis Generator](https://github.com/mybatis/generator)这款代码生成插件，而这款插件自身还提供了插件拓展功能用于强化插件本身，官方已经提供了一些[拓展插件](http://www.mybatis.org/generator/reference/plugins.html)，本项目的目的也是通过该插件机制来强化Mybatis Generator本身，方便和减少我们平时的代码开发量。  
>因为插件是本人兴之所至所临时发布的项目（本人已近三年未做JAVA开发，代码水平请大家见谅），但基本插件都是在实际项目中经过检验的请大家放心使用，但因为项目目前主要数据库为MySQL，Mybatis实现使用Mapper.xml方式，所以代码生成时对于其他数据库和注解方式的支持未予考虑，请大家见谅。  
  
---------------------------------------
插件列表：  
* 查询单条数据插件（SelectOneByExamplePlugin）
* MySQL分页插件（LimitPlugin）
* 数据Model链式构建插件（ModelBuilderPlugin）
* Example Criteria 增强插件（ExampleEnhancedPlugin（CriteriaBuilderPlugin这个是老版本叫法，已经弃用））
* Example 目标包修改插件（ExampleTargetPlugin）
* 批量插入插件（BatchInsertPlugin）
* 逻辑删除插件（LogicalDeletePlugin）
* 数据Model属性对应Column获取插件（ModelColumnPlugin）
* 存在即更新插件（UpsertPlugin） 
* Selective选择插入更新增强插件（SelectiveEnhancedPlugin（1.0.7-SNAPSHOT）） 
  
---------------------------------------
Maven引用：  
```xml
<dependency>
  <groupId>com.itfsw</groupId>
  <artifactId>mybatis-generator-plugin</artifactId>
  <!-- 稳定版本 -->
  <version>1.0.6</version>
  <!-- 快照版本(最新提交到中央库，增加了部分功能，欢迎测试反馈) -->
  <version>1.0.7-SNAPSHOT</version>
</dependency>
```
---------------------------------------
### 1. 查询单条数据插件
对应表Mapper接口增加了方法  
插件：
```xml
<!-- 查询单条数据插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.SelectOneByExamplePlugin"/>
```
使用：  
```java
public interface TbMapper {    
    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    Tb selectOneByExample(TbExample example);
}
```
### 2. MySQL分页插件
对应表Example类增加了Mysql分页方法，limit(Integer rows)、limit(Integer offset, Integer rows)和page(Integer page, Integer pageSize)  
插件：
```xml
<!-- MySQL分页插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.LimitPlugin"/>
```
使用：  
```java
public class TbExample {
    /**
     * 这是Mybatis Generator拓展插件生成的属性(请勿删除).
     * This field corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    protected Integer offset;

    /**
     * 这是Mybatis Generator拓展插件生成的属性(请勿删除).
     * This field corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    protected Integer rows;
    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table rc_user_token
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample limit(Integer rows) {
        this.rows = rows;
        return this;
    }

    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table rc_user_token
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample limit(Integer offset, Integer rows) {
        this.offset = offset;
        this.rows = rows;
        return this;
    }

    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample page(Integer page, Integer pageSize) {
        this.offset = page * pageSize;
        this.rows = pageSize;
        return this;
    }
    
    // offset 和 rows 的getter&setter
    
    // 修正了clear方法
    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     */
    public void clear() {
        oredCriteria.clear();
        orderByClause = null;
        distinct = false;
        rows = null;
        offset = null;
    }
}
public class Test {
    public static void main(String[] args) {
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .limit(10)  // 查询前10条
                .limit(10, 10)  // 查询10~20条
                .page(1, 10)    // 查询第2页数据（每页10条）
        );
    }
}
```
### 3. 数据Model链式构建插件
这个是仿jquery的链式调用强化了表的Model的赋值操作  
插件：
```xml
<!-- 数据Model链式构建插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ModelBuilderPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 直接new表Model的内部Builder类，赋值后调用build()方法返回对象
        Tb table = new Tb.Builder()
               .field1("xx")
               .field2("xx")
               .field3("xx")
               .field4("xx")
               .build();
    }
}
```
### 4. Example 增强插件(example,andIf,orderBy)
* Criteria的快速返回example()方法。  
* Criteria链式调用增强，以前如果有按条件增加的查询语句会打乱链式查询构建，现在有了andIf(boolean ifAdd, CriteriaAdd add)方法可一直使用链式调用下去。
* Example增强了setOrderByClause方法，新增orderBy(String orderByClause)方法直接返回example，增强链式调用，可以一路.下去了。
* 继续增强orderBy(String orderByClause)方法，增加orderBy(String ... orderByClauses)方法，配合数据Model属性对应Column获取插件（ModelColumnPlugin）使用效果更佳。   
插件：
```xml
<!-- Example Criteria 增强插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ExampleEnhancedPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // -----------------------------------example-----------------------------------
        // 表Example.Criteria增加了工厂方法example()支持，使用后可链式构建查询条件使用example()返回Example对象
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                .andField2EqualTo("xxx")
                .example()
        );
        
        // -----------------------------------andIf-----------------------------------
        // Criteria增强了链式调用，现在一些按条件增加的查询条件不会打乱链式调用了
        // old
        TbExample oldEx = new TbExample();
        TbExample.Criteria criteria = oldEx
                .createCriteria()
                .andField1EqualTo(1)
                .andField2EqualTo("xxx");
        // 如果随机数大于0.5，附加Field3查询条件
        if (Math.random() > 0.5){
            criteria.andField3EqualTo(2)
                    .andField4EqualTo(new Date());
        }
        this.tbMapper.selectByExample(oldEx);

        // new
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                .andField2EqualTo("xxx")
                // 如果随机数大于0.5，附加Field3查询条件
                .andIf(Math.random() > 0.5, new TbExample.Criteria.ICriteriaAdd() {
                    @Override
                    public TbExample.Criteria add(TbExample.Criteria add) {
                        return add.andField3EqualTo(2)
                                .andField4EqualTo(new Date());
                    }
                })
                // 当然最简洁的写法是采用java8的Lambda表达式，当然你的项目是Java8+
                .andIf(Math.random() > 0.5, add -> add
                        .andField3EqualTo(2)
                        .andField4EqualTo(new Date())
                )
                .example()
        );
        
        // -----------------------------------orderBy-----------------------------------
        // old
        TbExample ex = new TbExample();
        ex.createCriteria().andField1GreaterThan(1);
        ex.setOrderByClause("field1 DESC");
        this.tbMapper.selectByExample(ex);

        // new
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .orderBy("field1 DESC")
                // 这个配合数据Model属性对应Column获取插件（ModelColumnPlugin）使用
                .orderBy(Tb.Column.field1.asc(), Tb.Column.field3.desc())
        );
    }
}
```
### 5. Example 目标包修改插件
Mybatis Generator 插件默认把Model类和Example类都生成到一个包下，这样该包下类就会很多不方便区分，该插件目的就是把Example类独立到一个新包下，方便查看。  
插件：
```xml
<!-- Example 目标包修改插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ExampleTargetPlugin">
    <!-- 修改Example类生成到目标包下 -->
    <property name="targetPackage" value="com.itfsw.mybatis.generator.dao.example"/>
</plugin>
```
### 6. 批量插入插件
提供了批量插入batchInsert和batchInsertSelective方法，因为方法使用了使用了JDBC的getGenereatedKeys方法返回插入主键，所以只能在MYSQL和SQLServer下使用。
需配合数据Model属性对应Column获取插件（ModelColumnPlugin）插件使用，实现类似于insertSelective插入列！  
PS:为了和Mybatis官方代码风格保持一致，1.0.5+版本把批量插入方法分开了，基于老版本1.0.5-版本生成的代码请继续使用“com.itfsw.mybatis.generator.plugins.BatchInsertOldPlugin”不影响使用  
PS:继续PS，本来想继续保留老版本代码，不影响老版本已经生成代码使用的，但是始终没有绕过BindingException，只能把代码移入BatchInsertOldPlugin类~~~~~  
插件：
```xml
<!-- 批量插入插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.BatchInsertPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 构建插入数据
        List<Tb> list = new ArrayList<>();
        list.add(
                new Tb.Builder()
                .field1(0)
                .field2("xx0")
                .field3(0)
                .createTime(new Date())
                .build()
        );
        list.add(
                new Tb.Builder()
                .field1(1)
                .field2("xx1")
                .field3(1)
                .createTime(new Date())
                .build()
        );
        // 普通插入，插入所有列
        this.tbMapper.batchInsert(list);
        // !!!下面按需插入指定列（类似于insertSelective），需要数据Model属性对应Column获取插件（ModelColumnPlugin）插件
        this.tbMapper.batchInsertSelective(list, Tb.Column.field1, Tb.Column.field2, Tb.Column.field3, Tb.Column.createTime);
    }
}
```
### 7. 逻辑删除插件
因为很多实际项目数据都不允许物理删除，多采用逻辑删除，所以单独为逻辑删除做了一个插件，方便使用。  
插件：
```xml
<xml>
    <!-- 逻辑删除插件 -->
    <plugin type="com.itfsw.mybatis.generator.plugins.LogicalDeletePlugin">
        <!-- 这里配置的是全局逻辑删除列和逻辑删除值，当然在table中配置的值会覆盖该全局配置 -->
        <!-- 逻辑删除列类型只能为数字、字符串或者布尔类型 -->
        <property name="logicalDeleteColumn" value="del_flag"/>
        <!-- 未设置该属性或者该属性的值为null或者NULL,逻辑删除时会把该字段置为NULL。 -->
        <property name="logicalDeleteValue" value="9"/>
    </plugin>
    
    <table tableName="tb">
        <!-- 这里可以单独表配置逻辑删除列和删除值，覆盖全局配置 -->
        <property name="logicalDeleteColumn" value="del_flag"/>
        <property name="logicalDeleteValue" value="1"/>
    </table>
</xml>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 1. 逻辑删除ByExample
        this.tbMapper.logicalDeleteByExample(
                new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                .example()
        );

        // 2. 逻辑删除ByPrimaryKey
        this.tbMapper.logicalDeleteByPrimaryKey(1L);

        // 3. 同时Example中提供了一个快捷方法来过滤逻辑删除数据
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                // 新增了一个andDeleted方法过滤逻辑删除数据
                .andDeleted(true)
                // 当然也可直接使用逻辑删除列的查询方法，我们数据Model中定义了一个逻辑删除常量DEL_FLAG
                .andDelFlagEqualTo(Tb.DEL_FLAG)
                .example()
        );
    }
}
```
### 8. 数据Model属性对应Column获取插件
项目中我们有时需要获取数据Model对应数据库字段的名称，一般直接根据数据Model的属性就可以猜出数据库对应column的名字，可是有的时候当column使用了columnOverride或者columnRenamingRule时就需要去看数据库设计了，所以提供了这个插件获取model对应的数据库Column。  
* 配合Example Criteria 增强插件（ExampleEnhancedPlugin）使用，这个插件还提供了asc()和desc()方法配合Example的orderBy方法效果更佳。
* 配合批量插入插件（BatchInsertPlugin）使用，batchInsertSelective(@Param("list") List<XXX> list, @Param("selective") XXX.Column ... insertColumns)。  

插件：
```xml
<!-- 数据Model属性对应Column获取插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ModelColumnPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 1. 获取Model对应column
        String column = Tb.Column.createTime.value();

        // 2. 配合Example Criteria 增强插件（ExampleEnhancedPlugin）使用orderBy方法
        // old
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .orderBy("field1 DESC, field3 ASC")
        );

        // better
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .orderBy(Tb.Column.field1.desc(), Tb.Column.field3.asc())
        );
        
        // 3. 配合批量插入插件（BatchInsertPlugin）使用实现按需插入指定列
        List<Tb> list = new ArrayList<>();
        list.add(
                new Tb.Builder()
                .field1(0)
                .field2("xx0")
                .field3(0)
                .field4(new Date())
                .build()
        );
        list.add(
                new Tb.Builder()
                .field1(1)
                .field2("xx1")
                .field3(1)
                .field4(new Date())
                .build()
        );
        // 这个会插入表所有列
        this.tbMapper.batchInsert(list);
        // 下面按需插入指定列（类似于insertSelective）
        this.tbMapper.batchInsertSelective(list, Tb.Column.field1, Tb.Column.field2, Tb.Column.field3, Tb.Column.createTime);
    }
}
```
### 9. 存在即更新插件
1. 使用MySQL的[“insert ... on duplicate key update”](https://dev.mysql.com/doc/refman/5.7/en/insert-on-duplicate.html)实现存在即更新操作，简化数据入库操作（[[issues#2]](https://github.com/itfsw/mybatis-generator-plugin/issues/2)）。  
2. 在开启allowMultiQueries=true（默认不会开启）情况下支持upsertByExample，upsertByExampleSelective操作，但强力建议不要使用（需保证团队没有使用statement提交sql,否则会存在sql注入风险）（1.0.7-SNAPSHOT）（[[issues#2]](https://github.com/itfsw/mybatis-generator-plugin/issues/2)）。

插件：
```xml
<!-- 存在即更新插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.UpsertPlugin">
    <!-- 
    支持upsertByExample，upsertByExampleSelective操作
    ！需开启allowMultiQueries=true多条sql提交操作，所以不建议使用！
    ! 目前在1.0.7-SNAPSHOT支持，有需要的可以下载试用
    -->
    <property name="allowMultiQueries" value="true"/>
</plugin>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 1. 未入库数据入库，执行insert
        Tb tb = new Tb.Builder()
                .field1(1)
                .field2("xx0")
                .delFlag((short)0)
                .build();
        int k0 = this.tbMapper.upsert(tb);
        // 2. 已入库数据再次入库,执行update（！！需要注意如触发update其返回的受影响行数为2）
        tb.setField2("xx1");
        int k1 = this.tbMapper.upsert(tb);

        // 3. 类似insertSelective实现选择入库
        Tb tb1 = new Tb.Builder()
                .field1(1)
                .field2("xx0")
                .build();
        int k2 = this.tbMapper.upsertSelective(tb1);
        tb1.setField2("xx1");
        int k3 = this.tbMapper.upsertSelective(tb1);

        // 4. 开启allowMultiQueries后增加upsertByExample，upsertByExampleSelective但强力建议不要使用（需保证团队没有使用statement提交sql,否则会存在sql注入风险）
        Tb tb2 = new Tb.Builder()
                .field1(1)
                .field2("xx0")
                .field3(1003)
                .delFlag((short) 0)
                .build();
        int k4 = this.tbMapper.upsertByExample(tb2,
                new TbExample()
                        .createCriteria()
                        .andField3EqualTo(1003)
                        .example()
        );
        tb2.setField2("xx1");
        // !!! upsertByExample,upsertByExampleSelective触发更新时，更新条数返回是有问题的，这里只会返回0
        // 这是mybatis自身问题，也是不怎么建议开启allowMultiQueries功能原因之一
        int k5 = this.tbMapper.upsertByExample(tb2,
                new TbExample()
                        .createCriteria()
                        .andField3EqualTo(1003)
                        .example()
        );
        // upsertByExampleSelective 用法类似
    }
}
```
### 10. Selective选择插入更新增强插件（1.0.7-SNAPSHOT）
项目中往往需要指定某些字段进行插入或者更新，或者把某些字段进行设置null处理，这种情况下原生xxxSelective方法往往不能达到需求，因为它的判断条件是对象字段是否为null，这种情况下可使用该插件对xxxSelective方法进行增强。  
WAREN:配置插件时请把插件配置在所有插件末尾最后执行，这样才能把上面提供的某些插件的Selective方法也同时增强！

插件：
```xml
<!-- Selective选择插入更新增强插件！请配在所有插件末尾以便最后执行 -->
<plugin type="com.itfsw.mybatis.generator.plugins.SelectiveEnhancedPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 1. 指定插入或更新字段
        Tb tb = new Tb.Builder()
                .field1(1)
                .field2("xx2")
                .field3(1)
                .field4(new Date())
                .createTime(new Date())
                .delFlag(Tb.DEL_FLAG)
                .build();
        // 只插入或者更新field1,field2字段
        this.tbMapper.insertSelective(tb.selective(Tb.Column.field1, Tb.Column.field2));
        this.tbMapper.updateByExampleSelective(tb.selective(Tb.Column.field1, Tb.Column.field2),
                new TbExample()
                        .createCriteria()
                        .andIdEqualTo(1l)
                        .example()
        );
        this.tbMapper.updateByPrimaryKeySelective(tb.selective(Tb.Column.field1, Tb.Column.field2));
        this.tbMapper.upsertSelective(tb.selective(Tb.Column.field1, Tb.Column.field2));
        this.tbMapper.upsertByExampleSelective(tb.selective(Tb.Column.field1, Tb.Column.field2),
                new TbExample()
                        .createCriteria()
                        .andField3EqualTo(1)
                        .example()
        );

        // 2. 更新某些字段为null
        this.tbMapper.updateByPrimaryKeySelective(
                new Tb.Builder()
                .id(1l)
                .field1(null)   // 方便展示，不用设也可以
                .build()
                .selective(Tb.Column.field1)
        );
    }
}
```