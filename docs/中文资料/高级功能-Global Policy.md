# 什么时候需要全局角色

对于简单的应用，使用RBAC（Role-based access control），我们通常把角色和授权都放在SPEEDLE的一个service里。

然而在某些情况下，也许是一个由许多相对独立的子系统组成的复杂系统，也许你有很多个独立的应用但共享同一套身份系统，每个子系统或者应用都有自己的SPEEDLE service。这样你就会有很多个service，每个service里的角色都是独立的。同一个用户在一个service里的角色不会在另一个service里生效。如果有一些角色策略对所有service都是一样的，而你不想重复的去在每个service里定义它们，这时候你就可以把它们定义为全局角色。

# 定义全局角色策略

1. 首先需要创建一个名字为'global'的service：
```
spctl create service global
```

2. 然后就可以在'global' service里创建角色了：
```
spctl create rolepolicy -c "grant user Emma AdminRole" --service-name=global
```

# 使用全局角色策略

1. 在其它的service里定义角色或者授权时，可以引用全局角色：
```
spctl create policy -c "grant role AdminRole borrow books" --service-name=library
```

2. 运行时全局角色会生效：
```
curl -X POST  http://localhost:6734/authz-check/v1/is-allowed \
-d @- << EOF
{
 "subject": {"principals":[{"type":"user", "name":"Emma"}]},
 "serviceName": "library",
 "resource":"books",
 "action": "borrow"
}
EOF
```

将会返回allowed = true。因为Emma有全局角色AdminRole，而在service ‘library’里，我们给予了AdminRole这个权限。

# 对Global Service的更多说明
1. 它的名字必须是'global'。
2. 用户能够根据需要创建或者删除它，像其它的service。
3. 在它里面只能创建角色，不能创建授权。
4. 虽然似乎没有什么用处，用户也能对它调用ADS API。
