
maven global repository [http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/)
or repo.maven.apache.org/maven2  `id=central`
it is defined in `maven-model-builder-3.6.2.jar`

```
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
  ```

jboss maven repository [http://repository.jboss.com/maven2/](http://repository.jboss.com/maven2/)

## Maven 构建顺序 

   compile -> test --> package --> install.
   后一个会自动包含前一个命令
   
   
## Maven坐标

   groupId, artifactId, version, packaging (default `jar`), classifier(附属构件)
   
   构件文件名： `artifactId-version[-classifier].packaging`  
   
   构件在仓库中的位置
   
   `groupId/artifactId/version/artifactId-version[-classifier].packaging`
   see http://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-databind/2.9.9.3/
   
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
   * `mvn dependency:analyze`分析出的定义但没有用到的依赖要谨慎删除，因为有可能是为了显示的限制传递依赖的版本
         
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
 
 ## 镜像
   Use `mirrorOf` to specific a repositoryID, for all the request to that specific repository, will redirect to this mirror
   `id=*` will redirect request for all repositorys. 
   
   ```
    <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |-->
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
  </mirrors>
   ```
   
   ## Repository management software
   
   `Sonatype Nexus`, `Jarvana`, `MVNRepository`
   
   ## Maven生命周期
     
     More about lifecycle http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
     
     Maven有三套互相独立的生命周期
     `clean`生命周期：负责项目清理,包括 pre-clean->clean->post-clean
     `default`生命周期：负责构建所需要的所有步骤，包括 validate->initialize->...->compile->...->test->...->package->...->verify->install->deploy
     `site`生命周期：负责建立和发布项目站点,pre-site->site->post-site->site-deploy
     
     生命周期周期中的每一个阶段的执行都包含这个阶段之前的阶段。
     比如
     `mvn clean install` 执行了clean周期中的pre-clean,clean以及default周期中install之前的步骤
     
     maven生命周期中的每一步，都是和maven插件的目标绑定的。具体绑定可见 http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
 
   ## Maven插件和插件目标
   
   每一个插件完成多种功能，每一个功能点叫插件目标 `dependency:list`  其中dependency表示插件 `maven-dependency-plugin`,而list表示的是插件目标，除此之外，还有`dependency:tree`, `dependency:analyze`
   
   maven的官方插件 http://maven.apache.org/plugins/index.html
  
   as `test` step is handled by maven-surefire-plugin, we can use `mvn surefire:help -Ddetail=true` to get the help content of surefure, as surefire has a help goal. 
   
   ### 目标前缀
   we can use 
   `mvn help:describe -Dplugin=groupid:artifactId:version` 查看插件的帮助，
   -Dplugin可以简化为插件目标前缀，比如
   `mvn help:describe -Dplugin=surefire`
   `mvn help:describe -Dplugin=compiler`
   
   help是 `maven-help-plugin`的目标前缀,dependency是`maven-dependency-plugin`的目标前缀
