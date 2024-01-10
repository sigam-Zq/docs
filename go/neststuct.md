# go 结构体嵌套使用



## 使用结构体嵌套实现动态前端的字段显示


```GO
import (
	"encoding/json"
	"fmt"
)

type Abb struct {
	Name  string
	Phone string
}

type Baa struct {
	*Abb
	Name string
}

type Cdd struct {
	Name  string
	Phone string
}

type Dcc struct {
	Name string
	Cdd
}

func main() {

	// 直接赋值
	a := &Baa{Name: "baa"}

	printJson(a)
	//byteObj : {
	//         "Name": "baa"
	// }
	a.Abb = &Abb{Name: "abb", Phone: "131"}
	printJson(a)
	// byteObj : {
	// 	"Phone": "131",
	// 	"Name": "baa"
	// }

	fmt.Println("111111111111111111111111111111111111111111111111")

	d := Dcc{Name: "dcc"}

	printJson(d)
	// 	byteObj : {
	//         "Name": "dcc",
	//         "Phone": ""
	// }
	d.Cdd = Cdd{Name: "cdd", Phone: "131"}
	printJson(d)
	// byteObj : {
	// 	"Name": "dcc",
	// 	"Phone": "131"
	// }

	// 嵌套结构体的指针实现动态结构体效果（json tag 解析）

}

func printJson(obj any) {
	byteObj, err := json.MarshalIndent(obj, "", "\t")
	if err != nil {
		fmt.Printf("\033[31m printJson  %s  \033[0m\n ", err)
	}
	fmt.Printf("byteObj : %s \n", byteObj)
}


```

如上示例  使用指针的结构体嵌套 go 这种静态的语言也可以实现动态的结构体效果

* 但是这里总体是依赖于 json 解析tag 的过程中隐性实现的（效果可以对照到给前端返回的结构体存在 和不存在）


* 尝试- 如果给指针类型的结构体加上tag 会显示为 nil 的结构体对象

```go


type Baa struct {
	*Abb `json:"abb_a"`
	Name string
}

// console
//  {
//         "abb_a": null,
//         "Name": "baa"
// }
```


*  出现一个结构体嵌套赋值的错误
<p>
如下
