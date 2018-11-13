## 一、前言

从一开始，MyBatis就是一个XML驱动的框架。基于XML配置，映射语句也是用XML定义的。而从MyBatis 3开始，可以使用注解。

## 二、注解列表

| Annotation        | Target   |  XML equivalent  | Description |
| :--------   | :-----  | :---- | :---- |
| @CacheNamespace | Class |  `<cache>`  |为给定的命名空间（比如类）配置缓存。<br> 属性：implemetation,  eviction,  flushInterval,  size  和  readWrite 。 |
| @Property|    N/A|`<property>`|   Specifies the property value or placeholder(can replace by configuration properties that defined at the mybatis-config.xml). Attributes: name, value. (Available on MyBatis 3.4.2+)|
|@CacheNamespaceRef|    Class|  `<cacheRef>`| 参照另外一个命名空间的缓存来使用。<br>属性：value  - 也就是类的完全限定名。<br> name - 名称|
|@ConstructorArgs   | Method|   <constructor>   |收集一组结果传递给对象构造方法。<br> 属性：value，是形式参数的数组|
| @Arg  | N/A   | `<arg>`  `<idArg>` |单独的构造方法参数，是ConstructorArgs  集合的一部分。<br>属性:id,column,javaTypetypeHandler。<br>id属性是布尔值，来标识用于比较的属性，和<idArg>XML 元素相似.|
|@TypeDiscriminator|    Method |    <discriminator> |一组实例值被用来决定结果映射的表现。<br> 属性：Column, javaType ,  jdbcType typeHandler，cases。cases属性就是实例的数组。|
|@Case| N/A |<case> |单独实例的值和它对应的映射。<br> 属性：value  ，type ，results 。<br>Results 属性是结果数组，因此这个注解和实际的ResultMap 很相似，由下面的  Results注解指定|
|@Results   |Method |<resultMap>|   结果映射的列表，包含了一个特别结果列如何被映射到属性或字段的详情。<br> 属性：value ，是Result注解的数组|
|@Result |  N/A | `<result>` `<id>`  | 在列和属性或字段之间的单独结果映射。<br> 属性：id ，column ， property ，javaType ，jdbcType ，type Handler ，one，many。<br> id 属性是一个布尔值，表示了应该被用于比较的属性。<br> one 属性是单独的联系，和 <association> 相似，<br> 而many 属性是对集合而言的，和<collection>相似。|
|@One   |N/A    |<association>  |复杂类型的单独属性值映射。<br>属性：select，已映射语句（也就是映射器方法）的完全限定名，它可以加载合适类型的实例。注意：联合映射在注解 API中是不支持的。
|@Many  |N/A    |<collection>   |复杂类型的集合属性映射。<br>属性：select，是映射器方法的完全限定名，它可加载合适类型的一组实例。注意：联合映射在 Java注解中是不支持的。
|@MapKey|   Method |    | This is used on methods which return type is a Map. It is used to convert a List of result objects as a Map based on a property of those objects. Attributes: value, which is a property used as the key of the map.|
|@Options   |Method |Attributes of mapped statements.|  这个注解提供访问交换和配置选项的宽广范围，它们通常在映射语句上作为属性出现。而不是将每条语句注解变复杂，Options 注解提供连贯清晰的方式来访问它们。<br>属性：useCache=true，flushCache=false，resultSetType=FORWARD_ONLY，statementType=PREPARED，fetchSize= -1，timeout=-1 ，useGeneratedKeys=false ，keyProperty=”id“。理解Java 注解是很重要的，因为没有办法来指定“null ”作为值。因此，一旦你使用了 Options注解，语句就受所有默认值的支配。要注意什么样的默认值来避免不期望的行为
|  @Insert <br>  @Update <br> @Delete  <br> @Select | Method    | <insert> <br> <update> <br> <delete> <br> <select>|这些注解中的每一个代表了执行的真实 SQL。它们每一个都使用字符串数组（或单独的字符串）。如果传递的是字符串数组，它们由每个分隔它们的单独空间串联起来。<br>属性：value，这是字符串数组用来组成单独的SQL语句
|@InsertProvider  <br> @UpdateProvider <br> @DeleteProvider<br> @SelectProvider | Method |<insert><update><delete><select>| 这些可选的SQL注解允许你指定一个类名和一个方法在执行时来返回运行的SQL。基于执行的映射语句， MyBatis会实例化这个类，然后执由 provider指定的方法. 这个方法可以选择性的接受参数对象作为它的唯一参数，但是必须只指定该参数或者没有参数。<br>属性：type，method。type 属性是类的完全限定名。method  是该类中的那个方法名。|
|@Param|    Parameter|  N/A |当映射器方法需多个参数，这个注解可以被应用于映射器方法参数来给每个参数一个名字。否则，多参数将会以它们的顺序位置来被命名。<br>比如#{1}，#{2} 等，这是默认的。使用@Param(“person”)，SQL中参数应该被命名为#{person}。
|@SelectKey |Method |<selectKey>    |This annotation duplicates the <selectKey> functionality for methods annotated with @Insert, @InsertProvider, @Update, or @UpdateProvider. It is ignored for other methods. If you specify a@SelectKey annotation, then MyBatis will ignore any generated key properties set via the @Optionsannotation, or configuration properties. Attributes: statement an array of strings which is the SQL statement to execute, keyProperty which is the property of the parameter object that will be updated with the new value, before which must be either true or false to denote if the SQL statement should be executed before or after the insert, resultType which is the Java type of the keyProperty, and statementType is a type of the statement that is any one of STATEMENT, PREPARED or CALLABLE that is mapped to Statement, PreparedStatement and CallableStatement respectively. The default is PREPARED. |
|@ResultMap|    Method| N/A |This annotation is used to provide the id of a <resultMap> element in an XML mapper to a @Select or @SelectProvider annotation. This allows annotated selects to reuse resultmaps that are defined in XML. This annotation will override any @Results or @ConstructorArgs annotation if both are specified on an annotated select.
|@ResultType    |Method|    N/A |This annotation is used when using a result handler. In that case, the return type is void so MyBatis must have a way to determine the type of object to construct for each row. If there is an XML result map, use the @ResultMap annotation. If the result type is specified in XML on the <select> element, then no other annotation is necessary. In other cases, use this annotation. For example, if a @Select annotated method will use a result handler, the return type must be void and this annotation (or @ResultMap) is required. This annotation is ignored unless the method return type is void.
|@Flush |Method |N/A    |If this annotation is used, it can be called the SqlSession#flushStatements() via method defined at a Mapper interface.(MyBatis 3.3 or above)|

## 三、参考资料
1. [mybatis官方文档-Mapper Annotation](http://www.mybatis.org/mybatis-3/java-api.html)







