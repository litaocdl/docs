官网： https://golang.google.cn/




Tips：

# go 环境变量
  go env
  
# GO111MODULE

Go 寻找package依赖的时候依赖变量GO111MODULE

不使用go mod模式
go env -w GO111MODULE=off 依次在GOPATH,GOROOT中寻找

使用go mod模式
go env -w GO111MODULE=on 在GOROOT路径中寻找，
