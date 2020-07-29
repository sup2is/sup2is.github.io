---
layout: post
title: "Java Object Mapping"
tags: [NoSQL, MapStruct, JMapper, Orika]
date: 2020-07-29
comments: true
---



<br>

# OverView

JPA 기반의 API 서버를 구축할때 엔티티를 그대로 반환하는 것은 보안, 성능상 위험할 수 있다. 때문에 DTO(Data Transfer Object) 라는 별도의 값 엔티티를 만들어서 사용한다. DTO를 생성하는 방법은 여러가지인데 정적 팩터리 메서드를 제공해서 사용하는 방법, ModelMapper, Orika, Jmapper, Mapstruct 같은 별도의 라이브러리를 사용하는 방법 등등 여러가지가 있다. 이번시간에는 차례대로 Mapstruct, Jmapper, Orika를 사용해서 조금 더 쉽게 엔티티를 DTO로 변환하는 방법에 대해서 알아보도록 하겠다.

참고로 이 순서는 baeldung에서 [java-performance-mapping-frameworks](https://www.baeldung.com/java-performance-mapping-frameworks) 이라는 포스팅을 통해서 가장 성능이 좋은 mapping 라이브러리 3개를 선정했다. 관심이 있다면 참고해도 좋을 것 같다.



# 시작하기 전에

`Order`라는 객체 내부에는 `Customer`와 `Address` 타입의 멤버변수를 갖고 있다. 클라이언트에게는 단순히 flat된 형태로 보내야하는 요구사항이 있어서 `OrderDto` 타입으로 변환해야 한다는 간단한 시나리오가 존재한다.

**Order.java**

```java
package me.sup2is;

public class Order {

    private Customer customer;
    private Address billingAddress;
    private Address shippingAddress;

    public Order() {
    }

    public Order(Customer customer, Address billingAddress, Address shippingAddress) {
        this.customer = customer;
        this.billingAddress = billingAddress;
        this.shippingAddress = shippingAddress;
    }

// getter and setter
}
```

**Customer.java**

```java
package me.sup2is;

public class Customer {
    private String name;

    public Customer() {
    }


    public Customer(String name) {
        this.name = name;
    }


// getter and setter
}
```

**Address.java**

```java
package me.sup2is;

public class Address {

    private String street;
    private String city;

    public Address() {
    }

    public Address(String street, String city) {
        this.street = street;
        this.city = city;
    }

// getter and setter
}


```

**OrderDto.java**

```java
package me.sup2is;

public class OrderDto {

    private String customerName;
    private String shippingStreetAddress;
    private String shippingCity;
    private String billingStreetAddress;
    private String billingCity;

    public OrderDto() {
    }

    public OrderDto(String customerName, String shippingStreetAddress, String shippingCity, String billingStreetAddress, String billingCity) {
        this.customerName = customerName;
        this.shippingStreetAddress = shippingStreetAddress;
        this.shippingCity = shippingCity;
        this.billingStreetAddress = billingStreetAddress;
        this.billingCity = billingCity;
    }

//getter and setter

}
```

이제 MapStruct, JMapper, Orika를 사용해서 객체 매핑을 구현해보도록 하겠다.

# MapStruct

MapStruct는 apache 2.0 라이센스를 갖고 있고 2020년 7월 기준으로 스타 약 2.8k를 보유하고 있는 대표적인 매핑 라이브러리중 하나이다. 자세한 정보는 [https://github.com/mapstruct/mapstruct](https://github.com/mapstruct/mapstruct)에서 확인할 수 있다.

MapStruct는 java 애너테이션 기반의 매핑 프레임워크이고 다음과 같이 인터페이스를 선언하는 방식으로 사용한다.

```java
@Mapper
public interface CarMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}
```

**인터페이스를 선언함과 동시에 MapStruct가 자동으로 컴파일타임에 구현체를 만들어 실제 사용 가능한 상태로 만든다.**

MapStruct의 장점 세가지에 대해 소개한다.

1. **리플렉션을 사용하지 않고 메서드 호출을 사용해서 빠른 실행**
2. **컴파일 타임에 타입을 검사하고 명시적이다.**
3. **컴파일 타임에 잘못된 정보를 알려준다.**

간단한 엔티티 클래스들을 생성하고 MapStruct를 사용해서 매핑을 구현해보자.



## MapStruct 사용하기

먼저 의존성을 다음과 같이 추가한다.

```xml
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>1.3.1.Final</version>
        </dependency>

...

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>1.3.1.Final</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

가장 기본적인 방법으로 `@Mapper` 애너테이션과 `@Mapping` 애너테이션을 혼합한 형태로 사용 가능하다.

**MapStructOrderMapper.java**

```java
package me.sup2is.modelmapper;

import org.mapstruct.InheritInverseConfiguration;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.factory.Mappers;

@Mapper
public interface MapStructOrderMapper {

    MapStructOrderMapper INSTANCE = Mappers.getMapper(MapStructOrderMapper.class);

    @Mapping(source = "customer.name", target = "customerName")
    @Mapping(source = "shippingAddress.street", target = "shippingStreetAddress")
    @Mapping(source = "shippingAddress.city", target = "shippingCity")
    @Mapping(source = "billingAddress.street", target = "billingStreetAddress")
    @Mapping(source = "billingAddress.city", target = "billingCity")
    OrderDto orderToOrderDto(Order order);

    @InheritInverseConfiguration
    Order orderDtoToOrder(OrderDto orderDTO);

}

```

`@Mapper` 애너테이션을 사용해서 이 인터페이스가 MapStruct 객체임을 알리면 MapStruct가 알아서 자동으로 `MapStructOrderMapperImpl.java` 라는 구현체를 만들어낸다.

`@Mapping` 애너테이션을 사용해서 `source`와 `target`을 지정할 수 있다. `target` 옵션의 경우 반환되는 타입인 `OrderDto` 클래스에 매핑할 변수명이되고 `source` 옵션은 파라미터로 제공되는 `Order` 클래스의 매핑될 변수명이된다. 객체간 탐색은 `.`을 사용해서 탐색할 수 있다.

`@InheritInverseConfiguration`을 사용해서 역방향으로 객체를 매핑하는것 역시 가능하다.

준비가완료됐다면 컴파일을 한 뒤에 target 디렉토리에서 `MapStructOrderMapperImpl.class`가 적절하게 생성되었는지 확인해보자.

**자동으로 만들어진 MapStructOrderMapperImpl.class**

```java
package me.sup2is.modelmapper;

public class MapStructOrderMapperImpl implements MapStructOrderMapper {
    public MapStructOrderMapperImpl() {
    }

    public OrderDto orderToOrderDto(Order order) {
        if (order == null) {
            return null;
        } else {
            OrderDto orderDto = new OrderDto();
            orderDto.setBillingStreetAddress(this.orderBillingAddressStreet(order));
            orderDto.setShippingCity(this.orderShippingAddressCity(order));
            orderDto.setShippingStreetAddress(this.orderShippingAddressStreet(order));
            orderDto.setCustomerName(this.orderCustomerName(order));
            orderDto.setBillingCity(this.orderBillingAddressCity(order));
            return orderDto;
        }
    }

    
    ...
        
}
```



## 테스트

객체 매핑이 잘 이뤄지는지 간단한 테스트를 작성해보도록 하자

**MapStructTest.java**

```java
package me.sup2is;

import me.sup2is.modelmapper.*;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class MapStructTest {

    @Test
    public void basic_mapping() {
        //given
        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);

        //when
        OrderDto orderDTO = MapStructOrderMapper.INSTANCE.orderToOrderDto(order);

        //then
        assertEquals(billingAddress.getCity(), orderDTO.getBillingCity());
        assertEquals(billingAddress.getStreet(), orderDTO.getBillingStreetAddress());
        assertEquals(shippingAddress.getCity(), orderDTO.getShippingCity());
        assertEquals(shippingAddress.getStreet(), orderDTO.getShippingStreetAddress());
        assertEquals(customer.getName(), orderDTO.getCustomerName());

    }

    @Test
    public void inverse() {
        //given
        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);
        OrderDto orderDTO = MapStructOrderMapper.INSTANCE.orderToOrderDto(order);

        //when
        Order toOrder = MapStructOrderMapper.INSTANCE.orderDtoToOrder(orderDTO);

        //then
        assertEquals(orderDTO.getBillingCity(), toOrder.getBillingAddress().getCity());
        assertEquals(orderDTO.getBillingStreetAddress(), toOrder.getBillingAddress().getStreet());
        assertEquals(orderDTO.getCustomerName(), toOrder.getCustomer().getName());
        assertEquals(orderDTO.getShippingCity(), toOrder.getShippingAddress().getCity());
        assertEquals(orderDTO.getShippingStreetAddress(), toOrder.getShippingAddress().getStreet());
    }

}

