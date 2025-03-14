1. 자바 런타임 버전 문제
- 해결: 11 -> 17
```
java.lang.UnsupportedClassVersionError:
org/springframework/boot/loader/launch/JarLauncher has been compiled
by a more recent version of the Java Runtime (class file version 61.0),
this version of the Java Runtime only recognizes class file versions up to 55.0 
```

2. 칩셋 호환 문제
- 해결: docker build --platform linux/amd64
```
exec /usr/local/bin/docker-entrypoint.sh: exec format error
```
