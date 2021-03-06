---
date: 2018-10-10T00:11:02+01:00
title: 应用
weight: 20
tags: ["cli", "web", "application"]
---

## 网络应用

### 主要功能列表

* 网络应用 MVC (Model-View-Controller).
* 自动配置, 事先配置好的依赖可以注入到任何你指定的构造函数参数中或者结构体变量
* 依赖注入， 使用标签 \`inject:""\` 后构造函数.

### Go语言实现依赖注入

Dependency injection is a concept valid for any programming language. The general concept behind dependency injection is
called Inversion of Control. According to this concept a struct should not configure its dependencies statically but
should be configured from the outside.

Dependency Injection design pattern allows us to remove the hard-coded dependencies and make our application loosely
coupled, extendable and maintainable.

A Go struct has a dependency on another struct, if it uses an instance of this struct. We call this a struct dependency.
For example, a struct which accesses a user controller has a dependency on user service struct.

Ideally Go struct should be as independent as possible from other Go struct. This increases the possibility of reusing
these struct and to be able to test them independently from other struct.

The following example shows a struct which has no hard dependencies.

```go

package main

import (
	"github.com/hidevopsio/hiboot/pkg/app/web"
	"github.com/hidevopsio/hiboot/pkg/model"
	"github.com/hidevopsio/hiboot/pkg/starter/jwt"
	"time"
)

// This example shows that jwtToken is injected through method Init,
// once you imported "github.com/hidevopsio/hiboot/pkg/starter/jwt",
// jwtToken jwt.Token will be injectable.
func main() {
	// the web application entry
	web.NewApplication().Run()
}

// PATH: /login
type loginController struct {
	web.Controller

	token jwt.Token
}

type userRequest struct {
	// embedded field model.RequestBody mark that userRequest is request body
	model.RequestBody
	Username string `json:"username" validate:"required"`
	Password string `json:"password" validate:"required"`
}

func init() {
	// Register Rest Controller through constructor newLoginController
	web.RestController(newLoginController)
}

// newLoginController inject jwtToken through the argument jwtToken jwt.Token on constructor
// the dependency jwtToken is auto configured in jwt starter, see https://github.com/hidevopsio/hiboot/tree/master/pkg/starter/jwt
func newLoginController(token jwt.Token) *loginController {
	return &loginController{
		token: token,
	}
}

// Post /
// The first word of method is the http method POST, the rest is the context mapping
func (c *loginController) Post(request *userRequest) (response model.Response, err error) {
	jwtToken, _ := c.token.Generate(jwt.Map{
		"username": request.Username,
		"password": request.Password,
	}, 30, time.Minute)

	response = new(model.BaseResponse)
	response.SetData(jwtToken)

	return
}

```

## 命令行应用

### 功能列表

* Dependency Injection
* Struct Tag for dependency injection
* Sub command handler