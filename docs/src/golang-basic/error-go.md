---
title: go的标准化error处理
shortTitle: 2.如何优雅使用Go-error
description: go的标准化error处理
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-02-19
star: true
order: -9
---

## 1、建议这样写错误处理

```go
// 1
func autn() {
	var err error
	if err != nil {
		// handle err
	}
	//do stuff
}

// 2 
func a(r *http.Request) error {
	//err := r.ParseForm()
	//if err != nil {
	//	return err
	//}
	//return nil
	// 我们其实没有必要写这么多的代码，直接返回就可以了
	return r.ParseForm()
	// 这样用也可能造成递归的返回错误直到最上层也是原始的错误

	// 在处理错误的时候我们应该只做一个决定就是：记录日志或返回错误
	// 返回错误的时候尽量以 errors.New("包名：错误原因“) 的形式
}
```

## 2、怎么优化代码让其不再堆积

优化前：我们可以看到有4个地方都需要进行错误判断，我们可以想办法将所有的错误处理代码写到别的地方，进行调用即可。
```c
type Header struct {
	Key, Value string
}
type Status struct {
	Code   int
	Reason string
}

func WriteResponse(w io.Writer, status Status, headers []Header, body io.Reader) error {
	_, err := fmt.Fprintf(w, "%d %s\r\n", status.Code, status.Reason)
	if err != nil {
		return err // 1
	}
	for _, h := range headers {
		_, err := fmt.Fprintf(w, "%s:%s\r\n", h.Key, h.Value)
		if err != nil {
			return err // 2
		}
	}
	_, err = fmt.Fprint(w, "\r\n")
	if err != nil {
		return err // 3
	}
	_, err = io.Copy(w, body)
	return err // 4
}
```
优化后：

```c
type errWriter struct {
	w   io.Writer
	err error
}

// 把判断错误的一些细节放到errWriter里面去
func (ew *errWriter) Write(p []byte) (int, error) {
	if ew.err != nil {
		return 0, ew.err
	}
	var n, err = ew.w.Write(p)
	if err != nil {
		ew.err = err
	}
	return n, nil
}

func WriteResponse1(w io.Writer, status Status, headers []Header, body io.Reader) error {
	ew := &errWriter{w: w}
	fmt.Fprintf(ew, "%d %s\r\n", status.Code, status.Reason)
	for _, h := range headers {
		fmt.Fprintf(ew, "%s:%s\r\n", h.Key, h.Value)
	}
	fmt.Fprint(ew, "\r\n")
	io.Copy(ew, body)
	return ew.err
}
```

## 总结
在 Go 语言中，`error` 类型是一个内建的接口类型，它通常用来表示函数执行过程中发生的错误。与许多编程语言中的异常处理机制不同，Go 的错误处理采用显式返回的方式，要求开发者在调用函数时检查错误。

优点：

1. **简单明确**：
   - Go 语言没有复杂的异常处理机制（如 Java、Python 的 try-catch），而是采用简单的返回值方式，调用者明确地检查每个函数的返回值。
   - 这种方式避免了程序控制流被异常机制打断，代码更加简洁清晰。

2. **避免异常隐式传播**：
   - 在许多语言中，异常机制会导致错误隐式地向上传递，直到被捕获。这可能会导致错误被忽略或者不容易追踪。Go 的错误处理机制要求每个调用者显式处理错误，从而避免了这种情况。

3. **无性能开销**：
   - Go 语言的错误处理没有异常机制的额外性能开销（如异常堆栈的捕获和传递）。这对于高性能要求的应用程序来说，是一个重要的优点。

4. **明确的错误上下文**：
   - 由于每个函数调用都需要返回错误，开发者通常会在错误对象中附带详细的上下文信息，帮助调试和追踪问题。

5. **一致性**：
   - Go 中几乎所有的函数在遇到问题时都会返回 `error` 类型，而不是通过其他方式（如返回特殊值或状态码）。这种一致性让开发者能够用统一的方式处理不同的错误。

缺点：

1. **冗长的代码**：
   - Go 中的错误处理要求每次函数调用都必须显式地检查错误。这可能导致代码冗长，特别是在函数调用链较长的情况下。
   - 错误处理的代码可能会影响主业务逻辑的可读性。

2. **忽略错误的风险**：
   - 如果开发者不小心忽略了错误处理，程序可能会出现难以察觉的 bug。Go 语言中虽然编译器不会直接阻止未检查错误，但这仍然是一个潜在的风险。

3. **错误类型缺乏灵活性**：
   - Go 的错误处理机制通常通过 `error` 接口返回一个简单的错误信息（字符串）。虽然可以自定义错误类型，但这可能使得错误处理变得更加复杂，并且错误的层级结构不像异常那样自然。

4. **没有堆栈跟踪**：
   - Go 的错误类型本身没有内建的堆栈跟踪信息（除非开发者显式地在错误中添加这些信息）。这使得调试某些类型的错误（特别是复杂的运行时错误）时可能变得更加困难。

5. **错误传播较为繁琐**：
   - 错误需要逐层向上传播，并且每一层都需要显式地处理和返回错误。对于一些错误，开发者可能只需要进行简单的包装或者日志记录，而 Go 的错误机制要求每一层都返回错误，这使得处理过程显得冗长。

