将本地jar添加到本地Maven仓库

```powershell
mvn install:install-file -Dfile=E:\ios-license-1.0.2.jar -DgroupId=com.geostar.geoios -DartifactId=ios-license -Dversion=1.0.2 -Dpackaging=jar
```

