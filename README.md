# Apache Hadoop 2.7.1 binary for Windows 64-bit platform

This is an unofficial pre-compiled binary of Apache Hadoop 2.7.1 for Windows 64-bit platform. The tar.gz file is available [here](https://github.com/karthikj1/Hadoop-2.7.1-Windows-64-binaries/releases/download/v2.7.1/hadoop-2.7.1.tar.gz)  under the Releases link in this repo.

The official Hadoop release from Apache does not include a Windows binary and compiling from sources can be tedious so I've made this compiled distribution available.

If you like it, please star my repo.

# Hadoop 2.7.1 Compilation Instructions for Windows 64-bit

I compiled the source using the following tools
 - [Apache Maven 3.3.3](https://maven.apache.org/)
 - [Cygwin64](https://cygwin.com/setup-x86_64.exe). The standard download comes with a minimal set of tools so make sure it includes ```sh, cp, rm, mv, tar, gzip, mkdir, gcc, zlib```.
 - [CMake 3.3](http://www.cmake.org/) - Make sure to use this CMake and not the CMake that comes with cygwin. The win32 binary of Cmake is fine for win64 also (it only creates configuration files/makefiles so 32-bit vs 64-bit doesn’t matter).
 - [Oracle Java 8u51](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). I used 8u51 but any minor version of JDK 7 or JDK 8 should work. If you use JDK 8, read the notes below for a configuration change to avoid doclint errors.
 - [Google Protocol Buffers v2.5](https://github.com/google/protobuf/releases/tag/v2.5.0). Note that this has to be exactly version 2.5. The zip file ```protoc-2.5.0-win32.zip``` contains protoc.exe which works on win64 also. After downloading and unzipping, you can check that you have the right version by typing in ```protoc --version```. You should see a single output line that says 
 ``` libprotoc 2.5.0 ```.
 - [Microsoft SDK for Windows 7 v7.1](http://www.microsoft.com/en-us/download/details.aspx?id=8279). If you have the Visual C++ 2010 redistributable already installed, this installation may fail with an error that looks like this.
```
Installation of the “Microsoft Windows SDK for Windows 7” product has reported the following error: Please refer to Samples\Setup\HTML\ConfigDetails.htm document for further information.
```
This usually means that you need to uninstall your existing Visual C++ 2010 redistributable and install the one that comes with the SDK package even though it is a slightly older version.
 - [Microsoft .NET Framework v4.0](http://www.microsoft.com/en-us/download/details.aspx?id=17851) - Again, you will need exactly v4.0. If you have v4.5 or later installed, you will need to uninstall that and download/install v4.0.
 - Visual C++ 2010 x64 Redistributable v10.0.*30319* - Note the minor version number needs to be *exactly* as listed here. This is the version number that comes with the Microsoft SDK for Windows mentioned above.
 

The last 3 components should work on Windows 8.1 also as long as you uninstall the newer versions and install the versions mentioned above. 
You may also be able to use Visual Studio 2010 (on Windows 7) to build it but I have not tried it.

If you want to use the [Microsoft SDK for Windows 8.1](https://msdn.microsoft.com/en-us/windows/desktop/bg162891.aspx), you should use .NET Framework 4.5 and the Visual C++ redistributable that comes with the SDK for Windows 8.1. There is no command-line interface for this so you will have to use Visual Studio 2013 to build.

The rest of these notes assume you are building from the command line using the SDK for Windows 7 and .NET Framework v4.0.

You can largely use the instructions at [this site](http://www.srccodes.com/p/article/38/build-install-configure-run-apache-hadoop-2.2.0-microsoft-windows-os) to build but I've added some notes below to describe how it can be built with Java 8. I've also added some notes on build errors that are commonly encountered and how to fix them.

### Windows PATH environment variable 
Edit Path Variable to add bin directory of Cygwin (say ```C:\cygwin64\bin```), bin directory of Maven (say ```C:\maven\bin```) and installation path of Protocol Buffers (say ```c:\protobuf```). The protobuf directory should be the directory that contains ```protoc.exe```.

Also add the folder containing MSBuild.exe to the PATH (should be C:\Windows\Microsoft.NET\Framework64\v4.0.30319).

After successfully compiling Hadoop, you will need to add the Hadoop directory to the PATH as well. Installation instructions for Hadoop on Windows are [here](https://wiki.apache.org/hadoop/Hadoop2OnWindows).

### Windows JAVA_HOME and M2_HOME environment variables
These two environment variables need to be set to point to the JDK and Maven installations. The catch is that the path in JAVA_HOME cannot have space but most Windows users have their Java installation in C:\Program Files which does have a space. There are two workarounds 
- Use Windows shortened name (say 'PROGRA~1' for 'Program Files') or 
- My preferred method - create a folder called C:\Java which your JAVA_HOME can target. Put a [symlink](https://technet.microsoft.com/en-us/library/Cc753194.aspx) in C:\Java to point to your actual Java installation. This also makes things easier if you have multiple versions of Java on your machine.

### Windows Platform environment variable
Create an environment variable called Platform and set its value to x64 or Win32 to build for 64-bit or 32-bit Windows. Both the variable name and its value are case-sensitive.

### Compiling with Java 8
The directory where you downloaded the sources contains the configuration file for Maven (called ```pom.xml```). Find the section that says 
```
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
```
Just below that is another section enclosed in ```<configuration></configuration>``` tags.
Add a line inside this configuration section that says ```	      <additionalparam>-Xdoclint:none</additionalparam>```.
This will eliminate docLint errors due to compiling with Java 8.

### Compilation instructions
The maven command listed below should be run from the Windows SDK 7.1 Command Prompt, not the regular Windows command prompt. After installing  Windows 7.1 SDK, click on Windows 7.1 SDK in Start Menu and the Win 7.1 SDK Cmd prompt is one of the options. 

Compile using 
```
mvn package -Pdist,native-win -DskipTests -Dtar
```
Note that there are no spaces in ```-Pdist,native-win``` and the -P and -D options are case-sensitive.

If everything goes well in the previous step, then the native distribution ```hadoop-2.7.1.tar.gz``` will be created inside ```C:\hadoop-2.7.1-src\hadoop-dist\target``` directory. (assuming your unpacked source files were in ```C:\hadoop-2.7.1-src```).

### Common Errors
Some common errors encountered during compilation and workarounds are listed below. In general, if you encounter build errors, try rerunning maven as 
```
mvn package -Pdist,native-win -DskipTests -Dtar –X –e
```
Scrolling up a bit from the bottom after running the above command will show more detailed errors above the BUILD FAILED line. These can help with diagnosis.

- Could not create named visual generator – Usually, this means you are using the Cygwin version of cmake instead of the cmake downloaded from the cmake website. Check that your PATH env variable doesn’t list the Cygwin folder first if you have both.
- LINK : fatal error LNK1123: failure during conversion to COFF: file invalid or corrupt – Uninstall existing versions of Microsoft .NET Framework and Visual c++ 2010 redist and reinstall .NET Framework 4.0(has to be 4.0). Make sure the MSBuild.exe is on the PATH as described previously.
- Assorted could not find file errors – I found these somewhat misleading. If you see this on the cmake section, it means cmake itself could not be found. Once again, check that you have c:\cmake\bin on the PATH, not c:\cmake or something else.
- Cannot run program "sh" – c:\cygwin64\bin contains the file sh.exe. Make sure you have that file(it comes with Cygwin so download it via Cygwin if needed) and make sure the directory containing the file is on the PATH. If it still doesn’t work, manually add it to the PATH in the shell that you are using by typing SET PATH=c:\path\to\dir\containing\sh;%PATH%. Note there should be no spaces before and after the = sign.

After a successful build and after installing Hadoop, you may find that the dfs file operations work correctly but the mapreduce fails with the following error.
-	UnsatisfiedLinkError with org.apache.hadoop.io.nativeio.NativeIO$Windows.createDirectoryWithMode0 or org.apache.hadoop.io.nativeio.NativeIO$Windows.createFileWithMode0. 

This suggests a problem with the path so make sure the Windows PATH environment variable includes the hadoop/bin directory. If you have more than one Hadoop version installed, make sure any previous versions are not hanging around on the path before the current version's Hadoop/bin directory. That should fix this problem.

- When running the start-yarn and start-dfs commands, make sure you have write permissions to the hadoop namenode and datanode directories. Just make sure the two paths are different from each other. You can set these directories in etc/hadoop/hdfs-site.xml as:
```
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/any/path</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/any/path2</value>
    </property>
```
