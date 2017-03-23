# Restful API 的设计规范

## 1.uri
* URI 表示资源，资源一般对应服务器端领域模型中的实体类。

### 规范
* 不用大写；
* 用中杠-不用下杠_；
* 参数列表要encode；
* URI中的名词表示资源集合，使用复数形式。\

### 资源集合 vs 单个资源
URI表示资源的两种方式：资源集合、单个资源。

* 资源集合：
```
/zoos //所有动物园
/zoos/1/animals //id为1的动物园中的所有动物
```
* 单个资源：
```
/zoos/1 //id为1的动物园
/zoos/1;2;3 //id为1，2，3的动物园
```

### 避免层级过深的URI
/在url中表达层级，用于按实体关联关系进行对象导航，一般根据id导航。

过深的导航容易导致url膨胀，不易维护，如 GET /zoos/1/areas/3/animals/4，尽量使用查询参数代替路径中的实体导航，如GET /animals?zoo=1&area=3；

### 对Composite资源的访问
服务器端的组合实体必须在uri中通过父实体的id导航访问。
>>>组合实体不是first-class的实体，它的生命周期完全依赖父实体，无法独立存在，在实现上通常是对数据库表中某些列的抽象，不直接对应表，也无id。一个常见的例子是 User — Address，Address是对User表中zipCode/country/city三个字段的简单抽象，无法独立于User存在。必须通过User索引到Address：GET /user/1/addresses

## 2. Request
### HTTP方法

* 通过标准HTTP方法对资源CRUD
* GET：查询
```
 GET /zoos
 GET /zoos/1
 GET /zoos/1/employees
```

* POST：创建单个资源。POST一般向“资源集合”型uri发起
```
POST /animals  //新增动物
POST /zoos/1/employees //为id为1的动物园雇佣员工
```

* PUT：更新单个资源（全量），客户端提供完整的更新后的资源。与之对应的是 PATCH，PATCH 负责部分更新，客户端提供要更新的那些字段。PUT/PATCH一般向“单个资源”型uri发起
```
PUT /animals/1
PUT /zoos/1
```

* DELETE：删除
```
DELETE /zoos/1/employees/2
DELETE /zoos/1/employees/2;4;5
DELETE /zoos/1/animals  //删除id为1的动物园内的所有动物
```

### 安全性和幂等性
* 安全性：不会改变资源状态，可以理解为只读的；
* 幂等性：执行1次和执行N次，对资源状态改变的效果是等价的。
 
|| 安全性 | 幂等性
- | - | - 
GET| √ | √ 
POST | × | ×
PUT | × | √
DELETE | × | √

* 安全性和幂等性均不保证反复请求能拿到相同的response。以 DELETE 为例，第一次DELETE返回200表示删除成功，第二次返回404提示资源不存在，这是允许的。

### 复杂查询
* 查询可以捎带以下参数：

. | 示例 | 备注
- | - | -
过滤条件 | ?type=1&age=16 | 允许一定的uri冗余，如/zoos/1与/zoos?id=1
排序 | ?sort=age,desc |
分页 | ?limit=10&offset=3 |

## 待续。。。。。。。