```

양방향으로 매핑이 잘 되는것을 확인할 수 있다. MapStruct에 대한 자세한 정보는 [https://mapstruct.org/documentation/dev/reference/html/](https://mapstruct.org/documentation/dev/reference/html/)에서 확인할 수 있다.

# JMapper

두번째로 JMapper를 소개한다. JMapper 역시 오픈소스 프로젝트이고 apache 2.0 라이센스를 갖고 있다. JMapper에 대한 자세한 정보는 [https://github.com/jmapper-framework/jmapper-core](https://github.com/jmapper-framework/jmapper-core) 에서 확인할 수 있다.

JMapper는 애너테이션기반, xml 기반, api 형태로 다양한 매핑 방법을 지원한다.

**Annotation**

```java
class Destination{                      class Source{
    @JMap
    private String id;                      private String id;
    @JMap("sourceField")                    
    private String destinationField;        private String sourceField;
    private String other;                   private String other;

    // getters and setters...               // getters and setters...
}                                       }
```

**XML**

```xml
<jmapper>
  <class name="it.jmapper.bean.Destination">
    <attribute name="id">
      <value name="id"/>
    </attribute>
    <attribute name="destinationField">
      <value name="sourceField">
    </attribute>
  </class>
</jmapper>
```

**API**

```java
JMapperAPI jmapperAPI = new JMapperAPI()
    .add(mappedClass(Destination.class)
             .add(attribute("id")
                     .value("id"))
             .add(attribute("destinationField")
                     .value("sourceField")));
