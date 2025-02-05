1. 클래스에 붙은 @Validated를 프록시를 통해 활성화
- 그렇지만 강제로 활성화 하는 것이라 원래 클래스에서 @Validated 사라져도 동작한다. (거짓음성)
```
    public LineUpServiceTest() {
        // LocalValidatorFactoryBean을 사용해 @Validated 동작 활성화
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        ProxyFactory proxyFactory = new ProxyFactory(new LineUpService(lineUpRepository, alarmService));
        proxyFactory.addAdvice(new MethodValidationInterceptor(validatorFactory));
        this.lineUpService = (LineUpService) proxyFactory.getProxy();
    }
```
