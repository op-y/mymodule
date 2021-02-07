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

修改 `GOPROXY=direct` 结果直接 **Timeout** 了，原因你懂得，真是无语了。。。

```
go: github.com/op-y/mymodule@v1.0.0/go.mod: verifying module: github.com/op-y/mymodule@v1.0.0/go.mod: Get "https://sum.golang.org/lookup/github.com/op-y/mymodule@v1.0.0": dial tcp 216.58.200.49:443: i/o timeout
```

推一个v2.0.0试试，也不行，过了个把小时，`go clean -modcache`，删除测试项目的go.mod和go.sum，再试，好了。。。目前尚不清楚是什么原因导致之前的失败。

问题三：添加v2.0.0版本后，根据规范再module的名字加上了v2即 `github.com/op-y/mymodule/v2` 但是测试项目中没有修改所以会报错

```
go: errors parsing go.mod:
/Users/yezhiqin/gopath/src/mymoduletest/go.mod:5: require github.com/op-y/mymodule: version "v2.0.0" invalid: module contains a go.mod file, so major version must be compatible: should be v0 or v1, not v2
```

测试项目go.mod 中修改成 `github.com/op-y/mymodule/v2 v2.0.0` 后不报错了，但是程序使用的是v1.0.0，go.mod 变为如下

```
module mymoduletest

go 1.15

require (
	github.com/op-y/mymodule v1.0.0 // indirect
	github.com/op-y/mymodule/v2 v2.0.0
)
```

后来发现根据规范，如果要使用大于v1的版本，代码中import需要修改成 `import "github.com/op-y/mymodule/v2"`，此后才会正常使用v2.0.0 版本，后续删除 `github.com/op-y/mymodule v1.0.0 // indirect` 也能正常运行。

问题四：就是上边已经出现的 `// indirect` 这货，测试代码非常简单就是直接引用mymodule中的Print函数，不存在间接引用的。据说这个是Go Modules的BUG，据说而已。 