```

아래는 JMapper가 제공하는 기능에 대해 나열했다.

- [create](https://github.com/jmapper-framework/jmapper-core/wiki/getDestination-method) and [enrich](https://github.com/jmapper-framework/jmapper-core/wiki/getDestination-method) target objects
- apply a [specific logic](https://github.com/jmapper-framework/jmapper-core/wiki/Enumerations) to the mapping
- perform complex and recursive mapping
- manage [implicit](https://github.com/jmapper-framework/jmapper-core/wiki/Implicit-Relations) and [explicit](https://github.com/jmapper-framework/jmapper-core/wiki/Explicit-Relations) relationships of various types
- implement [explicit conversions](https://github.com/jmapper-framework/jmapper-core/wiki/Explicit-Conversions)
- configure the mapping using the annotation, xml and api
- apply [inherited configurations](https://github.com/jmapper-framework/jmapper-core/wiki/Inheritance-examples)
- write quick and simple xml configuration file, through a series of [utility methods](https://github.com/jmapper-framework/jmapper-core/wiki/Xml-Hander)
- define a custom [bytecode generator](https://github.com/jmapper-framework/jmapper-core/wiki/Bytecode-Generator)



## JMapper 사용하기

JMapper를 사용하기위해 의존성을 추가해준다.

```xml
        <dependency>
            <groupId>com.googlecode.jmapper-framework</groupId>
            <artifactId>jmapper-core</artifactId>
            <version>1.6.0.1</version>
        </dependency>
