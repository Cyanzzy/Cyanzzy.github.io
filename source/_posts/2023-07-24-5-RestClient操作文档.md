---
title: 5 RestClient操作文档
date: 2023-07-24 17:18:08
tags: 
  - Elasticsearch
categories: 
  - Technology
---

> 为了与索引库操作分离

- 初始化RestHighLevelClient
- 酒店数据在数据库，需要利用IHotelService去查询，所以注入这个接口

```java
@SpringBootTest
public class HotelDocumentTest {
    @Autowired
    private IHotelService hotelService;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://47.113.217.195:9200")
        ));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}

```

#  新增文档

将数据库的酒店数据查询出来，写入elasticsearch中。

> 数据库查询后的结果是一个Hotel类型的对象

```java
@Data
@TableName("tb_hotel")
public class Hotel {
    @TableId(type = IdType.INPUT)
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String longitude;
    private String latitude;
    private String pic;
}
```

与我们的索引库结构存在差异：longitude和latitude需要合并为location

因此，我们需要定义一个新的类型，与索引库结构吻合：

```java
@Data
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
    private String pic;

    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location = hotel.getLatitude() + ", " + hotel.getLongitude();
        this.pic = hotel.getPic();
    }
}
 
```

> 新增文档的DSL语句

```json
POST /{索引库名}/_doc/1
{
    "name": "Jack",
    "age": 21
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-es-20230706-13.png)

> 三部曲

- 创建Request对象
- 准备请求参数，也就是DSL中的JSON文档
- 发送请求

变化的地方在于，这里直接使用client.xxx()的API，不再需要client.indices()了。



> 导入酒店数据，基本流程一致，但是需要考虑几点变化：

- 酒店数据来自于数据库，我们需要先查询出来，得到hotel对象
- hotel对象需要转为HotelDoc对象
- HotelDoc需要序列化为json格式

> 业务步骤

1. 根据id查询酒店数据Hotel

2. 将Hotel封装为HotelDoc

3. 将HotelDoc序列化为JSON

4. 创建IndexRequest，指定索引库名和id
5. 准备请求参数，也就是JSON文档
6. 发送请求

```java
@Test
void testAddDocument() throws IOException {
    // 1.根据id查询酒店数据
    Hotel hotel = hotelService.getById(61083L);
    // 2.转换为文档类型
    HotelDoc hotelDoc = new HotelDoc(hotel);
    // 3.将HotelDoc转json
    String json = JSON.toJSONString(hotelDoc);

    // 1.准备Request对象
    IndexRequest request = new IndexRequest("hotel").id(hotelDoc.getId().toString());
    // 2.准备Json文档
    request.source(json, XContentType.JSON);
    // 3.发送请求
    client.index(request, RequestOptions.DEFAULT);
}
```

# 查询文档

> 查询的DSL语句如下：

```json
GET /hotel/_doc/{id}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-es-20230706-14.png)

可以看到，结果是一个JSON，其中文档放在一个`_source`属性中，因此解析就是拿到`_source`，反序列化为Java对象即可。

> 三部曲

* 准备Request对象。这次是查询，所以是GetRequest

- 发送请求，得到结果。因为是查询，这里调用client.get()方法
- 解析结果，就是对JSON做反序列化

```java
@Test
void testGetDocumentById() throws IOException {
    // 1.准备Request
    GetRequest request = new GetRequest("hotel", "61082");
    // 2.发送请求，得到响应
    GetResponse response = client.get(request, RequestOptions.DEFAULT);
    // 3.解析响应结果
    String json = response.getSourceAsString();

    HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
    System.out.println(hotelDoc);
}
```





# 删除文档

> 删除的DSL 

```json
DELETE /hotel/_doc/{id}
```

> 三部曲

- 准备Request对象，因为是删除，这次是DeleteRequest对象。要指定索引库名和id
- 准备参数，无参
- 发送请求。因为是删除，所以是client.delete()方法

```java
@Test
void testDeleteDocument() throws IOException {
    // 1.准备Request
    DeleteRequest request = new DeleteRequest("hotel", "61083");
    // 2.发送请求
    client.delete(request, RequestOptions.DEFAULT);
}
```

#  修改文档

> 语法说明

- 全量修改：本质是先根据id删除，再新增
- 增量修改：修改文档中的指定字段值

> 在RestClient的API中，全量修改与新增的API完全一致，判断依据是ID：

- 如果新增时，ID已经存在，则修改
- 如果新增时，ID不存在，则新增

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-es-20230706-15.png)

> 三部曲

- 准备Request对象。这次是修改，所以是UpdateRequest
- 准备参数。也就是JSON文档，里面包含要修改的字段
- 更新文档。这里调用client.update()方法

```java
@Test
void testUpdateDocument() throws IOException {
    // 1.准备Request
    UpdateRequest request = new UpdateRequest("hotel", "61083");
    // 2.准备请求参数
    request.doc(
        "price", "952",
        "starName", "四钻"
    );
    // 3.发送请求
    client.update(request, RequestOptions.DEFAULT);
}
```





# 批量导入文档

> 利用BulkRequest批量将数据库数据导入到索引库中。

- 利用mybatis-plus查询酒店数据

- 将查询到的酒店数据（Hotel）转换为文档类型数据（HotelDoc）

- 利用JavaRestClient中的BulkRequest批处理，实现批量新增文档

批量处理BulkRequest，其本质就是将多个普通的CRUD请求组合在一起发送。其中提供了一个add方法，用来添加其他请求：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-es-20230706-16.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-es-20230706-17.png)

> 三部曲

- 创建Request对象。这里是BulkRequest
- 准备参数。批处理的参数，就是其它Request对象，这里就是多个IndexRequest
- 发起请求。这里是批处理，调用的方法为client.bulk()方法

```java
@Test
void testBulkRequest() throws IOException {
    // 批量查询酒店数据
    List<Hotel> hotels = hotelService.list();

    // 1.创建Request
    BulkRequest request = new BulkRequest();
    // 2.准备参数，添加多个新增的Request
    for (Hotel hotel : hotels) {
        // 2.1.转换为文档类型HotelDoc
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 2.2.创建新增文档的Request对象
        request.add(new IndexRequest("hotel")
                    .id(hotelDoc.getId().toString())
                    .source(JSON.toJSONString(hotelDoc), XContentType.JSON));
    }
    // 3.发送请求
    client.bulk(request, RequestOptions.DEFAULT);
}
```

```json
GET /hotel/_search
```

# 总结

文档操作的基本步骤：

- 初始化RestHighLevelClient
- 创建XxxRequest。XXX是Index、Get、Update、Delete、Bulk
- 准备参数（Index、Update、Bulk时需要）
- 发送请求。调用RestHighLevelClient#.xxx()方法，xxx是index、get、update、delete、bulk
- 解析结果（Get时需要）