官网： https://golang.google.cn/


## Go的文件夹和package
 按照如下文件架结构示例：  
 
  ![image](https://user-images.githubusercontent.com/42050086/137091722-5e5181e6-5bc0-4dbd-ba6a-f3cc19c886b4.png)


 包名(package名称)和目录名(文件所在的一级目录名称)之间没有必然关系，import后面跟的是目录名称，如果在import的时候不指定别名，go自动使用package语句后面的名称定义为pacakge名称
  
```  
  The PackageName is used in qualified identifiers to access exported identifiers of the package within the importing source file. It is declared in the file block. If the PackageName is omitted, it defaults to the identifier specified in the package clause of the imported package.
  
```
    
 a. 包名可以和目录名称不一样  
 b. 一个目录下不可以有多个package名称,否则会报错  
 ![image](https://user-images.githubusercontent.com/42050086/137093573-d8e3b89c-f2c7-4f61-bb25-62086b3bdfdf.png)  
 c. 不同目录下可以有相同的package
 d. import语句import的时候特殊用法
    
 ```
      1. 正常import  
      import (
        	"fmt"
	        "test001/greetings"
	        "test001/network"
      )
    
     
      2. 为package取别名,用aliasF.Printf调用 pacakge fmt
     import (
          aliasF  "fmt"
	        "test001/greetings"
	        "test001/network"
      )
      
      
     3. "." import, 直接调用Printf，不用加package名称
     import (
          . "fmt"
	        "test001/greetings"
	        "test001/network"
      )
      
     4. "_" import, 不能调用fmt中的函数，只执行fmt中的init方法
     import (
          _ "fmt"
	        "test001/greetings"
	        "test001/network"
      )
 ```


## Go 环境变量  
  ### 获取环境变量   
   
    `go env`
  
  ### GO111MODULE 变量  
    
  Go 配置寻找package的方式，通过命令 `go env -w GO111MODULE=on|off` 进行设置
  
  传统方式 GO111MODULE=off
  
  go会一次在GOPATH,GOROOT中寻找依赖。假设项目目录为： 
      
   ![image](https://user-images.githubusercontent.com/42050086/137087082-d3ebd6e8-3252-4876-af9c-5cb5dc658943.png)

   greetings.go中定义了自定义方法并export未greetings package, main.go中引用package greetings中的方法。要确保引用成功有两种方法
      
   1. 确保 src目录在 GOPATH中。 这样在go build的时候会一次在GOPATH, GOROOT中寻找依赖并成功找到package。 
      
    import package写为package名称  
       ```
          import (
	            "fmt"
	            "greetings"
            )
       ```
       
   ![image](https://user-images.githubusercontent.com/42050086/137087391-01e2da96-4a10-4b72-a7aa-1275acba730a.png)

   2. 如果src不在GOPATH中，引用自定义包或者其他包需要采用相对路径的方式，以上面目录举例，
        
      import package可以写为如下也能成功 go build  
         ```
          import (
	            "fmt"
	            "../greetings"
            )
         ```
         
         
  Go mod方式 GO111MODULE=on

  Go mod方式不要求src文件是否在GOPATH目录中，取而代之是通过在`src`目录下的go.mod文件定义依赖
  对于任意一个go项目，需要以下步骤  
  1. 在src目录下定义project的pacakge
      ```
         go mod init test001
      ```
   生成文件定义了当前project export的pacakge以及对外界的依赖
   
   ![image](https://user-images.githubusercontent.com/42050086/137088549-35a933bb-b0f7-478b-ba1c-27c4262ab563.png)

   2. 对于project内部不同package之间的依赖需要从project的`package开始依赖`
   如上例子表示，project层面的pacakge为test001,那么main依赖greetings需要import为
        ```
          import (
	            "fmt"
	            "test001/greetings"
            )
         ```
    否则会报错 ` cannot find module for path _/Users/tao/work/golang_learning/src/greetings`
         
         
      
