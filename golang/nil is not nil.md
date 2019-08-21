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
	fmt.Println(reflect.TypeOf(a), reflect.ValueOf(a))
	// *interface {} <nil>
	var b interface{} = (*interface{})(nil)
	fmt.Println(reflect.TypeOf(b), reflect.ValueOf(b))
	// *interface {} <nil>
	
	fmt.Println(a == b)
	// true
	fmt.Println(a == nil, b == nil)
	// true false
}
```
***a＝＝b 用的是他们的真实值，a的值就是nil，而变量b的值其实有两部分，一部分是dynamic type 一部分是dynamic value 它的value部分是nil但是type不是nil***