```

앞에서 설명한것처럼 JMapper를 사용하는 방법은 총 세가지인데 API를 사용하는 방법을 먼저 알아보도록 하겠다. JMapper에서 제공하는 `JMapperAPI`를 사용해서 다음과 같이 객체 매핑을 구현할 수 있다.

**JMapperTest.java**

```java
package me.sup2is;

import com.googlecode.jmapper.JMapper;
import com.googlecode.jmapper.api.JMapperAPI;
import me.sup2is.modelmapper.*;
import org.junit.Test;

import static com.googlecode.jmapper.api.JMapperAPI.attribute;
import static org.junit.Assert.assertEquals;

public class JMapperTest {
    
    @Test
    public void jmapper_using_api() {
        //given
        JMapperAPI jMapperAPI = new JMapperAPI()
                .add(JMapperAPI.mappedClass(OrderDto.class)
                        .add(attribute("customerName").value("${customer.name}"))
                        .add(attribute("shippingStreetAddress").value("${shippingAddress.street}"))
                        .add(attribute("shippingCity").value("${shippingAddress.city}"))
                        .add(attribute("billingStreetAddress").value("${billingAddress.street}"))
                        .add(attribute("billingCity").value("${billingAddress.city}")));

        JMapper<OrderDto, Order> orderMapper = new JMapper<>(OrderDto.class, Order.class, jMapperAPI);
        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);

        //when
        OrderDto orderDto = orderMapper.getDestination(order);

        //then
        assertEquals(billingAddress.getCity(), orderDto.getBillingCity());
        assertEquals(billingAddress.getStreet(), orderDto.getBillingStreetAddress());
        assertEquals(shippingAddress.getCity(), orderDto.getShippingCity());
        assertEquals(shippingAddress.getStreet(), orderDto.getShippingStreetAddress());
        assertEquals(customer.getName(), orderDto.getCustomerName());
    }
}
```

애너테이션 기반으로도 다음과 같이 사용할 수 있다. 별도의 `JMapperOderDto` 라는 클래스를 만들어서 `@JMap` 애너테이션으로 매핑될 필드명을 지정해줄 수 있다.

```java
package me.sup2is.modelmapper;

import com.googlecode.jmapper.annotations.JMap;

public class JMapperOrderDto {

    @JMap("${customer.name}")
    private String customerName;
    @JMap("${shippingAddress.street}")
    private String shippingStreetAddress;
    @JMap("${shippingAddress.city}")
    private String shippingCity;
    @JMap("${billingAddress.street}")
    private String billingStreetAddress;
    @JMap("${billingAddress.city}")
    private String billingCity;

//getter and setter
    
}
```

실제 사용방법은 아래와 같다.

**JMapperTest.java**

```java

    @Test
    public void jmapper_using_annotation() {
        JMapper<JMapperOrderDto, Order> orderMapper = new JMapper<>(JMapperOrderDto.class, Order.class);
        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);

        //when
        JMapperOrderDto orderDto = orderMapper.getDestination(order);

        //then
        assertEquals(billingAddress.getCity(), orderDto.getBillingCity());
        assertEquals(billingAddress.getStreet(), orderDto.getBillingStreetAddress());
        assertEquals(shippingAddress.getCity(), orderDto.getShippingCity());
        assertEquals(shippingAddress.getStreet(), orderDto.getShippingStreetAddress());
        assertEquals(customer.getName(), orderDto.getCustomerName());
    }
```

추가적으로 JMapper에는 **Relational Mapping** 이라는 기능이 있는데 아래와같이 `User` 객체 한개로 `UserDto1`, `UserDto2`를 동시에 매핑하게 해주는 기술이다.

```java
public class User {
    private long id;    
    private String email;
}

public class UserDto1 {  
    private long id;
    private String username;
}

public class UserDto2 {
    private long id;
    private String email;
}

