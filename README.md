# mymodule
Go Module 测试用的 Repo

# 问题记录

问题一：v0.0.4 到 v0.0.4-0 这次测试中，实际应该是v0.0.5-0，go mod 会为后续版本升级一个patch号，所以验证过程中发生了找不到版本的问题

```
go: github.com/op-y/mymodule@v0.0.4-0: reading github.com/op-y/mymodule/go.mod at revision v0.0.4-0: unknown revision v0.0.4-0
```

问题二：测试v1.0.0 版本中，发现找不到该版本，实际上已经push到github上了，看异常报错中通过proxy去验证module，怀疑是push后时间太短sumdb没有同步，所以就404了?

```
go: github.com/op-y/mymodule@v1.0.0/go.mod: verifying module: github.com/op-y/mymodule@v1.0.0/go.mod: reading https://goproxy.cn/sumdb/sum.golang.org/lookup/github.com/op-y/mymodule@v1.0.0: 404 Not Found
	server response: not found: github.com/op-y/mymodule@v1.0.0: invalid version: unknown revision v1.0.0
```

修改 GOPROXY=direct 结果直接Timeout了，原因你懂得，真是无语了。。。

```
go: github.com/op-y/mymodule@v1.0.0/go.mod: verifying module: github.com/op-y/mymodule@v1.0.0/go.mod: Get "https://sum.golang.org/lookup/github.com/op-y/mymodule@v1.0.0": dial tcp 216.58.200.49:443: i/o timeout
```

推一个v2.0.0试试
