# map 操作


* map 使用 delete 删除不存在的元素会不会报错？

```go

func main() {

	aMap := make(map[string]struct{})

	aMap["1"] = struct{}{}

	delete(aMap, "2")

	println(aMap)

}


```

> 控制台打印

```bash

$ ls
main.go
(base) 
zq102@MSI MINGW64 /d/desktop/gitDemo/go/CodeExample/mapDel
$ go build -o run main.go 
(base) 
zq102@MSI MINGW64 /d/desktop/gitDemo/go/CodeExample/mapDel
$ ./run 
0xc000059e70

```