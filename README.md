### Spring Boot Native Image Step by Step

1. ###### Windows 10 Environment Setup

   - Download graalvm JDK, prefer to select community version 17. Detailed information you can refer to https://www.graalvm.org/22.1/docs/getting-started/windows/

   - Update or setup the environment JAVA-HOME to the graalvm JDK directory

   - Install the native-image by the below command. It will download from github and not stability

     ```powershell
     gu install native-image
     ```

   - Upgrade/Install you Windows SDK and VC++ components, VC++ component version 14.29.30133 and SDK version 10.0.18362.0 verified is positive. 

   - Update or setup the environments, it will be used when to compile the JAR to exe (By your directories of VS)

     - SET LIB=D:\Windows Kits\10\Lib\10.0.18362.0\um\x64;D:\Windows Kits\10\Lib\10.0.18362.0\ucrt\x64;D:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.29.30133\lib\x64
     - SET INCLUDE=D:\Windows Kits\10\Include\10.0.18362.0\um;D:\Windows Kits\10\Include\10.0.18362.0\ucrt;D:\Windows Kits\10\Include\10.0.18362.0\shared;D:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.29.30133\include

2. ##### GraalVM Native Image support

   - Spring Boot Document https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.testing.with-native-build-tools
   - Sample for Spring Boot 3 & 2 https://www.baeldung.com/spring-native-intro
   - Graalvm: https://www.graalvm.org/latest/docs/

3. ###### Prerequisites

   - The last version for Spring Native is 0.12.2.  Spring Native 0.12.2 only supports Spring Boot 2.7.7, so change the version if necessary. (Tested with Spring boot 2.7.14 can work as well. Maybe some features can't work)
   - <strong>Spring Boot 2.X</strong>. As not integrated with Spring native, many configurations are required
     - Add dependencies for spring native and plugin for native image build

   ```xml
   <repositories>
       <repository>
           <id>spring-milestones</id>
           <name>Spring Milestones</name>
           <url>https://repo.spring.io/milestone</url>
       </repository>
   </repositories>
   <pluginRepositories>
       <pluginRepository>
           <id>spring-milestones</id>
           <name>Spring Milestones</name>
           <url>https://repo.spring.io/milestone</url>
       </pluginRepository>
   </pluginRepositories>
   <dependency>
       <groupId>org.springframework.experimental</groupId>
       <artifactId>spring-native</artifactId>
       <version>0.12.1</version>
   </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.apache.tomcat.embed</groupId>
                <artifactId>tomcat-embed-core</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.apache.tomcat.embed</groupId>
                <artifactId>tomcat-embed-websocket</artifactId>
            </exclusion>
        </exclusions>
   </dependency>
   <dependency>
       <groupId>org.apache.tomcat.experimental</groupId>
       <artifactId>tomcat-embed-programmatic</artifactId>
       <version>${tomcat.version}</version>
   </dependency>
   <!--if need build image by spring boot-->
   <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <image>
                <builder>paketobuildpacks/builder:tiny</builder>
                <env>
                    <BP_NATIVE_IMAGE>true</BP_NATIVE_IMAGE>
                </env>
            </image>
        </configuration>
   </plugin>
   <!--Spring AOT maven plugin-->
   <plugin>
       <groupId>org.springframework.experimental</groupId>
       <artifactId>spring-aot-maven-plugin</artifactId>
       <version>0.12.2</version>
       <executions>
           <execution>
               <id>generate</id>
               <goals>
                   <goal>generate</goal>
               </goals>
           </execution>
       </executions>
   </plugin>
   <plugin>
       <groupId>org.graalvm.buildtools</groupId>
       <artifactId>native-maven-plugin</artifactId>
       <version>0.9.23</version>
       <configuration>
           <mainClass>com.example.demo.DemoApplication</mainClass>
           <skipNativeTests>true</skipNativeTests>
           <requiredVersion>22.3</requiredVersion>
           <buildArgs>
               <buildArg>--report-unsupported-elements-at-runtime</buildArg>
               <arg>--initialize-at-run-time=io.netty.handler.ssl.BouncyCastleAlpnSslUtils,io.netty</arg>
           </buildArgs>
       </configuration>
   </plugin>
   <profiles>
       <profile>
           <id>native</id>
           <build>
               <plugins>
                   <plugin>
                       <groupId>org.graalvm.buildtools</groupId>
                       <artifactId>native-maven-plugin</artifactId>
                       <version>0.9.23</version>
                       <executions>
                           <execution>
                               <id>build-native</id>
                               <goals>
                                   <goal>compile-no-fork</goal>
                               </goals>
                               <phase>package</phase>
                           </execution>
                           <execution>
                               <id>add-reachability-metadata</id>
                               <goals>
                                   <goal>add-reachability-metadata</goal>
                               </goals>
                           </execution>
                       </executions>
                       <configuration>
                           <skip>false</skip>
                           <imageName>demo</imageName>
                           <fallback>false</fallback>
                           <quickBuild>true</quickBuild>
                       </configuration>
                   </plugin>
                   <plugin>
                       <groupId>org.springframework.boot</groupId>
                       <artifactId>spring-boot-maven-plugin</artifactId>
                       <configuration>
                           <classifier>exec</classifier>
                       </configuration>
                   </plugin>
               </plugins>
           </build>
       </profile>
   </profiles>
   ```

   - <strong>Spring 3.x</strong> (Recommend)

     - Spring 3.x has integrated the spring native features, so suggest to use the version to try the feature. 

       - Use the spring boot parent
       - Enable graalvm build tools maven plugin

       ```xml
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>3.1.1</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       ...
       <build>
           <plugins>
               <plugin>
                   <groupId>org.graalvm.buildtools</groupId>
                   <artifactId>native-maven-plugin</artifactId>
                   <configuration>
                       <mainClass>com.example.demo.DemoApplication</mainClass>
                   </configuration>
               </plugin>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
       ```

   

   - Spring 2 & 3 build native image

     ```
     clean native:compile -Pnative
     ```

   - Execute and Verify
     - cd ./target
     - ./xxx.exe

4. ##### Linux for the setup

   - Linux is easily to setup the environment to build the native image, but need focus on some points
     - gcc++ version
     - Install the zlib-devel if it is missing
       - yum install zlib-devel (Both x86_64 and i686 are required)
