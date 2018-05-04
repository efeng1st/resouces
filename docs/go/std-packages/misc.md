errors包
==================
该包实现了操作错误的功能。该包定义了一个errorString结构体类型，包含一个s错误字符串。

另外该包暴露一个New()函数，用于创建新的errorString类型。还暴露了一个Error方法。

```
// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```