```

```java
@Test
public void givenUser_whenUseApi_thenConverted(){
    JMapperAPI jmapperApi = new JMapperAPI()
      .add(mappedClass(User.class)
      .add(attribute("id")
        .value("id")
        .targetClasses(UserDto1.class,UserDto2.class))
      .add(attribute("email")
        .targetAttributes("username","email")
        .targetClasses(UserDto1.class,UserDto2.class)));
    
    RelationalJMapper<User> relationalMapper = new RelationalJMapper<>
      (User.class,jmapperApi);
    User user = new User(1L,"john@test.com");
    UserDto1 result1 = relationalMapper
      .oneToMany(UserDto1.class, user);
    UserDto2 result2 = relationalMapper
      .oneToMany(UserDto2.class, user);
 
    assertEquals(user.getId(), result1.getId());
    assertEquals(user.getEmail(), result1.getUsername());
    assertEquals(user.getId(), result2.getId());
    assertEquals(user.getEmail(), result2.getEmail());            
}
```

`relationalMapper.oneToMany()` 메서드를 사용해서 두가지 형태의 Dto를 매핑하는 것을 확인할 수 있다. JMapper에 더욱 자세한 내용은 [https://jmapper-framework.github.io/jmapper-core/](https://jmapper-framework.github.io/jmapper-core/)에서 확인할 수 있다. JMapper는 두번째로 빠른 속도를 갖고 있지만 점유율은 매우 낮은 편이다.

# Orika

마지막으로 Orika에 대해서 알아보도록 하겠다. Orika도 apache 2.0 라이센스 오픈소스 프로젝트이다. 한국에서는 많이 사용하지 않지만 점유율이 꽤 높은편에 속한다. [https://github.com/orika-mapper/orika](https://github.com/orika-mapper/orika)

다음은 Orika가 제공하는 기능들이다.

- Map complex and deeply structured objects
- "Flatten" or "Expand" objects by mapping nested properties to top-level properties, and vice versa
- Create mappers on-the-fly, and apply customizations to control some or all of the mapping
- Create converters for complete control over the mapping of a specific set of objects anywhere in the object graph--by type, or even by specific property name
- Handle proxies or enhanced objects (like those of Hibernate, or the various mock frameworks)
- Apply bi-directional mapping with one configuration
- Map to instances of an appropriate concrete class for a target abstract class or interface
- Map POJO properties to Lists, Arrays, and Maps

## Orika 사용하기

의존성을 추가해준다.

```xml
        <dependency>
            <groupId>ma.glasnost.orika</groupId>
            <artifactId>orika-core</artifactId>
            <version>1.5.4</version>
        </dependency>
```

다른 라이브러리들과 비슷하게 사용하기때문에 크게 어려운 점은 없을 것 같다.

**OrikaTest.java**

```java
package me.sup2is;

import ma.glasnost.orika.BoundMapperFacade;
import ma.glasnost.orika.MapperFacade;
import ma.glasnost.orika.MapperFactory;
import ma.glasnost.orika.converter.ConverterFactory;
import ma.glasnost.orika.impl.DefaultMapperFactory;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class OrikaTest {

    @Test
    public void orika_mapping() {
        //given
        MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
        mapperFactory.classMap(OrderDto.class, Order.class)
                .field("customerName","customer.name")
                .field("shippingStreetAddress", "shippingAddress.street")
                .field("shippingCity", "shippingAddress.city")
                .field("billingStreetAddress", "billingAddress.street")
                .field("billingCity", "billingAddress.city")
                .register();


        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);

        //when
        MapperFacade mapperFacade = mapperFactory.getMapperFacade();
        OrderDto orderDto = mapperFacade.map(order, OrderDto.class);

        //then
        assertEquals(billingAddress.getCity(), orderDto.getBillingCity());
        assertEquals(billingAddress.getStreet(), orderDto.getBillingStreetAddress());
        assertEquals(shippingAddress.getCity(), orderDto.getShippingCity());
        assertEquals(shippingAddress.getStreet(), orderDto.getShippingStreetAddress());
        assertEquals(customer.getName(), orderDto.getCustomerName());
    }

