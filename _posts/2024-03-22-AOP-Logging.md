---
title: AOP를 이용한 Logging 이용하기
author: devforest
date: 2024-03-27 15:00:00 +0000
categories: [spring]
tags: [spring, aop]
---
​
## AOP란
​
- 스프링 AOP (Aspect Oriented Programming) 이란 관점 지향 프로그래밍이라고 부른다. 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화하겠다는 것이다. 여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다. 
​
## AOP 주요 개념

- Aspect : 위에서 설명한 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함
- Target : Aspect 를 적용하는 곳 (클래스, 메서드 등)
- Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점, 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 구체적으로 Advice가 실행될 지점을 정할 수 있음


### 스프링 AOP 특징

- 스프링 빈에만 AOP를 적용 가능
- 스프링 IoC와 연동하여 엔터프라이느 애플리케이션에서 가장 흔한 문제인 중복코드, 프록시 클래스 작성의 번거로움 등에 대한 해결책을 지원하는 것이 목적

<br>

#### 플러그인

~~~
    // AOP
	implementation 'org.springframework.boot:spring-boot-starter-aop'
~~~

build.gradle 파일에 AOP 플러그인을 추가한다.

#### LogConfig 파일 추가

~~~
@Slf4j
@Component
@Aspect
public class LogConfig {

    // com.aop.controller 이하 패키지의 모든 클래스 이하 모든 메서드에 적용
    @Pointcut("execution(* com.study.signin.api.controller..*.*(..))")
    private void cut(){}

    // Pointcut 에 의해 필터링된 경로로 들어오는 경우 메서드 호출 전에 적용
    @Before("cut()")
    public void beforeParameterLog(JoinPoint joinPoint) {
        // 메서드 정보 받아오기
        Method method = getMethod(joinPoint);
        log.info("======= method name = {} =======", method.getName());

        // 파라미터 받아오기
        Object[] args = joinPoint.getArgs();
        if (args.length <= 0) log.info("no parameter");
        for (Object arg : args) {
            log.info("parameter type = {}", arg.getClass().getSimpleName());
            log.info("parameter value = {}", arg);
        }
    }

    // Poincut에 의해 필터링된 경로로 들어오는 경우 메서드 리턴 후에 적용
    @AfterReturning(value = "cut()", returning = "returnObj")
    public void afterReturnLog(JoinPoint joinPoint, Object returnObj) {
        // 메서드 정보 받아오기
        Method method = getMethod(joinPoint);
        log.info("======= method name = {} =======", method.getName());

        log.info("return type = {}", returnObj.getClass().getSimpleName());
        log.info("return value = {}", returnObj);
    }

    // JoinPoint 로 메서드 정보 가져오기
    private Method getMethod(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        return signature.getMethod();
    }
}
~~~

위의 코드를 추가하면 controller를 호출했을 때 로그가 발생되는 것을 확인할 수 있다.

<br>