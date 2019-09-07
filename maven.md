
maven global repository http://repo1.maven.org/maven2/

## Maven 构建顺序

   compile -> test --> package --> install.
   后一个会自动包含前一个命令
   
## Maven坐标

   groupId, artifactId, version, packaging (default `jar`), classifier(附属构件)
   
   构件文件名： `artifactId-version[-classifier].packaging`  
   
 ## Dependency
 
   ### scope: mvn 有在构件的时候会用到三个classpath, 编译classpath,测试classpath和运行classpath. scope就是用来定义和限制classpath.
   
   
   * compile: `c`-`t`-`r`.  依赖的有效范围在三个classpath
   * test:  c-`t`-r. 依赖的有效范围在测试classpath
   * provided:  `c`-`t`-r. 依赖的有效范围在编译和测试的classpath,在运行时态无效，运行时态由其他地方提供依赖。比如servlet-api
        ,运行时有container提供。
   * runtime: c-`t`-`r`. 依赖的有效范围在测试和运行时有效，编译时无效。比如JDBC的具体实现，编译的时候用jdk的JDBC接口就行，
        不需要具体实现的依赖。
   * system: `c`-`t`-r. 依赖可见范围和`provided`一样，但是必须通过`systemPath`显示指定依赖文件的路径
   * import: c-t-r. 不会对maven的classpath产生任何影响，也就是都看不见
     
     传递依赖，A -依赖-> B, B -依赖-> C. A->B:第一直接依赖，B->C:第二直接依赖。 A -> C叫传递依赖。如果在maven中定义A的dependency,
     传递依赖是否在maven中存在并且让C影响maven的class，主要取决于第二直接依赖。
     
   * 如果第二直接依赖是`compile`,传递依赖存在且为第一直接依赖
   * 如果第二直接依赖是`runtime`,传递依赖存在且为第一直接依赖，except第一直接依赖为`compile`的时候会变成runtime
   * 如果第二直接依赖是`provided`,传递依赖只有在第一直接依赖是`provided`的时候存在
   * 如果第二直接依赖是`test`,传递依赖不存在
     
     
   |    依赖   | 第二直接依赖 |  compile  |test| provided |  runtime  |
   |-|-|-|-|-|-|
   | 第一直接依赖 |            |           |    |          |         |    
   |compile   |            | compile   | NO |       NO |   runtime |
   |test      |           |  test     |  NO|       NO |    test   |
   |provided  |           |  provided |  NO| provided |  provided |
   |runtime   |           |  runtime  |  NO|        NO|   runtime |
    
  如果多个冲突的传递依赖(版本不同)被解析到，maven用来解决依赖冲突(dependency mediation)的方法
  * 路径最短有限。如果有两多个冲突的传递依赖存在，传递路径最短的优先
  * 路径相同的情况下，pom文件中声明顺序优先
     
  ### optional
   如果依赖配置为可选的，那么此依赖只对当前pom起作用，当其他文件依赖当前pom的时候，此依赖不传递
   
  ### exclusions
  
   如果项目 A -依赖-> B, B -依赖-> C(version1.1)
   如果我们配置项目A的pom，想用C的版本是version1.2，就可以用exclusions排除项目C的version1.1，并且显示声明项目C的version1.2的依赖
   
         --依赖-- > B  -- X(exclusion) --> C (v1.1)
      A
         --依赖-- > C (v1.2)
         
   ### 解析查看一个pom的依赖
   
   `mvn dependency:list(tree|analyze)`
   
   ### Best Practice
   
    * 显示声明项目中用到的直接依赖
         
 ```
   <dependencies>
     <dependency>
       <groupId>...</groupId>
       <artifactId>...</artifactId>
       <version>...</version>
       <type>...</type>  #packaging, default is jar
       <scope>...</scope> # `compile`(default),  `test`,`provided`,`runtime`,`system`
       <optional>...</optional> #is dependency optional
       <exclusions> # 
          <exclusion>
             <groupId>
             <artifactId>
           </exclusion>
       </exclusions>
     </dependency>
    </dependencies>
 ```
