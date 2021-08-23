###  java项目混淆加密方案

proguard简单来说是为了防止反编译，更准确的说，是使得代码易读性变差。

#### maven pom配置中加入以下：

```
<!-- ProGuard混淆插件-->
            <plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>2.0.11</version>
                <executions>
                    <execution>
                        <!-- 混淆时刻，这里是打包的时候混淆-->
                        <phase>package</phase>
                        <goals>
                            <!-- 指定使用插件的混淆功能 -->
                            <goal>proguard</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!-- 是否将生成的PG文件安装部署-->
                    <attach>true</attach>
                    <!-- 是否混淆-->
                    <obfuscate>true</obfuscate>
                    <!-- 指定生成文件分类 -->
                    <attachArtifactClassifier>pg</attachArtifactClassifier>
                    <proguardInclude>${basedir}/proguard.conf</proguardInclude>
                    <libs>
                        <lib>${java.home}/lib/rt.jar</lib>
                        <lib>${java.home}/lib/jce.jar</lib>
                    </libs>
                    <!-- 对什么东西进行加载，这里仅有classes成功，不可能对配置文件及JSP混淆吧-->
                    <injar>classes</injar>
                    <outjar>${project.build.finalName}-pg.jar</outjar>
                    <!-- 输出目录-->
                    <outputDirectory>${project.build.directory}</outputDirectory>
                </configuration>
            </plugin>
```

