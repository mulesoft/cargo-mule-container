# Introduction

This project provides a [Cargo](http://cargo.codehaus.org/) implementation for [Mule](http://www.mulesoft.org) container.
It provides abstration to deploy Mule applications to installed Mule servers.

Documentation will focus on [Maven](http://maven.apache.org/) integration. Find more details on other integration on [Cargo](http://cargo.codehaus.org/) website.

## Configuration

To use cargo-mule-container you will need to:

1. define mulesoft.org release repository
2. add cargo as a dependency to your project
3. add cargo-mule-container as a dependency to cargo
4. define your mule standalone installation path
5. define applications you want to deploy
6. bind cargo start/stop goals to a Maven phase to start/stop a mule instance ``(optional)``
7. bind cargo deploy/undeploy goals to a Maven phase to deploy/undeploy your applications
8. use maven-failsafe plugin to run tests on your deployed applications ``(optional)``

## Define mulesoft.org release repository

Update your pom to contain:

```xml
...
  <pluginRepositories>
    <pluginRepository>
      <id>muleforge.release</id>
      <url>http://repository.mulesoft.org/releases/</url>
    </pluginRepository>
  </pluginRepositories>
...
```

## Add cargo and cargo-mule-container as a dependency

Update your pom to list cargo as a dependency:

```xml
...
  <properties>
    <cargoVersion>1.2.3</cargoVersion>
    <muleVersion>3.3.0</muleVersion>
  </properties>

  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          <dependency>
            <groupId>org.mule.tools.cargo</groupId>
            <artifactId>container</artifactId>
            <version>${muleVersion}</version>
          </dependency>
      </plugin>
    </plugins>
  </build>
...
```

Depending on your deployment options you may need to add other dependencies. See section 'Use cases'.

## Container definition

Update your pom to define your container:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          ...
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            <containerId>mule3x</containerId>
            <home>/Path/To/Mule</home>
          </container>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
```

 See section 'Use cases' for available containers.

## Applications definition

 Update your pom to list your deployed applications:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          ...
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            ...
          </container>
          <configuration>
            <deployables>
              <deployable>
                <groupId>org.mule.examples</groupId>
                <artifactId>mule-example-echo</artifactId>
                <type>zip</type>
              </deployable>
            </deployables>
          </configuration>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
```

You can also deploy application defined by current project:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          ...
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            ...
          </container>
          <deployer>
            <type>installed</type>
          </deployer>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
```

## Bind container start/stop

Update your pom to start/stop mule:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        ...
        <executions>
          <execution>
            <id>start-container</id>
            <phase>pre-integration-test</phase>
            <goals>
              <goal>start</goal>
            </goals>
          </execution>
          <execution>
            <id>stop-container</id>
            <phase>post-integration-test</phase>
            <goals>
              <goal>stop</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
...
```

## Bind applications deployment

Update your pom to deploy/undeploy applications:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        ...
        <executions>
          <execution>
            <id>start-container</id>
            <phase>pre-integration-test</phase>
            <goals>
              <goal>deploy</goal>
            </goals>
          </execution>
          <execution>
            <id>stop-container</id>
            <phase>post-integration-test</phase>
            <goals>
              <goal>undeploy</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
...
```

If you need to start/stop a container to deploy your applications last two steps can be combined:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        ...
        <executions>
          <execution>
            <id>start-container</id>
            <phase>pre-integration-test</phase>
            <goals>
              <goal>start</goal>
              <goal>deploy</goal>
            </goals>
          </execution>
          <execution>
            <id>stop-container</id>
            <phase>post-integration-test</phase>
            <goals>
              <goal>undeploy</goal>
              <goal>stop</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
...
```

## Run integration tests

Update your pom to run integration tests:

```xml
...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-failsafe-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>integration-test</goal>
              <goal>verify</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
...
```

If you follow all those steps then running `mvn post-integration-test` you will see:

1. mule container being started
2. applications being deployed
3. tests being run
4. applications being undeployed
5. mule container being stopped


# Use cases

Bellow are sample configurations detailing how to deploy echo example on a mule standalone.

## Deploy echo example in an installed mule instance (mule installation required)

You can find a complete sample [here](https://github.com/mulesoft/cargo-mule-container/tree/master/integration-tests/installed).

Allows to start a container using a local Mule installation.
This container implementation does not support deployable and file deployer implementation can be used.

When using this container you will need to provide:
* a valid Mule home location

This container is implemented by org.mule.tools.cargo.container.Mule3xInstalledLocalContainer class and must be used with associated:
* local configuration (org.mule.tools.cargo.container.configuration.Mule3xLocalConfiguration)
* deployable type (org.mule.tools.cargo.deployable.MuleApplicationDeployable)

Example using a mule application:

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          <dependency>
            <groupId>org.mule.tools.cargo</groupId>
            <artifactId>container</artifactId>
            <version>${project.version}</version>
          </dependency>
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            <containerId>mule3x</containerId>
            <home>/path/to/your/mule/home</home>
          </container>
          <deployer>
            <type>installed</type>
          </deployer>
          <deployables>
            <deployable>
              <groupId>org.mule.examples</groupId>
              <artifactId>mule-example-echo</artifactId>
              <type>zip</type>
            </deployable>
          </deployables>
        </configuration>
      </plugin>
    </plugins>
  </build>
...
```

Or using the regular mule-standalone distribution as dependency (no need for local mule installation):

```xml
...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
          <execution>
            <id>unpack</id>
            <phase>package</phase>
            <goals>
              <goal>unpack</goal>
            </goals>
            <configuration>
              <artifactItems>
                <artifactItem>
                  <groupId>org.mule.distributions</groupId>
                  <artifactId>mule-standalone</artifactId>
                  <version>${muleVersion}</version>
                  <type>zip</type>
                  <overWrite>true</overWrite>
                  <outputDirectory>${mule.distribution}</outputDirectory>
                </artifactItem>
              </artifactItems>
              <overWriteReleases>true</overWriteReleases>
              <overWriteSnapshots>true</overWriteSnapshots>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          <dependency>
            <groupId>org.mule.tools.cargo</groupId>
            <artifactId>container</artifactId>
            <version>${project.version}</version>
          </dependency>
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            <containerId>mule3x</containerId>
            <home>${mule.distribution}/mule-standalone-${muleVersion}</home>
          </container>
          <deployer>
            <type>installed</type>
          </deployer>
          <deployables>
            <deployable>
              <groupId>org.mule.examples</groupId>
              <artifactId>mule-example-echo</artifactId>
              <type>zip</type>
            </deployable>
          </deployables>
        </configuration>
      </plugin>
    </plugins>
  </build>
...
```

## Specify Java system proerties

```xml
...
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.cargo</groupId>
        <artifactId>cargo-maven2-plugin</artifactId>
        <version>${cargoVersion}</version>
        <dependencies>
          <dependency>
            <groupId>org.mule.tools.cargo</groupId>
            <artifactId>container</artifactId>
            <version>${project.version}</version>
          </dependency>
        </dependencies>
        <configuration>
          <wait>false</wait>
          <container>
            <containerId>mule3x</containerId>
            <home>/path/to/your/mule/home</home>
          </container>
          <deployer>
            <type>installed</type>
          </deployer>
          <deployables>
            <deployable>
              <groupId>org.mule.examples</groupId>
              <artifactId>mule-example-echo</artifactId>
              <type>zip</type>
            </deployable>
          </deployables>
          <configuration>
              <properties>
                  <cargo.system-properties>mule.mmc.bind.port=8888</cargo.system-properties>
              </properties>
          </configuration>
        </configuration>
      </plugin>
    </plugins>
  </build>
...
```