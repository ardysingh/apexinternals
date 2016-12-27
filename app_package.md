# Application Package

An Application package is a jar archive with following structure

```
$ tree .
├── app
│   ├── application.jar
│   └── application.json
├── lib
│   ├── fastutil-6.6.4.jar
│   └── malhar-library-3.7.0-SNAPSHOT.jar
└── META-INF
    ├── MANIFEST.MF
    └── properties.xml
```

The app package contains following directories
* `app`
 
  This directory contain jar file with StreamingApplication. `apexcli` will
  look for applications defined in jar files located in this directory. The applications can also be
  described using `json` and `property` files.
  
* `lib`
   
  This directory contain jar files required for running streaming application. This
  directory should not include jar files provided by Hadoop and Apex platform as those are 
  included by `apexcli` while launching the application. The jar present in this directory is added to CLASSPATH for all containers.
  
* `META-INF`
  
  MANIFEST.MF file contains meta data about the application package. This file is generated
  by the maven.
  
  This directory contains application configuration files. `apexcli` will
  read `properties.xml` as default top level configuration file for the application. see [apexcli](apexcli.md) for more information
  about how properties are applied during launch.
  
