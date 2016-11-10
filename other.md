# kalix-parent
  基础核心pom文件
## pom.xml
**pluginManagement**

plugin for enhance bean entity

    <plugin>
		<!--the plugin for enhance the bean entity-->
        <groupId>org.apache.openjpa</groupId>
        <artifactId>openjpa-maven-plugin</artifactId>
        <version>${openjpa-maven-plugin.version}</version>
        <configuration>
            <includes>**/entities/*.class</includes>
            <addDefaultConstructor>true</addDefaultConstructor>
            <enforcePropertyRestrictions>true</enforcePropertyRestrictions>
            <persistenceXmlFile>${project.build.outputDirectory}/META-INF/persistence-jpa.xml</persistenceXmlFile>
        </configuration>
        <executions>
            <execution>
                <id>enhancer</id>
                <phase>process-classes</phase>
                <goals>
                    <goal>enhance</goal>
                </goals>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>org.apache.openjpa</groupId>
                <artifactId>openjpa</artifactId>
                <version>${openjpa.version}</version>
            </dependency>
        </dependencies>
    </plugin>
</plugins>
	

**bundle plugin**

plugin for compile a project as a bundle,add some properties to MANIFEST.MF file

	<plugin>
	    <groupId>org.apache.felix</groupId>
	    <artifactId>maven-bundle-plugin</artifactId>
	    <version>${maven-bundle-plugin.version}</version>
	    <extensions>true</extensions>
	    <configuration>
	        <instructions>
	            <Bundle-SymbolicName>${bundle.symbolicName}</Bundle-SymbolicName>
	            <Bundle-Version>${project.version}</Bundle-Version>
	            <Export-Package>
	                ${bundle.namespace}.*;version="${project.version}"
	            </Export-Package>
	            <Import-Package>*</Import-Package>
	            <_include>-osgi.bnd</_include>
	        </instructions>
	    </configuration>
	</plugin>
**ant plugin**

plugin action:

auto deploy bundle file to karaf/deploy folder

auto deploy setting.xml to maven/conf folder

auto deploy *.cfg to karaf/ext folder

	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-antrun-plugin</artifactId>
	    <version>${maven-antrun-plugin.version}</version>
	    <executions>
	        <execution>
	            <id>deploy</id>
	            <phase>install</phase>
	            <goals>
	                <goal>run</goal>
	            </goals>
	            <configuration>
	                <target>
	                    <taskdef resource="net/sf/antcontrib/antcontrib.properties"/>
	                    <if>
	                        <equals arg1="${project.packaging}" arg2="bundle" />
	                        <then>
	                            <if>
	                                <available file="target/${project.artifactId}-${project.version}.jar"/>
	                                <then>
	                                    <if>
	                                        <available file="${karaf.deploy.path}" type="dir"></available>
	                                        <then>
	                                            <copy file="target/${project.artifactId}-${project.version}.jar"
	                                                  todir="${karaf.deploy.path}"/>
	                                        </then>
	                                        <else>
	                                            <echo>Karaf deploy path does not exist</echo>
	                                        </else>
	                                    </if>
	                                </then>
	                                <else>
	                                    <echo>The file does not exist</echo>
	                                </else>
	                            </if>
	                        </then>
	                    </if>
	                    <if>
	                        <available file="${maven.conf.path}" type="dir"></available>
	                        <then>
	                            <if>
	                                <available file="src/main/resources/settings.xml"/>
	                                <then>
	                                    <copy file="src/main/resources/settings.xml"
	                                          todir="${maven.conf.path}"/>
	                                </then>
	                            </if>
	                        </then>
	                    </if>
	                    <if>
	                        <equals arg1="${project.artifactId}" arg2="framework-core-etc" />
	                        <then>
	                            <if>
	                                <available file="${karaf.etc.path}" type="dir"></available>
	                                <then>
	                                    <copy todir="${karaf.etc.path}">
	                                        <fileset dir="src/main/resources"/>
	                                    </copy>
	                                </then>
	                                <else>
	                                    <echo>Karaf deploy path does not exist</echo>
	                                </else>
	                            </if>
	                        </then>
	                    </if>
	                </target>
	            </configuration>
	        </execution>
	    </executions>
	    <dependencies>
	        <dependency>
	            <groupId>ant-contrib</groupId>
	            <artifactId>ant-contrib</artifactId>
	            <version>${ant.contrib.version}</version>
	        </dependency>
	    </dependencies>
	</plugin>	


**project deploy**

the id must the same as config in setting.xml

	<distributionManagement>
	    <repository>
	        <id>private-nexus-library-releases</id>
	        <name>private-nexus-library-releases</name>
	        <url>http://nexus3.kalix.com/repository/maven-releases/</url>
	    </repository>
	    <snapshotRepository>
	        <id>private-nexus-library-snapshots</id>
	        <name>private-nexus-library-snapshots</name>
	        <url>http://nexus3.kalix.com/repository/maven-snapshots/</url>
	    </snapshotRepository>
	</distributionManagement>

## About Maven setting.xml
The setting.xml file will auto deploy to ${maven.path}/conf if the maven.path config right.

**peoject deploy**
	
	<server>
		<id>private-nexus-library-releases</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
	<server>
		<id>private-nexus-library-snapshots</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
**use local nexus**

	<mirror>
		<id>nexus2-remote</id>
		<mirrorOf>*</mirrorOf>
		<name>Nexus Mirror Remote</name>
		<url>http://172.18.82.10:8081/nexus/content/groups/public/</url>
	</mirror>
**system evn profile**

	<profile>
	    <id>linux-env</id>
	    <activation>
	        <activeByDefault>false</activeByDefault>
	        <os>
	            <family>Linux</family>
	        </os>
	    </activation>
	    <properties>
	        <karaf.path>~/java-develop/tools/apache-karaf-${karaf.version}</karaf.path>
	    </properties>
	</profile>
	<profile>
	    <id>windows-env</id>
	    <activation>
	        <activeByDefault>false</activeByDefault>
	        <os>
	            <family>Windows</family>
	        </os>
	    </activation>
	    <properties>
	        <karaf.path>d:/java-develop/tools/apache-karaf-${karaf.version}</karaf.path>
	    </properties>
	</profile>
