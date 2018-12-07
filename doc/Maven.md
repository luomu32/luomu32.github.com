# Maven

Maven是Java中非常流行的包管理工具。

## 运行流程

~~清理->初始化->编译->测试->打包->集成测试->验证->部署->生成站点~~

每个阶段通过不同的插件来完成整个构建流程。当然一个插件也可以跨多个流程。

三套生命周期：clean、default和site。加粗表示常用的阶段。

1. clean
   1. pre-clean
   2. clean：清理上次构建产生的文件
   3. post-clean
2. default
   1. validate：验证项目结构，必要的配置文件是否存在
   2. initialize：构建前的初始化，比如初始化参数、创建目录
   3. generate-sources：产生在编译过程中需要的源代码
   4. process-sources：处理源代码
   5. **generate-resources**：产生资源文件？
   6. **process-resources**：将资源文件复制到classpath对应包中
   7. **compile**：编译
   8. process-classes：产生变异过程中生成的文件
   9. generate-test-sources：产生测试代码
   10. process-test-sources：处理测试代码
   11. **generate-test-resources**：
   12. **process-test-resources**：
   13. **test-compile**：编译测试代码
   14. process-test-classes：
   15. **test**：运行测试用例
   16. prepare-package：处理打包前的初始化工作
   17. package：将编译后的文件打包或压缩文件
   18. pre-integration-test：集成测试前的准备工作
   19. integration-test：集成测试
   20. post-integration-test：完成集成测试后的工作，比如清理
   21. verify：检测测试后的包是否完好
   22. **install**：将打包好的构建物安装到本地依赖仓库中
   23. **deploy**：发布到远程仓库中
3. site
   1. pre-site
   2. site
   3. post-site
   4. site-deploy

通过Maven命令，可以执行对应的阶段。比如`mvn clean install`。因为阶段按照顺序执行，不能跳着执行，所以如果执行clean阶段，那么之前的pre-clean会被先执行。也就是说，不需要写成`mvn clean package install`，因为package阶段在install之前，执行install阶段会先执行package阶段。

## 插件

Maven规定了每个项目构建的生命周期，但没有实现。实现交由不同的插件。Maven官方提供了很多的插件用来完成在核心生命周期的比较基础的功能。换而言之，一个插件必须绑定到一个生命周期才能被调用生效。当然一个插件可以绑定到多个生命周期。插件每个实现的功能称之为**目标**。

Maven默认为一些生命周期的阶段绑定好了插件的目标。比如执行clean周期的clean阶段，Mavne会调用maven-clean-plugin插件的clean目标。如果需要手动绑定，可以在项目的`pom.xml`文件中指定：

```xml
<plugin>
    <groupId>xx.xxx.xx</groupId>
    <artifactId>xxx-xxx-xx</artifactId>
    <version>x.x.x</version>
    <executions>
        <execution>
            <id>xxx</id>
            <phase></phase>
            <goals>
                <goal>xxx</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

如上图示例，`phase`节点用于绑定的生命周期的阶段，`goal`指定执行的目标。一个阶段可以执行多个目标，如果插件支持的话。有些插件指定目标可以不指定阶段，插件会为目标设置一个默认的阶段。比如`maven-source-plugin`插件的`jar-no-fork`目标默认在`package`阶段执行。通过命令`mvn help:describe -Dplugin=org.apache.mavne.plugins:maven-source-plugin:3.0.0 -Ddetail`查看插件的详细信息，这个功能通过插件`maven-help-plugin`来完成的。

插件执行目标时，有时候还可以指定参数。

- 通过命令行：`mvn install -Dmaven.test.skip=true`。`maven-surefire-plugin`插件提供了`maven.test.skip`参数

- pom文件配置：

  ```xml
  <groupId></groupId>
  <artifactId></artifactId>
  <configuration>
      ....
  </configuration>
  ```

  `configuration`配置还可以放在具体的目标中

插件的目标可以通过命令行直接执行，比如`mvn dependency:tree`，执行`maven-dependency-plugin`的`tree`目标。`maven <插件名称|前缀>:<目标>[-D参数名=参数值...]`

插件的仓库和依赖的仓库是不同的。也可以在pom或setting文件中配置

官方插件：

- maven-clean-plugin
- maven-compile-plugin
- maven-source-plugin
- maven-resource-plugin
- maven-site-plugin

### 自定义插件开发

创建一个项目。项目的`packaging`必须是`maven-plugin`。

```java
@Goal
public class Mojo extends AbstractMojo{
    @Override
    public void execute() throws MojoExecutionException,MojoFailureException{
        //逻辑代码....
    }
}
```



## 仓库

内置中央仓库

`<distributionManagemenet>`节点可以配置发布构件的仓库。通过`<server>`节点可以配置发布该仓库所需的用户名和密码，通过id来绑定仓库。`<mirror>`节点配置镜像。`mirrorOf`配置*表示所有仓库的请求都会转发到镜像仓库上去。

## 依赖

Maven会对项目分别控制三套classpath：编译、测试和运行。称之为**依赖范围**

- compile。默认。编译、测试、运行三套classpath都加入，都生效
- test。只加入测试classpath
- provided。加入编译和测试的classpath。打出包时不会包括该范围的依赖
- runtime。测试和运行
- system。范围与provided一致。需搭配systemPath指定依赖路径，不由Maven仓库管理依赖。
- import。不对三套classpath产生影响。只将dependencyManagement依赖导入当前项目的dependencyManagement中。

重复的传递依赖，先按照路径，路径短的引用；路径一致，再按声明顺序，在前的引用。

可选依赖。

## Profile

Profile可以用于对不同的环境生效不同的配置。

有多种方式激活：

- 通过命令行。比如：`mvn clean install -Pdev`。
- 在setting文件配置：

```xml
<acticeProfiles>
    <activeProfile>xxx</activeProfile>
</acticeProfiles>
```

- 根据条件激活：

```xml
<profile>
    <activation>
        <property>
            <name>profileProperty</name>
            <value>dev</value>
        </property>
    </activation>
</profile>
```

配合命令`mvn clean install -DprofileProperty=dev`。或：

```xml
<profile>
    <activation>
        <os>
        </os>
    </activation>
</profile>
```

或：

```xml
<profile>
    <activation>
        <file>
            <missing>xxx</missing>
            <exists>xxx</exists>
        </file>
    </activation>
</profile>
```

`mvn help:active-profiles`查看激活的profile。

profile可以配置在pom文件和settting文件中。

## Archetype

用于生成项目骨架。该功能通过`maven-archetype-plugin`插件来实现的。所以通过命令`mvn archetype:generate`可以创建

从archetype-catalog.xml文件中读取可用的Archetype列表。`maven-archetype-plugin`插件内置了一个Archetype Catalog。可以在.m2目录下新建一个archetype-catalog.xml来配置。Maven的中央仓库也提供了一个Archetype Catalog。

