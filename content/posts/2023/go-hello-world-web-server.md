---
title: "Creating a Hello World web application and form processing in Go using a Dev Container"
date: 2023-03-12T13:41:00Z
tags: ['go', 'hello world', 'dev containers']
icon: brand-golang
aliases:
  - /posts/go-hello-world-web-server/
summary: >-
    In this post, we're going to look through creating a simple web application in Go using a dev container, which will host a simple "Hello World" page, and then we can extend that with a page with submits a form.
---

## Introduction

In this post, we're going to look through creating a simple web application in Go using a dev container, which will host a simple "Hello World" page, and then we can extend that with a page with submits a form.

## Setting up the dev container

To get started here, we're going to use dev containers with vscode, so our first step is to create a new folder for our project, and then open that folder in vscode. We can then create a new dev container by running the "Add Dev Container Configuration Files..." command and using the "Go" template. This will create the `.devcontainer` folder for us along with the default setup which will cover our needs. We can then re-open the folder in the container by running the "Reopen in Container" command in vscode.

## Setting up the project

As our project is so simple, we can actually get away with using a single file for the application. So let's start by creating a new file called `main.go` in the root of our project. We can then add the following code to that file:

```go
package main

import (
    "html/template"
    "log"
    "net/http"
)

type IndexData struct {
    PageTitle  string
    QueryParam string
}

func main() {
    http.HandleFunc("/", indexHandler)
    log.Println("Starting on :8080")
    http.ListenAndServe(":8080", nil)
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
    tmpl, err := template.ParseFiles("templates/index.html")

    if err != nil {
        log.Println(err)
        return
    }

    query := r.URL.Query()
    QueryParam := query.Get("QueryParam")

    pageData := IndexData{
        PageTitle:  "Hello World",
        QueryParam: QueryParam,
    }

    tmpl.Execute(w, pageData)
}
```

This will create a simple web application which will server a simple page. This is made up of several parts:

### `package` and `import` statements

The `package` statement here defines the package which this code will live inside. This is similar to the concept of namespaces in other languages. In this case, we're using the `main` package, which is the default package for any executable code.

The `import` statements then load in additional packages which we'll be using in our code. For this little project, we only need a few things so far:

* `html/template` - This is the package which we'll use to render our HTML templates, which can take an HTML file with some placeholders, which we're then able to fill using our own data dynamically
* `log` - This is the package which we'll use to log any errors which occur
* `net/http` - This is the package which we'll use to create our web server

### `type` definitions

After this, we have a `type` definition. This will define a type for use to use when passing data from our Go code to our template which will be written in HTML. In this case, we're defining a type called `IndexData` which will have two properties, `PageTitle` and `QueryParam`, both of which are `string` properties.

### `main` function

The `main` function is the entry point for our application. This is setup to add our handlers for requests, configure our logging, and start the web application on the port we'd like to use.

### `indexHandler` function

This function handles any requests which come through to our web application. This is broken up into a few parts:

* Load in the template using `template.ParseFiles`
* If there is an error with the template, log it and return
* Get the query parameters from the request
* Create an instance of our `IndexData` type, and populate it with the data we want to pass to our template
* Execute the template using our `IndexData` and return the response to the user as HTML

With this, we can load a simple HTML page, and also load in a query parameter from the URL that the user has requested. For example, if the user went to `http://localhost:8080?QueryParam=Hello`, then the `QueryParam` property in our `IndexData` type would be set to `Hello`, which can then be passed to our template and rendered on the page.

## Creating the template

Next, we'll need to create a template file to use to render the HTML for this page. For this, we can create a new folder called `templates` in the root of our project, and then create a new file called `index.html` in that folder. We can then add the following code to that file:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <p>{{.PageTitle}}</p>
        <p>{{.QueryParam}}</p>
        <a href="/add">Adder</a>
    </body>
</html>
```

## Running the application

At this point, we're able to run our application and see the results. To do this, open up the terminal inside of your dev container, and then run the following command:

```bash
go run main.go
```

This will run the `main.go` file and start up the web application. You can then open up a browser and navigate to `http://localhost:8080` to see the results.

