### 大字段处理
#### 使用延迟属性抓取（Using lazy property fetching）

     Hibernate3 对单独的属性支持延迟抓取，这项优化技术也被称为组抓取（fetch groups）。 请注意，该技术更多的属于市场特性。在实际应用中，优化行读取比优化列读取更重要。但是，仅载入类的部分属性在某些特定情况下会有用，例如在原有表中拥有几百列数据、数据模型无法改动的情况下。

     可以在映射文件中对特定的属性设置 lazy，定义该属性为延迟载入。

     还有一种可以优化的方法，它使用 HQL 或条件查询的投影（projection）特性，可以避免读取非必要的列， 这一点至少对只读事务是非常有用的。它无需在代码构建时“二进制指令”处理，因此是一个更加值得选择的解决方法。

     有时你需要在 HQL 中通过抓取所有属性，强行抓取所有内容。

Example:

<pre>
     @Lob //声明属性对应的是一个大文件数据字段
     @Basic(fetch = FetchType.LAZY) //设置为延迟加载，当我们在数据库中取这条记录的时候，不会去取这个字段
     private Byte[] file;
</pre>

SELECT id, brief, category, create_time, creator, last_modified_time, parent_category_id, sequence, sub_title, title, type, view_times, doc, `name`
FROM
	help_doc helpdoc0_
WHERE
	helpdoc0_.id =2

#### Use HQL, new constructor, and exclude large column from query

TODO: add hql example
