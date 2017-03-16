#资源 Resource
##面向资源的API
- 集合包含相同类型的资源列表 例如	一个用户有联系人集合

- 资源具有一些​​状态和零个或多个子资源 每个子资源可以是简单资源或收集资源
  For example, Gmail API has a collection of users, each user has a collection of messages, a collection of threads, a collection of labels, a profile resource, and several setting resources.

- 在请求中，get是读取一个资源，post是创造一个资源 put和patch是更新一个存在的资源，delete是删除一个资源

  Request methods: the action a client wants to perform on a specific endpoint. GET is used to retrieve a resource, POST, to create one, PUT and PATCH to update an existing one, and DELETE an existing one