## Configuring `tasks.json` to run the application

As we're probably going to want to run this application a lot, now would be a good time to setup a `tasks.json` file to make this easier in vscode. We can do this by creating a new file called `.vscode/tasks.json` in the root of our project, and then adding the following code to that file:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Run main.go",
            "type": "shell",
            "command": "go run main.go",
            "group": {
                "kind": "test",
                "isDefault": true
            }
        }
    ]
}
```

This will mean that if we run the command "Run Test Task" in vscode, it will open a new terminal and run the `go run main.go` command for us.

## Adding a template with a `form`

Now that we can render an HTML file and use a query parameter, let's add a new page which will allow us to submit a form. To do this, let's first setup our new template file. This will be similar to the `index.html` file, but we'll add a form to it. We can create a new file called `add.html` in the `templates` folder, and then add the following code to that file:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <form action="/add" method="post">
            <input type="number" name="value1" value="{{.Value1}}" /><br />
            <input type="number" name="value2" value="{{.Value2}}" /><br />
            <p>
                <span>{{.Result}}</span>
            </p>
            <input type="submit" value="Submit" />
        </form>
    </body>
</html>
```

*Note: the `action` in the `form` element denotes where the form will be posted to when the `submit` button is pressed*

In this template, we're using three values from a model:

* `Value1` - This is the first value which the user will enter, and it will be bound to the first `input` element. Note that the `name` of this element is `value1`, which will be used to identify the value when the form is submitted
* `Value2` - This is the second value which the user will enter, and will work like `Value1`
* `Result` - This is the result of the addition of the two values, and will be displayed to the user

## Adding a handler for the new page

Next, we can add a new handler for our new page. We can do this by returning to our `main.go` file and adding a few extra parts to it.

### Adding a new `type` definition

First, we'll add a new `type` definition to handle this page. We can do this by adding the following code to the file:

```go
type AddData struct {
    Value1 int
    Value2 int
    Result int
}
```

This will define a new type to use as a model for this page. It containers three properties, all of which are integers, `Value1`, `Value2`, and `Result`.

### Creating our handler

Next, we can add a new handler for this page. We can do this by adding the following code to the file:

```go
func addHandler(w http.ResponseWriter, r *http.Request) {
    tmpl, err := template.ParseFiles("templates/add.html")

    if err != nil {
        log.Println(err)
        return
    }

    rawVal1 := r.FormValue("value1")
    rawVal2 := r.FormValue("value2")

    val1, err1 := strconv.Atoi(rawVal1)
    val2, err2 := strconv.Atoi(rawVal2)

    if err1 != nil || err2 != nil {
        val1 = 0
        val2 = 0
    }

    result := val1 + val2

    log.Printf("Adding %d and %d", val1, val2)

    pageData := AddData{
        Value1: val1,
        Value2: val2,
        Result: result,
    }

    tmpl.Execute(w, pageData)
}
```

This works in a similar manner to the `indexHandler` function, but there are a few differences:

* Rather than loading in the `index.html` template, we're loading in the `add.html` template
* We're getting the values from the form using `r.FormValue` rather than `r.URL.Query`, this is because we want to look at the posted form rather than the query parameters
* We're converting the values from strings to integers using `strconv.Atoi`. Values which come from the form are always strings, so we need to convert them to the correct type before we can use them
* Once we have those values as integers, we can add them together and store the result in the `Result` property of our `AddData` type

### Setting up the handler to be called

Finally, we can add our new handle to the `main` function by adding the following code to the function:

```go
http.HandleFunc("/add", addHandler)
```

*Note: this should go above the `indexHandler` function, as this takes priority*

At this point, we should be able to re-run our application and go to the `/add` page. We should see a form which allows us to enter two numbers, and then displays the result of adding them together. When the page is returned, the values which we entered will be displayed in the form, so we can easily change them and see the result change.

## Code

You can find the code for this post on GitHub: <https://github.com/cwrenhold/go-web-hello-world>
