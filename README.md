# Reproduction of test-jar bug in maven shade plugin

## Steps to reproduce
1. Run `mvn clean install`
2. Look at the installed pom `cat cat ~/.m2/repository/org/example/uber-jar/1.0-SNAPSHOT/uber-jar-1.0-SNAPSHOT.pom`


## Output
The build output indicates the test-jar is included in the uber-jar
```shell
...
[INFO] --- shade:3.5.0:shade (default) @ uber-jar ---
[INFO] Including org.example:module-one:jar:1.0-SNAPSHOT in the shaded jar.
[INFO] Including org.example:module-one:test-jar:tests:1.0-SNAPSHOT in the shaded jar.
[INFO] Including org.example:module-two:jar:1.0-SNAPSHOT in the shaded jar.
...
```

List contents of JAR
```shell
>$ jar -tf ~/.m2/repository/org/example/uber-jar/1.0-SNAPSHOT/uber-jar-1.0-SNAPSHOT.jar 
...
moduleone/ModuleOne.class
moduleone/TestModuleOne.class
moduletwo/ModuleTwo.class
...
```

List contents of installed POM file
```shell
>$ cat /home/nick/.m2/repository/org/example/uber-jar/1.0-SNAPSHOT/uber-jar-1.0-SNAPSHOT.pom 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <parent>
    <artifactId>shade-plugin-test-jar-repro</artifactId>
    <groupId>org.example</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>uber-jar</artifactId>
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.5.0</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <artifactSet>
                <includes>
                  <include>org.example</include>
                </includes>
              </artifactSet>
            </configuration>
          </execution>
        </executions>
        <configuration>
          <useDependencyReducedPomInJar>true</useDependencyReducedPomInJar>
        </configuration>
      </plugin>
    </plugins>
  </build>
  <dependencies>
    <dependency>
      <groupId>org.example</groupId>
      <artifactId>module-one</artifactId>
      <version>1.0-SNAPSHOT</version>
      <type>test-jar</type>
      <scope>compile</scope>
    </dependency>
  </dependencies>
</project>
```

## Expected behaviour
All "included" dependencies are omitted from the POM

## Actual behaviour
`org.example:module-one:test-jar:compile` remains in the POM despite being "included" in the uber-jar