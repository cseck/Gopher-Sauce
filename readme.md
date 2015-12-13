# Gopher Sauce Server Generation
![Implies that Gopher sauce uses Go to write go programs.](https://lh3.googleusercontent.com/-vnw4NhuJKQY/VmuZgzodIPI/AAAAAAAAAAM/mbm7EfWjfsY/s0/xlarge.png "xlarge.png")

Gopher Sauce is is a command line tool used for compiling GoS projects. A GoS project uses XML sources and Go lang packages to create a web server. It uses xml to build onto Go lang's [template](https://golang.org/pkg/html/template/)  processing functionality, making it easier to define custom objects,methods and structs that are accessible to your templates as well as any Go lang declarations within your GoS project.
Gopher Sauce makes web server application distribution easy by compiling your entire Gopher Sauce application's sources and misc. file resources into one binary. 

### How to install GoS
	go get -u github.com/cheikhshift/gos/...

### Handle GoS Dependencies
The application generated by Gopher Sauce has certain dependencies. Please run the command below to download them before attempting to run or export your application. (Assuming $GOPATH is set)

	$GOPATH/bin/gos dependencies

# Table of Content

 1. [GoS XML template](#gos-xml-template)
 2. [GoS project layout](#gos-project-layout)
 3. [Core Usage](#core-usage) 
	 1. [Using GoS command without parameters](#using-gos-command-without-parameters)
	 2. [GoS running application](#gos-running-application)
	 3. [GoS export application](#gos-export-application)
 4. [Configuring GoS xml file](#configuring-gos-xml-file)
	1. [Deploy](#deploy)
	2. [Port](#port)
	3. [Output](#output)
	4. [Key](#key)
	5. [Init](#init)
	6. [Main](#main)
	7. [Var](#variables)
	8. [Header](#header)
		1. [Structs](#structs)
		2. [Objects](#objects)
9. [Methods](#methods)
10. [Timers](#timers)
11. [Templates](#templates)
12. [API Generation](#api-generation)
13.  [Imports](#imports)
 5.  [GoS builtin template functions](#gos-builtin-template-functions)
 6. [GoS page](#gos-page)
 7. [Bug tracking](#bug-tracking)

# GoS XML template
The GoS XML template is where your application's imports, structs,objects,methods,timers,templates and end point hooks. Here is a fully documented project template [file](https://github.com/cheikhshift/Gopher-Sauce/blob/master/src/github.com/cheikhshift/gosapphire/server.xml)

 Below is an example of a very minimal GoS XML source : 
This Example will save a file named server_out.go to the package package in the gos cmd tool. It will create one struct called DemoChild in the native go manner.

	 <?xml version="1.0" encoding="UTF-8"?>
	<gos>
		<deploy>webapp</deploy>
		<port>8080</port>
		
		<output>server_out.go</output>
	
		<key>something-secretive-is-what-a-gorrilla-needs</key>
		<header> 
			<!-- remember to Jumpline when stating methods or different struct attributes, it is vital for our parser \n trick -->
		<struct name="DemoChild">
			SomeOtherAttr string
		</struct>
		</header>
	</gos>

# GoS project layout
This section covers the integration of GoS with your current Go configuration. GoS project minimally requires a GoS XML template, a web root folder and a template root folder.

 - Template folder is the root folder for all templates declared within your application
 - Web root can contain both usual server resource data and template pages as well. Template pages within the  web root folder have access to user sessions.
	 - *webroot/dist/ is initialized as a fileserver to correct issues when serving css files. It will not compile templates and handle api end points with the `/dist/` prefix.

Your GoS project should look like this
		
		$GOPATH/src/
			|-	your/package/name
							|-		gos_xml_template.xml
							|-		web_root/
										|-	index.html
									template_root/
										|-	email.tmpl 
	

# Core Usage
This section covers how to use the command line tool

## Using GoS command without parameters
It is possible to invoke gos directly without commands. Simply answer the questions and watch your application compile. The program will guide you through the process.

	$GOPATH/bin/gos

## GoS running application
To compile and run your GoS application invoke `gos` with the run parameter.
The example below will compile a package called `sample/test` with configuration file 	`server.xml`, a web root folder called `web` as well as a templates directory called  `tmpl` ,
		
*Command of process described
	 $GOPATH/bin/gos run sample/test server.xml web tmpl
	 
The command above should compile and execute your program. Invoking the `gos` with the run parameter will put go-bindata in -debug mode so that you can edit your server resources and templates while your application is in running.
## GoS export application
Exporting your application will wrap up all of your GoS project resources as one binary ready for distribution (Zips require too much work). To compile and export your GoS application invoke `gos` with the export parameter.
For example to export a package called `sample/test` with configuration file 	`server.xml`, a web root folder called `web` as well as a templates directory called  `tmpl`, the command below will be ran : 

	*Command of process described
	$GOPATH/bin/gos export sample/test server.xml web tmpl

Unlike running the GoS application, this build will have static copies of its resources at the time of compilation. The executable will be placed in your GoS project root (package root). This copy can be ran anywhere without the need of resources within your package.

# Configuring GoS XML file
This section covers documentation of GoS XML template tags

## Deploy
Application deploy specifies the manner that GoS should compile your application. Use `webapp` to have GoS generate a webserver for you. This tag should always be within the root of the `<gos/>` tag.
*This tag is required

	<gos>
		...
		<deploy>webapp</deploy>
		...
	</gos>
		
## Port
(if applicable) Specifies the port your application should listen on. 

	<gos>
		...
		<port>8080</port>
		...
	</gos>
## Output
The name of the file that GoS will save.
*This tag is required
	
	<gos>
		...
		<output>server_out.go</output>
		...
	</gos>
## Key

(if applicable) used as cookie store key within your web app

	<gos>
		...
		<key>my-key-braah</key>
		...
	</gos>
## Init
GoS will wrap the data within this tag into the usual Go `init()` function
For example :
		
		<gos>
			...
			<init> 
				fmt.Println("Logging from init function")	
			</init>
			...
		</gos>
Will translate into

	func init(){
		fmt.Println("Logging from init function")
	}
This can be useful for registering custom declared structs for Session use.

## Main
GoS will wrap the data within this tag into the go `main()` function.
For example : 

		<gos>
			...
			<main> 
				fmt.Println("Logging from main function")	
			</main>
			...
		</gos>
Will translate into

	func main(){
		...
		fmt.Println("Logging from Main function")
		...
	}


## Variables
To declare public variables that are accessible to your program use the `var` tag. Use the type parameter to specify the variable type. Use the `*` qualifier as appropriate.
For example 

		<var type="string">FreeServer</var>
Will become (within your Go program)

		var FreeServer string 
## Header
Think of the `<header/>` tag as the interface of your application. It holds structure and object declarations.
### Structs
The `<struct/>` tag will declare a new Go structure within within your generated Go src.
#### Attributes
name - The name of this structure
** Innerxml Struct parameter declarations.

The example below declares a Go struct named DemoGos with a sub structure called DemoChild. The struct DemoGos has attributes SomeAttr and Child. 

These structs are available everywhere within your application.

Whenever this struct is initialized within a template, these variables are accessible as that object's properties.

	<gos>
		...
		<header>
			<struct name="DemoChild">
				SomeOtherAttr string
			</struct>
			<struct name="DemoGos">
				SomeAttr string
				Child *DemoChild
			</struct>
		</header>
		...
	</gos>
### Objects
The `<object/>` tag declares a GoS object. It needs struct name to have a parameter map, which is used whenever this object is initialized within a template. Please keep in mind that the struct specified for this object must be returned all the time.
#### Attributes
	struct - the name of a struct available to the application
	name - name of object
The example below will declare an object called `myDemoObject`. The inner content of this tag is where object methods are declared. (remember to jumpline after each declaration!!!) 
These functions also need to be declared within your GoS application xml template or another imported xml template.
 

	<gos>
		...
		<header>
		<object struct="DemoGos" name="myDemoObject">
			Hackfmt(save string)
			WhatsMyAttr(save string,end string) string
			WhatsMyAttrLength() string
		</object>
		</header>
		...
	</gos>
By declaring an object :
 - GoS will create a Go lang `type` with the object's name linked to its respective struct
 - GoS will create a local and template accessible method to initialize this object. Local method :  `net_myDemoObject(args ...interface{}) (d DemoGos)`. Template method : {{$emptyObject :=  myDemoObject}}.

-  Invoking object with static parameters : `{{$object := myDemoObject /{`\`SomeAttr\`:\`ValueIn\`}/ }}`. To pass initialization variables we can use a static json escape holders `/{` and `}/`. To initialize your variable with other objects create a method that returns this object by using the desired input initialization objects to use.

- Each function specified within this tag, will be generated to accept this object as a mutable parameter within templates as well as your application's local scope. Local function generated :
 `	func (object DemoGos) Hackfmt(save string) `
 You can access this function via templates by invoking these calls within any template in this application : `{{$object | Hackfmt "data"}}` or `{{$error := $object | Hackfmt "data" }}`. The return type declared will be returned, so please make sure that type is returned within the linked method.

*If an Object and a template share a struct you can initialize the template and compile it to html with this call `{{b<template name> $object}}`


## Methods
Sorry for the confusion, Methods translate into usual go functions (`func`). 
This section covers how to create functions for your applications. These functions, known as Methods within GoS are linked for use with your templates, api end points, objects, timers and anywhere else within your application.

### Attributes

	name - Specifies the name of the function. Please keep in mind that usage of this function outside of templates requires the `net_` added to name because GoS appends that prefix to avoid any method redeclaration. If you desire to write methods for strict usage outside of template files we recommend writing a normal Go package and importing it with the `<import/>` tag.
	
	return - The return type of the function. Ie: string,bool or a custom struct `DemoGos` 
	var - This is a comma delimited string of the variables you wish this function to have. Say we need a function that has two parameters `param1 string, param2 string` it will be declared `param1,param2`. Before using the variable make sure its type is defined within the method. For example to log `param1` the call would be `fmt.Println(param1.(string))`
	
	autoface - GoS has the capability of taking methods declared within objects and look for the corresponding method name, and create a Go function to mutate the corresponding object with the matching method. This feature is enabled by default to disable it for a method and have it not converted as a method to mutate an object set the attribute to false.
	limit - This is a comma delimited string to specify the object types this function will work with, if linked. For example to limit a function to DemoGos and myDemoObject this attribute will be set to `DemoGos,myDemoObject` 
	object - If this method is linked to an object this specifies the local name of the object within the function. By default it is `object`
	**InnerXml - This contains your method declarations. GoS will wrap up the inner xml data with your method attributes to create a Go func

In the example below, one method will be created, the corresponding method will then be added as an object method as well.  Since input variables are declared within the object, we do not need to add the `var` attribute to our method declaration. 

		<gos>
			...
			<header>
			<object struct="DemoGos" name="myDemoObject">
				Hackfmt(save string)
			</object>
			</header>
			...
			<methods>
				<method name="Hackfmt">
				fmt.Println(save + " -> " + object.SomeAttr) 
				</method>
			</methods>
			...
		</gos>
		... Output by GoS :
		//Generated GoLang function
		func (object DemoGos) Hackfmt(save string)  
		//Generated template method for object
		func  net_Hackfmt(save string,object DemoGos) string 

Once a method is used within an interface it will not be generated again and can only be invoked with the linked object type.  The snipped below will use declared function `Hackfmt` within a template. The `net_` prefix is not needed within templates.
	
	{{$object := myDemoObject}}
	{{$object | Hackfmt "saveString" }}
	<!-- Another way to call it -->
	{{Hackfmt "saveString" $object }}

The method can also be used outside of templates with the snipped below :
	
	object := DemoGos{}
	object.Hackfmt("saveString")	

Another test case is declaring a method that is not used with any objects and is still accessible within templates and function calls outside of it as well. The variables set in the var tag need their type declared during use. The example below we will declare a function that sends emails and returns a `bool`.		

		<gos>
			...
				<methods>
					<method name="sendEmail" var="to,from" return="bool">
					    go fmt.Println("Send Email -> " + to.(string) + " ->" + from.(string))
						return true
				</method>
				...
			</methods>
			...
		</gos>
		... Output by GoS :  
		//Generated template method
		func net_sendEmail(args ...interface{}) bool

Now we will create a template class that will output a Bootstrap alert if an email is sent and an error if not. 
Note the usage of the Pipe call although "from" is not an object and send email is not an object method.

	{{ if "from" | sendEmail "to" }}
			    {{Bootstrap_alert /{`Strong`:`Success`,`Type`:`success`, `Text` : `Email Sent`}/ }}
	{{else}}
		       {{Bootstrap_alert /{`Strong`:`Error`,`Type`:`danger`, `Text` : `Please Login`}/ }}
	{{end}}
	
Do not let the `|` scare you (known as pipe),  in our case the function parameter `"from"` becomes the last parameter of the function call and ones following the method name the first parameters, ordered in the way they are inputted. hence : `LastParam | functionName param1 param2... `.
Another way to invoke your function is  :
	
		{{if sendEmail "to" "from" }}
			  {{Bootstrap_alert /{`Strong`:`Success`,`Type`:`success`, `Text` : `Email Sent`}/ }}
		{{end}}

GoS has built in templates as well, [check them out here](#builtin-template-functions)

That Easy!!!!!!
## Timers
This section covers how to create methods that are called after every certain unit of time intervals. GoS uses the `<timer/>` tag to generate timer execution block with the target method specified.


### Attributes
	
	name - The name of the timer. Variable only accessible within `main()` function call
	method - The name of the method declared in GoS, that is ran after each interval. Please keep in mind that a timer method cannot have a return call within the method and variable declarations. Only the name will be used, any other attribute of the method name declared will be ignored. Your method will also not be available to your application. For example to use a method named `Hackfmt` this attribute would just be that methods name (Hackfmt). 
	interval - The number multiplied by the Go lang `time` units to determine the intervals of execution
	unit - This is the Go lang time unit constant used to determine how often your timer will execute

The following example will link Hackfmt to a timer that will be executed every 60 seconds

	<gos>
		...
		<timers>
		<timer method="Hackfmt" interval="60" unit="Second" name="PublicName"></timer>
	</timers>
		...
	</gos>
	Generated output :
	...
	main(){
	...
	PublicName := time.NewTicker(time.Second *  60)
					    go func() {
					        for _ = range PublicName.C {
					           // -> Hackfmt()
					        }
					    }()
	...
	}

## Templates
This section covers how to declare templates outside of your web root. This allows you to create template files that are not served within your server. This allows for the creation of template classes, that for example send emails and returns a success alert as feedback in the form of HTML. 
This allows for easy html template class generation, and binary compilation of file to ensure security of code and file  invisibility. 

### Attributes
	
	name - This is the name of the template within your application. You can load a template by simply calling `{{<template name>}}`
	
	tmpl - This specifies the path to the template file with the declared template root in mind. For example if our file was in `PACKAGENAME/tmpl/file.tmpl` the tmpl attribute will be `file` because GoS will append the needed prefixes to find and compile the template file. 
	
	struct - This specifies a declared `struct` to be used as the template's parameter map. 

In the example below, a struct will be declared for an alert. The example will initialize the template in three ways: by initializing the struct and then building it, initializing it with static Json parameters and without any parameters at all. Remember within any template file in your project you may load the template by typing its name. 

The examples below will declare a struct and use that struct's parameters within the template. It is possible to build a template from static JSON init and even via object initialization, mutating and then compiling it to `HTML code` 

The example is using a template file with path 	`bootstrap/alert`.
*Due to the canonical nature of our asset to binary tools, file paths must be in lower case.

Server.xml
	
	<gos>
		...
		<header>
			<struct name="Bootstrap_alert">
					Strong string
					Text string
					Type string
			</struct> 
			...
		</header>
		...
		<templates>
			...
			 <template name="Bootstrap_alert" tmpl="bootstrap/alert" struct="Bootstrap_alert" /> 
		</templates>
	</gos>

tmpl/bootstrap/alert.tmpl :

		
	 <div class="alert alert-{{.Type}} alert-dismissible fade in" role="alert">
	<strong>{{.Strong}}</strong> <p>{{.Text}}</p>
	</div>

Now to use the template within other templates there are three ways of doing this. (With the example above in mind)
				
		<!-- No parameters -->
		{{Bootstrap_alert}}
		<!-- Static JSON -->
		{{Boostrap_alert /{`Type`:`danger`,`Strong`: `Strong`,`Text` :`Test`  }/ }}
		<!--Struct object Init, Static Json parameters can be supplied as well!! Add  `c` to your template name to initialize its struct; And add `b` to your template name to output the html -->
		{{ $mutable := cBootstrap_alert }}
		{{ $mutable | dbOperation "queryvalue"  }}
		{{bBootstrap_alert}}
		

## API Generation
This section covers how to create http request hooks within your GoS application. If a request's method and path match a declared endpoint, the method declared as the matching end point in the GoS project will be ran. Linking a method to your web server's api will prevent it from being used anywhere else within your application. Similar to a template within your web root, you can access session variables within api endpoint linked methods with the variable `session`.

### Attributes

	path - the path to the api hook within your server
	method - The GoS declared method ran when the Hook is called.
	type - This specifies the request method ie: POST,GET,PUT,DELETE

The example below will declare a method and api end point within the GoS configuration called login. The hook defined will create a rest call : `POST /index/api`

	<gos>
		...
			<methods>
				<method name="login" > 
					//login function
					response = mResponse(Button{Color:"#fff"})
					fmt.Println("Login!! -> " + session.Values["username"].(string))
			</method>
			...
		</methods>
		...
		<endpoints>
			   <end path="/index/api" method="login" type="POST" ></end>
		</endpoints>
		...
	</gos>
	This condition will be added as a check prior to the server attempting to find a file matching the request path within the web root : 
	
	if  r.URL.Path == "/index/api" && r.Method == strings.ToUpper("POST") {
		response = mResponse(Button{Color:"#fff"})
		fmt.Println("Login!! -> " + session.Values["username"].(string))
	}
	

Here are the variables that are available to your api method code block :

	response - String response of api.
	session - Current Gorilla session of the request (if applicable). Use the session.Values dictionary to access and save data.
	Handy functions :
	mResponse - will convert any object into a JSON string for output. The example below converts Button into a JSON string for output.
	

	
## Imports
Imports allow you to import other Go lang packages for use within your GoS XML template. It also allows you to import other GoS xml template by specifying an XML file relative to `$GOPATH/src`. This will include that files imports wether template files or Go lang packages, structs,objects,methods,timers,templates and api endpoints. 

This tag belongs only in the root of the `<gos/>` tag
Example import :
		
		<import src="mongo.xml"/>
		<!-- Importing Golang Packages -->
		<import src= "gopkg.in/mgo.v2"/>
		<import src="gopkg.in/mgo.v2/bson"/>

To expedite the process of using external packages within your application you can you set the `fetch` attribute to true.
The example below will download the package using the `go get` command during the generation of your application.

	<import src= "gopkg.in/mgo.v2" fetch="true"/>

# Builtin template functions
This section covers the list of functions available within all your templates compiled using GoS.
Please keep in mind that the .Session and .R variable are only available to template files within your server web root.

 - js - will take its only input and add it as the src attribute of the html `<script/>` tag `<script src="var"></script>`
	 - Usage : `{{js  "dist/js/bootstrap.js"}}`
 - css - Will take its only input and add it as the href attribute of the html  `<link/>` tag.
	 - Usage : `{{css "dist/css/bootstrap.css" }} `
 - sd - Will delete the current page session
	 - Usage : `{{.Session | sd }}` 
 - sr - Will remove a specified session key
	 - Usage : `{{.Session | sr "KeyName" }}` 
 - sc - Will check to see if a session key exists
	 - Usage : `{{.Session | sc "KeyName" }}`
 - ss - Will set a string value as a session variable.
	 - Usage : `{{.Session | ss "KeyName" "Variable" }}` 
 - sso - Will set a struct as a session variable. 
	 - Usage : This requires three steps to work.
		 -   Import `encoding/gob` with the import tag. Do not set the fetch attribute.
		 - Register the struct types within your `<init/>` tag of your GoS project root. 
					 
					 init(){ 
					 ...
					 gob.Register(&Object{})
					 
		
		- Once the steps above are completed you can now set the linked object as session variables : `{{.Session | sso "KeyName" $object_with_Object_struct }}`
 - sgo - Will retrieve a a session stored object
	 - Usage : `{{$desiredObject =  .Session | sgo "keyName" }}` 
 - sg - Will retrieve a string stored as a session variable.
	 - Usage : `{{$string := .Session | sg "KeyName" }}` 
 - form - Will retrieve a request variable no matter how it is submitted.
	- Usage : `{{ $input = .R | }}` .  `.R` is a page variable with type `http.Request` from the Go lang package `net/http`
 - eq - Will compare two variables and return a `bool` of value true if they are equal
	 - Usage : `{{if eq "Obj1" "Obj1" }} {{end}}` 
 - neq - Will compare two variables and return a `bool` of value true if they are not equal.
	 - Usage :  `{{if neq "Obj1" "Obj2" }} {{end}}` 
 - lte - Will see if the first number declared is less than or equal to than the second number declared, if this statement proves to be true it will return a `bool` with the value true.
	 - Usage : `{{if lte 5 10 }} {{end}}` 
 - lt - Will see if the first number declared is less than the second number declared, if this statement proves to be true it wil return a `bool` with value true.
	 - Usage  : `{{if lt 5  7 }} {{end}}`
 - gte - Will see if the first number declared is greater than or equal to the second number declared, if this statement proves to be true it will return a `bool` with value true
	 - Usage : `{{if gte 5 2}} {{end}}` 
 - gt -  Will see if the first number declared is greater than or equal to the second number declared, if this statement proves to be true it will return a `bool` with value true
	 - Usage : `{{if gt 5 2}} {{end}}` 

# Gos page
This section covers the variables accessible to a template page in your GoS project's web root.

More information on [sessions.Session](http://www.gorillatoolkit.org/pkg/sessions)

The Go lang struct for a page in your web root : 

		 type Page struct {
					    Title string
					    Body  []byte
					    request *http.Request
					    isResource bool
					    R *http.Request
					    Session *sessions.Session
			}

These variables are accessible to template web pages within your project's web root. For templates in your template class folder, GoS will use the explicitly defined struct for that template.

# Bug tracking

Github Issue tracker!!!!