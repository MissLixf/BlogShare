# PYMODM使用TIPS

* `ListField`存储其他文档的引用列表时，可以这样定义：

  ```python
  class MongoPaper(MongoModel):
      """mongo试卷"""
      paper_id = fields.IntegerField(verbose_name='sql试卷pk', primary_key=True)
      outlines = fields.ListField(field=fields.ReferenceField(Outline), verbose_name='提纲', blank=True)
  ```

  可以直接为`outlines`字段赋值：

  ```python
  m_paper = MongoPaper(
      paper_id=123, 
      outlines=['5cbdaf60f4880d0cd8a77ee0', '5cbdb25ef4880d4de4f32fe1', '5cbd8d03f4880d5d543c3263'])
  
  # 保存时，会自动将列表中的_id字符串转化为ObjectId对象
  if m_paper.is_valid():
      m_paper.save()
  ```

* 如果要插入子文档，利用`to_son()`方法，将model转化为son对象，或者直接用字典字段。

* `MongoModel`实例的`pk`属性，返回`ObjectId`对象

* `MongoModel`中的`ReferenceField`字段，在mongo数据库中存储为`ObjectId`数据类型

* `QuerySet`的`only`方法默认会返回`_id`, `_cls`字段

* 如果在定义模型时，通过`primary_key`指定某个字段为主键，保存时可以使用该字段。但查询时，仍然通过`_id`字段

* 如果自定义A集合的主键为`int`，B集合中的文档通过`ReferenceField`引用了A集合中的文档，那么在MongoDB存储中，引用字段类型是`int`而不是`ObjectId`，但是Pymodm依然可以自动解析引用：

  ```json
  {
      "_id" : ObjectId("5cd23e44f4880d8c48cd29a5"),
      "status" : 0,
      "score" : 19,
      "cost_time" : 3600,
      "submit_time" : 1557282372,
      "answer_sheet" : [ 
          {
              "user_answer" : "C",
              "is_correct" : 2,
              "points" : 0,
              "exercise" : ObjectId("5cb99660f4880d50385dfeff"),
              "outline_index" : 1,
              "exercise_index" : 1,
              "type" : 0,
              "model" : 0,
              "is_child" : false,
              "parent" : null,
              "score" : 2
          }, 
      ],
      "user_id" : 1062,
      "paper" : 11,  // 引用paper
      "_cls" : "api_service.models.UserPaper"
  }
  ```

  ```python
  print(user_paper.paper.name)  # 第一次模拟测试
  ```

* `EmbeddedMongoModel`子文档的`Meta`属性，也可以设定`final=True`，来禁止生成`_cls`字段

* 如果已经针对某个集合建立了索引，修改该索引时提示索引已存在，需要先删除旧的索引。

* mongoDB的默认主键是`ObjectId`对象，是一个24位的16进制字符串，其中前4位表示时间，在python中可以通过调用该对象的`generation_time`属性来获取，源码如下：

  ```python
  @property
  def generation_time(self):
      """A :class:`datetime.datetime` instance representing the time of
          generation for this :class:`ObjectId`.
  
          The :class:`datetime.datetime` is timezone aware, and
          represents the generation time in UTC. It is precise to the
          second.
          """
      timestamp = struct.unpack(">i", self.__id[0:4])[0]
      return datetime.datetime.fromtimestamp(timestamp, utc)
  ```

* pymodm会自动请求`ReferenceField`字段引用的对象(除非在`no_auto_dereference`上下文中)：

  ```python
  print(isinstance(exercise.parent, MongoModel))  # True
  ```

  但是如果调用`to_son().to_dict()`方法后，这种引用不复存在。

* 获取MongoModel表的所有字段：

  ```python
  MongoModel._mongometa.get_fields()  # 由field对象组成的列表
  # field对象的attname属性可以获取具体的字段名
  [field_obj.attname for field_obj in type(model_obj)._mongometa.get_fields()]
  ```

* pymodm如果解析到引用列表，会自动请求列表中的每个引用对象（包括打印时）。

* 设置某字段的`blank=True`属性后，保存文档时，如果该字段未赋值，那么文档在MongoDB数据库中的存储不含该字段。但是在获取文档对象该字段的值时，并不会报`AttributeError`，而是返回`None`

* `project`指定文档的返回字段时，不能指定`ReferenceField`引用文档的具体字段。

* 聚合实例：

  ```python
  # 聚合计算各试卷的待批和已批数
  count_info = models.UserPaper.objects.aggregate(
      {'$match': {'paper': {'$in': paper_id_list}}},
      {'$group': {
          '_id': {'paper': '$paper', 'status': '$status'},  # 根据paper和status字段进行分组，然后进行分组求和
          'count': {'$sum': 1}
      }})
  """ 
  [
      {u'count': 2, u'_id': {u'status': 2, u'paper': 11}}, -- paper11，status2的，合计2个
      {u'count': 3, u'_id': {u'status': 1, u'paper': 11}}, -- paper11, status1的，合计3个
      {u'count': 1, u'_id': {u'status': 1, u'paper': 12}}  -- paper12, status1的，合计1个
  ]
  
  # 将聚合结果整理为如下格式，传入序列化上下文中
  {
      paper_id: (status_1_count, status_2_count),
      paper_id: (status_1_count, status_2_count),
  }
  """
  ```

  ```python
  # 根据是否小于等于当前用户分数进行分组统计
  rank_info = models.UserPaper.objects.aggregate(
      {'$match': {'paper': paper_id}},
      {'$group': {
          '_id': {
              'behind': {'$lte': ['$score', user_paper_obj.score]},
          },
          'count': {'$sum': 1}
      }})
  """
  如果都小于等于，那么列表中只有一个元素且behind为True; 
  如果都大于，那么也是一个元素，且behind为False; 
  都存在的话是两个元素
  [
      {u'count': 1, u'_id': {u'behind': False}},   -- 比当前用户分数高的有1个
      {u'count': 4, u'_id': {u'behind': True}}     -- 比当前用户分数低或等于的有4个
  ]    
  """
  ```

* 往`fields.DateTimeField`字段中写入utc时间：

  ```python
  datetime.datetime.utcnow()
  ```

  

