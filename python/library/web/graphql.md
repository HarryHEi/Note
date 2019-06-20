---
title: GraphQL
date: 2019-06-13 16:14:00
tags: [python]
---

# GraphQL
[GraphQL](https://www.howtographql.com/basics/0-introduction/)是一种新的API标准，旨在实现前后端独立开发。

REST:
特点: 拥有对个访问端点，针对每个接口设计接收和返回的数据格式。

优点: 方便针对不同端点设计接口的缓存策略。
缺点: 接口设计不够灵活，不能满足所有客户端需求，可能会产生冗余数据。

GraphQL:
特点: 只有一个访问端点，由用户的请求决定返回的数据格式。

优点: 接口设计灵活，客户端根据需要自定义查询内容。
缺点: 难以设计缓存策略。


# N + 1
使用`DataLoader`解决[N + 1](https://apirobot.me/posts/django-graphql-solving-n-1-problem-using-dataloaders)问题。

有如下数据结构
```
class Company(models.Model):
    name = models.CharField(max_length=16)


class Dept(models.Model):
    company = models.ForeignKey('Company', on_delete=models.CASCADE, related_name='dept')
    name = models.CharField(max_length=16)
```

`Company`是`Dept`的父节点，如果希望查询的`Dept`列表中包含`Company`的`name`，为了避免多次查询，可以定义一个根据`id`列表查询`Company`名称列表的`DataLoader`。
```
from promise import Promise
from promise.dataloader import DataLoader

class CompanyNameLoader(DataLoader):
    def batch_load_fn(self, keys):
        return Promise.resolve([company.name for company in Company.objects.filter(id__in=keys)])
```

当实例化一个`DataLoader`后，调用`load`加载一个`key`，或者`loads`加载多个`key`，但是并不会立即调用查询，而是缓存下来，也就是`batch_load_fn`只会执行一次。

所以需要做的工作是将`CompanyNameLoader`实例和一次会话绑定，然后在`resolve`中使用`load`或者`loads`加载名称。
```
from graphene_django import DjangoObjectType

class DeptNode(DjangoObjectType):
    class Meta:
        model = Dept
        filter_fields = ['id', 'name', 'company__name']
        interfaces = (relay.Node, )

    company_name = String()

    @staticmethod
    def resolve_company_name(parent, info):
        request = info.context
        if not hasattr(info.context, 'company_name_loader'):
            company_name_loader = CompanyNameLoader()
            request.company_name_loader = company_name_loader
        else:
            company_name_loader = request.company_name_loader

        return company_name_loader.load(key=parent.company_id)
```

放到Query中
```
from graphene_django.filter import DjangoFilterConnectionField

class Query:
    dept = relay.Node.Field(DeptNode)
    all_dept = DjangoFilterConnectionField(DeptNode)
```

同样的，查询`Company`列表需要包含`Dept`信息时，也需要通过`DataLoader`避免多次查询。

和之前类似，也是可以缓存需要查询的`company_id`，不同的是，每个`company`可以有多个或者0个`dept`，这时需要返回`dept`列表的列表，形如`[[], [dept_1, dept_2], ...]`

定义`CompanyDeptLoader`，遍历查询结果，利用`defaultdict`快速构建映射，然后按照`keys`的顺序构建返回`dept`列表
```
class CompanyDeptLoader(DataLoader):
    @staticmethod
    def batch_load_fn(keys):
        company_to_dept = defaultdict(list)

        for dept in Dept.objects.filter(company__in=keys):
            company_to_dept[dept.company_id].append(dept)

        return Promise.resolve([company_to_dept.get(key, []) for key in keys])
```

使用方法相同
```
class CompanyNode(DjangoObjectType):
    class Meta:
        model = Company
        filter_fields = ['id', 'name']
        interfaces = (relay.Node, )
        connection_class = CompanyConnection

    @staticmethod
    def resolve_dept(parent, info):
        request = info.context
        if not hasattr(info.context, 'company_dept_loader'):
            company_dept_loader = CompanyDeptLoader()
            request.company_dept_loader = company_dept_loader
        else:
            company_dept_loader = request.company_dept_loader

        return company_dept_loader.load(parent.id)
```

放到`Query`中
```
class Query:
    company = relay.Node.Field(CompanyNode)
    all_company = DjangoFilterConnectionField(CompanyNode)
```

# 分页和筛选
[relay](https://docs.graphene-python.org/projects/django/en/latest/queries/#relay)用于处理分页和分片，[django-filter](https://docs.graphene-python.org/projects/django/en/latest/tutorial-relay/#schema)处理筛选，[配合使用](https://github.com/graphql-python/graphene-django/issues/636)处理筛选和分页。

定义`Connection`
```
class CompanyConnection(relay.Connection):
    class Meta:
        abstract = True
```

`DjangoObjectType`设置`interfaces`和`connection_class`
```
class CompanyNode(DjangoObjectType):
    class Meta:
        model = Company
        filter_fields = {
            'id': ['exact'],
            'name': ['exact', 'icontains', 'istartswith'],
        }
        interfaces = (relay.Node, )
        connection_class = CompanyConnection
```

定义`Query`
```
class Query:
    company = relay.Node.Field(CompanyNode)
    all_company = DjangoFilterConnectionField(CompanyNode)
```
测试
```
{
  allDept (first:2, company_Name: "bilibili"){
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
    edges {
      node {
        id
        name
        companyName
      }
    }
  }
}
```
返回值
```
{
  "data": {
    "allDept": {
      "pageInfo": {
        "hasNextPage": true,
        "hasPreviousPage": false,
        "startCursor": "YXJyYXljb25uZWN0aW9uOjA=",
        "endCursor": "YXJyYXljb25uZWN0aW9uOjE="
      },
      "edges": [
        {
          "node": {
            "id": "RGVwdE5vZGU6MQ==",
            "name": "sell",
            "companyName": "bilibili"
          }
        },
        {
          "node": {
            "id": "RGVwdE5vZGU6Mg==",
            "name": "development",
            "companyName": "bilibili"
          }
        }
      ]
    }
  }
}
```

# Mutation
可以使用`SerializerMutation`[重用](https://docs.graphene-python.org/projects/django/en/latest/mutations/#django-rest-framework)Django Rest Framework 的 serializer。

创建一个`ModelSerializer`
```
class CompanySerializer(ModelSerializer):
    class Meta:
        model = Company
        fields = '__all__'
```

继承`SerializerMutation`
```
class MutateCompany(SerializerMutation):
    class Meta:
        serializer_class = CompanySerializer
```

定义Mutation
```
class Mutation:
    mutate_company = MutateCompany.Field()
```

创建Company
```
mutation {
  mutateCompany (input: {name: "test"})
  {
    id
    name
  }
}
```

返回值
```
{
  "data": {
    "mutateCompany": {
      "id": 19,
      "name": "test"
    }
  }
}
```

修改Company
```
mutation {
  mutateCompany (input: {id:19 ,name: "new name"})
  {
    id
    name
  }
}
```

返回值
```
{
  "data": {
    "mutateCompany": {
      "id": 19,
      "name": "new name"
    }
  }
}
```
