# Spring Security Dependency Resolution Demo


Prior to using NES, we should confirm that the application prior to NES support has the following dependency tree as it relates to the spring-security-oauth2 dependency.

```shell
# Others omitted for brevity
+--- org.springframework.security.oauth:spring-security-oauth2:2.5.2.RELEASE
|    +--- org.springframework:spring-beans:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-core:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-context:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-webmvc:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework.security:spring-security-core:4.2.20.RELEASE -> 5.8.16 (*)
|    +--- org.springframework.security:spring-security-config:4.2.20.RELEASE -> 5.8.16 (*)
|    +--- org.springframework.security:spring-security-web:4.2.20.RELEASE -> 5.8.16 (*)
|    \--- commons-codec:commons-codec:1.14 -> 1.15
# ...
```

