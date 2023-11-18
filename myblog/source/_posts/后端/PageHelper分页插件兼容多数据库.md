springboot + 多数据源 mysql、oracle + pagehelper 分页

如果使用单数据源，可在配置文件中配置如下：

```yml
pagehelper:
  helperDialect: mysql
  reasonable: false
  supportMethodsArguments: true
  params: count=countSql
```

如果使用多数据源，需要修改 pagehelper 配置项，配置如下

```yml
pagehelper:
  reasonable: false
  supportMethodsArguments: true
  params: count=countSql
  # 默认false,当为true时，自动检验适合的数据库
  auto-dialect:
    true
    # 这个一定要加上，不然mysql和oracle分页两个只能用一个，另一个会报错，加上后，两中数据库分页都可以用了
  auto-runtime-dialect: true
```

一些参数的说明：
(1) helperDialect：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。 你可以配置 helperDialect 属性来指定分页插件使用哪种方言。配置时，可以使用下面的缩写值：
oracle,mysql,mariadb,sqlite,hsqldb,postgresql,db2,sqlserver,informix,h2,sqlserver2012,derby
特别注意：使用 SqlServer2012 数据库时，需要手动指定为 sqlserver2012，否则会使用 SqlServer2005 的方式进行分页。
你也可以实现 AbstractHelperDialect，然后配置该属性为实现类的全限定名称即可使用自定义的实现方法。

(2) offsetAsPageNum：默认值为 false，该参数对使用 RowBounds 作为分页参数时有效。 当该参数设置为 true 时，会将 RowBounds 中的 offset 参数当成 pageNum 使用，可以用页码和页面大小两个参数进行分页。

(3) rowBoundsWithCount：默认值为 false，该参数对使用 RowBounds 作为分页参数时有效。 当该参数设置为 true 时，使用 RowBounds 分页会进行 count 查询。

(4) pageSizeZero：默认值为 false，当该参数设置为 true 时，如果 pageSize=0 或者 RowBounds.limit = 0 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 Page 类型）。

(5) reasonable：分页合理化参数，默认值为 false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认 false 时，直接根据参数进行查询。

(6) params：为了支持 startPage(Object params)方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 pageNum,pageSize,count,pageSizeZero,reasonable，不配置映射的用默认值， 默认值为 pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero。

(7) supportMethodsArguments：支持通过 Mapper 接口参数来传递分页参数，默认值 false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页。 使用方法可以参考测试代码中的 com.github.pagehelper.test.basic 包下的 ArgumentsMapTest 和 ArgumentsObjTest。

(8) autoRuntimeDialect：默认值为 false。设置为 true 时，允许在运行时根据多数据源自动识别对应方言的分页 （不支持自动选择 sqlserver2012，只能使用 sqlserver）
