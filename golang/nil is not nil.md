# nil != nil

## 第一印象
***这怎么可能。***

## 实现
```
func foo() error {
	var err *os.PathError = nil
	return err
}

func main() {
	err := foo()
	fmt.Println(err)  // <nil>
	fmt.Println(err == nil)  // false
}
```
***还真实现了***

## 解答
***interface被两个元素value和type所表示。只有在value和type同时为nil的时候，判断才会为true。而此时的err类型是\*os.PathError***

## 再次思考
```
func main() {
	a := (*interface{})(nil)
	// *interface {} <nil>
	fmt.Println(reflect.TypeOf(a), reflect.ValueOf(a))
	var b interface{} = (*interface{})(nil)
	// *interface {} <nil>
	fmt.Println(reflect.TypeOf(b), reflect.ValueOf(b))
	// true false
	fmt.Println(a == nil, b == nil)
}
```
***为什么类型值都相同结果却不同***

## 解答
***a等同为var a \*interface{} = nil***
***b的值为nil,类型是\*interface{}***