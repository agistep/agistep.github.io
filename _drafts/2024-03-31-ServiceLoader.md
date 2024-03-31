

[java docs](https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/ServiceLoader.html) 에서 Service Loader 를 다음과 같은 한줄로 정의한다.

> A facility to load implementations of a service.  
> 서비스 구현을 로드하는 기능입니다.

서비스 구현을 로드하는 기능이 왜 필요한지 부터 이해할 필요가 있을 것 같다. (여기서 서비스 구현의 의미는 비지니스 로직이 담겨진 구체적 클래스를 의미한다.)

우리가 라면을 만드는 코딩을 한다고 가정해보자.

```
public class 신라면제조 {

    public void make() {
        pourWater();
        boilWater();
        putNoodle();
        boilNoodle();
        putSoup();
        putVegetable();
    }

    public static void main(String[] args) {
        신라면제조 신라면 = new 신라면제조();
        신라면.make();
    }
}
```

나는 방금 신라면을 만드는 코드를 작성했다. 이 요구사항에서 어느날 갑자기 진라면으로 변경했다고 가정해보자. 그렇다면 위에 코드가 어떻게 변할까?

```
public class 진라면제조 {

    public void make() {
        pourWater();
        boilWater();
        putNoodle();
        boilNoodle();
        putSoup();
        putVegetable();
    }

    public static void main(String[] args) {
        진라면제조 진라면 = new 진라면제조();
        진라면.make();
    }
}
```

이렇게 변할 것이다. 이 시점에 머리 좋은 누군가는 인터페이스를 만들 것이다.

```
public interface Noodle {
    void pourWater();
    void boilWater();
    void putNoodle();
    void boilNoodle();
    void putSoup();
    void putVegetable();
}
```

그러면 진라면이든, 신라면이든 **어떤 라면을 만들 것인지는 Noodle Interface 를 사용하는 Client이 결정하게 된다.**

**Client 가 책임을 가지게 되는 것이고, 이것이 만약 라면이 만들어지는 중간 과정에서 어떤 라면인지 결정되게 된다면 Client 는 어떤 라면인지 나중에 알게 될 수 밖에 없다.**


좀더 구체적으로 Noodle Interface 를 활용해 구현된 객체는 다음과 같이 사용하게 될 것이다.

```
public static void main(String[] args) {
        Noodle 신라면 = new 신라면제조();
        신라면.make();

        Noodle 진라면 = new 진라면제조();
        진라면.make();

        Noodle 안성탕면 = new 안성탕면제조();
        안성탕면.make();
    }
```

**new** 키워드에 집중할 필요가 있다. new 라는 키워드를 사용하게 되면 특정 구현체에 의존적이게 된다. 갑자기 만들어야 하는 라면이 변경되어 바꿔야 한다면 new 코드를 변경해야 될 것이다. **이것은 필연적으로 코드에 변경을 가해야만 한다.**그리고 그것은 컴파일 시점에 결정되게 된다. 만약 런타임 시점에 변경이 필요할 때는 어떻게 해야될까?****

**ServiceLoader(서비스로더)는 구현의 책임을 최대한 Client에게 넘겨주는 역할을 하는 동시에 런타임 시점에 구현을 결정하게 만들어 준다.**

이때 등장하는 것이 바로 ServiceLoader 이다.

### 이제부터는 ServiceLoader 가 무엇인지 살펴보자.

_javadoc에 있는 예시를 그대로 가져왔으니, 좀더 자세한 내용은 [java docs](https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/ServiceLoader.html) 에서 확인하실 수 있습니다._

등장인물로는 Service, ServiceLoader, 서비스 제공자(Factory, Proxy), 클래스 경로에 서비스 제공자 배치하는 리소스 디렉토리가 등장하게 된다.

아래 Codec 이라는 Service 가 있다고 가정한다.

```
package com.example;
public interface CodecFactory {
	Encoder getEncoder(String encodingName);
	Decoder getDecoder(String encodingName);
}
```

