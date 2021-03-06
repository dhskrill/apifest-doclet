#ApiFest Doclet
ApiFest Doclet is a tool that generates ApiFest mapping configuration file (XML) from Javadoc.
Here are the custom Javadoc annotations that ApiFest Doclet is aware of:

- @apifest.external - the endpoint visible to the world;
- @apifest.internal - your API endpoint;
- @apifest.action - the class name of the action that will be executed before requests hit your API;
- @apifest.filter - the class name of the filter that will be executed before responses from API are returned back;
- @apifest.scope - scope(s)(space-separated list) of the endpoint;
- @apifest.auth.type - *user* if user authentication is required, *client-app* if only client application authentication is required. 
Note, that if the endpoint could be accessible without access token, then just skip this tag;
- @apifest.re.{varName} - regular expression used for variable with name {varName} (without brackets); if several variables, then
add @apifest.re.{varName} for each of them.


Currently, JAX-RS HTTP method annotations are used for setting the HTTP method of the endpoint.

###Features

- generates ApiFest mapping configuration file from your Javadoc - custom annotations used;
- keeps your code clean - no versions required in Javadoc annotations;
- all version/environment specific settings are passed as variables; 
- easy integration in maven projects


###Usage
ApiFest Doclet requires the following environment variables:

- mapping.version - the version your API will be exposed externally;
- mapping.filename - the name of the mapping configuration file that will be generated;
- backend.host - the host(your API is running on) where requests should be translated to;
- backend.port - the port of the backend.host;
- application.path - the application path used to obtain all application resources.
- defaultActionClass - the fully qualified action class that will be added if no action is declared in Javadoc annotations
- defaultFilterClass - the fully qualified filter class that will be added if no filter is declared in Javadoc annotations

If your project uses maven, here is an example integration of ApiFest Doclet in your pom.xml:
```
...
<profiles>
    <profile>
      <id>gen-mapping</id>
      <build>
        <plugins>
          <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>properties-maven-plugin</artifactId>
            <version>1.0-alpha-2</version>
            <executions>
              <execution>
                <phase>validate</phase>
                <goals>
                  <goal>read-project-properties</goal>
                </goals>
                <configuration>
                  <files>
                    <file>${basedir}/src/main/resources/project.properties</file>
                  </files>
                </configuration>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>2.9.1</version>
            <executions>
              <execution>
                <phase>validate</phase>
                <goals>
                  <goal>javadoc</goal>
                </goals>
                <configuration>
                  <doclet>com.apifest.doclet.Doclet</doclet>
                  <docletArtifact>
                    <groupId>com.apifest</groupId>
                    <artifactId>apifest-doclet</artifactId>
                    <version>0.1.0</version>
                  </docletArtifact>
                  <additionalJOptions>
                    <additionalJOption>-J-Dmapping.version=${mapping.version}</additionalJOption>
                    <additionalJOption>-J-Dmapping.filename=${mapping.filename}</additionalJOption>
                    <additionalJOption>-J-Dbackend.host=${backend.host}</additionalJOption>
                    <additionalJOption>-J-Dbackend.port=${backend.port}</additionalJOption>
                    <additionalJOption>-J-Dapplication.path=${application.path}</additionalJOption>
                    <additionalJOption>-J-DdefaultActionClass=${defaultActionClass}</additionalJOption>
                    <additionalJOption>-J-DdefaultFilterClass=${defaultFilterClass}</additionalJOption>
                  </additionalJOptions>
                  <useStandardDocletOptions>false</useStandardDocletOptions>
                </configuration>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
...
```

where *backend.host* and *backend.port* are defined in *project.properties* file; *mapping.version* and *mapping.filename* 
could be defined in your pom.xml file so they stick to the code. *application.path* is the application path used to 
obtain all application resources.
Then you can use the following command in order to generate mapping xml file:
```mvn install -P gen-mapping```

Note, that *mapping.version* value will be prepended to the apifest.external annotation value.
If no value is set for *mapping.filename*, then *output_mapping_[mapping.version].xml* will be used. 
If the example pom integration is used, then the mapping file will be stored in *target/site/apidocs* directory. 