```

양방향 매핑, 매핑과정에서 특정 로직을 수행하고싶다면 별도의 `Converter`를 생성할 수 있다.

**MyOrikaConverter.java**

```java
package me.sup2is;

import ma.glasnost.orika.MappingContext;
import ma.glasnost.orika.converter.BidirectionalConverter;
import ma.glasnost.orika.metadata.Type;

public class MyOrikaConverter extends BidirectionalConverter<Order, OrderDto> {
    @Override
    public OrderDto convertTo(Order source, Type<OrderDto> destinationType, MappingContext mappingContext) {
        return new OrderDto(source.getCustomer().getName(),
                source.getShippingAddress().getStreet(),
                source.getShippingAddress().getCity(),
                source.getBillingAddress().getStreet(),
                source.getBillingAddress().getCity());
    }

    @Override
    public Order convertFrom(OrderDto source, Type<Order> destinationType, MappingContext mappingContext) {
        return new Order(new Customer(source.getCustomerName()),
                new Address(source.getBillingStreetAddress(), source.getBillingCity()),
                new Address(source.getShippingStreetAddress(), source.getShippingCity()));
    }
}

```

이렇게 생성한 Converter를 등록시켜주기만 하면 정상적으로 동작하는것을 확인할 수 있다. `BoundMapperFacade` 를 사용해서 양방향 으로 매핑이 잘 되는지 확인해 보자.

```java
    @Test
    public void orika_map_reverse() {
        //given
        MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
        ConverterFactory converterFactory = mapperFactory.getConverterFactory();
        converterFactory.registerConverter(new MyOrikaConverter());

        Address billingAddress = new Address("경리단길", "서울");
        Address shippingAddress = new Address("광평로", "서울");
        Customer customer = new Customer("sup2is");
        Order order = new Order(customer, billingAddress, shippingAddress);
        BoundMapperFacade<Order, OrderDto> boundMapperFacade = mapperFactory.getMapperFacade(Order.class, OrderDto.class);
        OrderDto orderDto = boundMapperFacade.map(order);

        //when
        Order toOrder = boundMapperFacade.mapReverse(orderDto);

        //then
        assertEquals(orderDto.getBillingCity(), toOrder.getBillingAddress().getCity());
        assertEquals(orderDto.getBillingStreetAddress(), toOrder.getBillingAddress().getStreet());
        assertEquals(orderDto.getCustomerName(), toOrder.getCustomer().getName());
        assertEquals(orderDto.getShippingCity(), toOrder.getShippingAddress().getCity());
        assertEquals(orderDto.getShippingStreetAddress(), toOrder.getShippingAddress().getStreet());
    }
```

테스트가 문제없이 동작하는 것을 확인할 수 있다.

# 마무리

총 세가지의 Object Mapping 라이브러리들을 확인했는데 점유율 순으로 보면 MapStruct가 압도적인 1위를 차지하고있다. 많은 사람들이 사용하기때문에 그만큼 성능도 보장되어있고 새로운 기능이나 버그픽스가 더 빠르게 이루어질것이다. 그리고 실제로 사용했을때도 MapStruct를 사용할때 코드가 가장 깔끔하고 컴파일타임에 거의 웬만한 에러를 감지할 수 있어서 개발 단계에서도 빠르게 사용할 수 있을 것 같다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!
<br>

예제: [https://github.com/sup2is/study/tree/master/java/java-object-mapping](https://github.com/sup2is/study/tree/master/java/java-object-mapping)



**References**

- [https://www.baeldung.com/java-performance-mapping-frameworks](https://www.baeldung.com/java-performance-mapping-frameworks)
- [https://mapstruct.org/documentation/dev/reference/html/](https://mapstruct.org/documentation/dev/reference/html/)
- [https://www.baeldung.com/jmapper](https://www.baeldung.com/jmapper)
- [https://orika-mapper.github.io/orika-docs/](https://orika-mapper.github.io/orika-docs/)
- [https://www.baeldung.com/orika-mapping](https://www.baeldung.com/orika-mapping)