다음 코드는 서비스에 대한 서비스 로더를 얻은 CodecFactory 다음 해당 반복자(향상된 for 루프에 의해 자동으로 생성됨)를 사용하여 위치한 서비스 공급자의 PNG 인스턴스를 생성합니다.

```
ServiceLoader<CodecFactory> loader = ServiceLoader.load(CodecFactory.class);
     for (CodecFactory factory : loader) {
         Encoder enc = factory.getEncoder("PNG");
         if (enc != null)
             ... use enc to encode a PNG file
             break;
         }
```

여기서 CodeFactory 는 서비스 제공자의 역할을 하게 됩니다.

CodeFactory 의 구현체로는 2가지 있습니다.

```
provides com.example.CodecFactory with com.example.impl.StandardCodecs;
provides com.example.CodecFactory with com.example.impl.ExtendedCodecsFactory;
```

여기서 PNG encoder 를 사용해야 하는데, Standard, extended 둘 중에 하나를 사용해야 한다면 어딘가에는 명시되어 있어야 한다. 여기서 **클래스 경로에 서비스 제공자 배치하는 리소스 디렉토리**가 등장한다. resources/META-INF/services/com.example.CodecFactory  라는 파일을 생성하고, CodecFactory에 대한 구현체를 어떤 것을 사용할 것인지 명시합니다.  
첫 줄에 com.example.impl.StandardCodecs # Standard codecs 이렇게 명시하게 되면 StandardCodecs 의 PNG 의 encoder 를 사용하게 된다.

이렇게 ServiceLoader 는 resources/META-INF/services/com.example.CodecFactory 의 구현체를 변경함으로써 서비스의 구현을 코드 변경없이 런타임시점에 변경할 수 있다.

**구현체의 주입을 외부에서 주입시켰다.** Spring Framework 에서도 비슷한 것이 있지 않았던가? 맞다. Spring Framework 의 IoC(Inversion Of Control)에서도 ServiceLoader 가 사용되었다. 다만 우리가 앞서 언급한 resources/META-INF/services/... 이 형태가 아닌 spring.factories 로 사용되었다.

어떻게 SpringBoot 에서 사용되었는지 코드로 이해해보자.

### SpringBoot 에서 spring.factories 는 어떻게 활용되는가?

SpringApplication.class 는 SpringBoot 의 시작지점이 된다.

```
public ConfigurableApplicationContext run(String... args) {
		...
		SpringApplicationRunListeners listeners = getRunListeners(args); // Look at this line!
		listeners.starting(bootstrapContext, this.mainApplicationClass);
		...
		return context;
	}
```

여기서 getRunListener(args) 를 살펴보면, 다음과 같은 코드를 찾아볼 수 있다.

```
private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args),
				this.applicationStartup);
	}
```

getSpringFactoriesInstances는 다음과 같이 되어있다.

```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

드디어 내가 찾고자 헀던 SpringFactoriesLoader 가 발견했다. 해당 클래스를 자세히 살펴보면, META-INF/spring.factories 의 위치에서 특정 클래스를 Load 해온다는 사실을 알 수 있다.

spring.factories 파일을 검색해보면 많은 파일들이 나오는데, org.springframework.boot 의 spring.factories 를 보면 SpringBoot가 컴파일되면서 어떤 구현 클래스가 초기에 load 되어야 하는지 알 수 있다. ServiceLoader 를 활용하여 SpringBoot 에서 사용되었다.

이 말은 SpringBoot의 구현체를 SpringBoot 를 사용하는 내가 변경할 수도 있다는 이야기로도 이어진다. 자세한 내용은 어디서 찾아볼 수 있다.

[https://docs.spring.io/spring-boot/docs/2.0.0.M4/reference/html/boot-features-developing-auto-configuration.html](https://docs.spring.io/spring-boot/docs/2.0.0.M4/reference/html/boot-features-developing-auto-configuration.html)
