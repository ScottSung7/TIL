- 로그 보기
```
컨트롤러에서
/.ngrinder/perftest/0_999/11/logs
```

- 자바 1.8이나 11을 이용해야 한다.
- M1과 compatible하게 하려면 Azul의 zulu를 쓰는 것이 좋다.
```
brew install --cask zulu@11
jenv add /Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home
jenv versions
jenv global 11

```
