_[< 03 역할, 명령 및 이벤트 매핑으로 돌아가기](../03-roles-commands-events-mapping/README.md)_

## 모델링 및 개발

![](../img/coffeeshop-ddd-subdomains.jpg)

이제 전체 스토리, 제한된 컨텍스트 및 **충분한** 애그리거트(Aggregate), 명령 및 이벤트가 있습니다. 이제 도메인 모델을 개발하여 크런치된 모델(crunched model)이 올바른지를 증명해야 할 때입니다.

> 디자인 & 개발 모델을 반복적으로, 점진적으로 진행하는 것이 좋습니다. 이 워크숍을 폭포식으로 운영하지 않는 것이 좋습니다. 많은 시간을 보냈지만 마지막 순간에 통제할 수 없는 놀라움을 경험하게 됩니다.

### 예제 별 사양

```java
Feature: Order Americano in seat

  Scenario: Drink Americano, stay in
    Given customer wants to order coffee with the following detail
      | coffee    | quantity | price |
      | Americano | 2        | 80    |
    When the order is confirmed
    Then the total fee should be 160l


```

구체적인 요구 사항 시나리오를 원하십니까? **유일한 방법은 예제에 대해 이야기하는 것입니다.**

실제 문서는 팀이 예에 따라 동일한 이해로 협력할 수 있도록 도와줍니다.

위와 같이 특징과 시나리오를 읽으십시오. 모든 이해관계자가 읽고 이해할 수 있으며, 거기에 기술 용어가 설명되어 있지 않으므로 이해 관계자와 대화하기 좋은 방법입니다.

팀은 예를 확인한 후, 개발자가 이 문서를 활용하여 단위 테스트 코드 골격을 생성하고 이에 따라 이를 구현할 수 있도록 이러한 문서에 협력해야 합니다.

### 단위 테스트 환경 내의 TDD

이 워크숍에서는 cucumber-java를 사용하여 예제를 실행합니다.

```java
package cucumber;


import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;
import org.junit.runner.RunWith;

@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = "json:target/cucumber-report.json",
        glue = "cucumber",
        features = "src/test/resources/")
public class RunCucumberTest {
}

```



### 단위 테스트 코드 스켈레톤(skeleton) 생성

![](../img/run-cucumber-steps.png)



cucumber-java 단계를 실행함으로써 Java 컴파일러는 **기능 : Order_Americao**에 대한 구현 방법이 없다고 불평했습니다.

Cucumber는 이러한 모든 시나리오가 구현되지 않았다고 불평했으며 이 방법을 구현하도록 권장합니다.

```java
아래 코드 조각을 사용하여 누락된 단계를 구현할 수 있습니다.

Given("customer wants to order coffee with the following detail", (io.cucumber.datatable.DataTable dataTable) -> {
    // Write code here that turns the phrase above into concrete actions
    // For automatic transformation, change DataTable to one of
    // E, List<E>, List<List<E>>, List<Map<K,V>>, Map<K,V> or
    // Map<K, List<V>>. E,K,V must be a String, Integer, Float,
    // Double, Byte, Short, Long, BigInteger or BigDecimal.
    //
    // For other transformations you can register a DataTableType.
    throw new cucumber.api.PendingException();
});

When("the order is confirmed", () -> {
    // Write code here that turns the phrase above into concrete actions
    throw new cucumber.api.PendingException();
});

Then("the total fee should be {int}l", (Integer int1) -> {
    // Write code here that turns the phrase above into concrete actions
    throw new cucumber.api.PendingException();
});


Process finished with exit code 0

```



### 코드 스켈레톤에서 도메인 모델 구현

기능: Order_Americano 단계를 이행하는 TDD 스타일 접근 방식입니다.

```java
package cucumber;

import io.cucumber.java8.En;
import solid.humank.coffeeshop.order.commands.CreateOrder;
import solid.humank.coffeeshop.order.models.Order;
import solid.humank.coffeeshop.order.models.OrderId;
import solid.humank.coffeeshop.order.models.OrderItem;
import solid.humank.coffeeshop.order.models.OrderStatus;

import java.math.BigDecimal;
import java.time.OffsetDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class OrderAmericanoSteps implements En {

    CreateOrder cmd;
    Order createdOrder;

    public OrderAmericanoSteps() {

        Given("customer wants to order coffee with the following detail", (io.cucumber.datatable.DataTable dataTable) -> {
            List<Map<String, String>> testData = dataTable.asMaps(String.class, String.class);
            Map<String, String> sample = testData.get(0);

            String productId = sample.get("coffee");
            int qty = Integer.valueOf(sample.get("quantity"));
            BigDecimal price = new BigDecimal(sample.get("price"));

            List<OrderItem> items = new ArrayList<>();
            items.add(new OrderItem(productId, qty, price));
            cmd = new CreateOrder(new OrderId(1, OffsetDateTime.now()), "0", OrderStatus.INITIAL, items);

        });

        When("the order is confirmed", () -> createdOrder = Order.create(cmd));

        Then("the total fee should be {int}l", (Integer int1) -> {
            assertEquals(createdOrder.totalFee().longValue(), int1.longValue());
        });
    }
}


```

## 포트 어댑터(Port-adapter) 개념으로 각 마이크로 서비스 설계

![image](../img/implementation.png)

> 유명한 Port-Adapter 패턴은 마이크로 서비스 개발을 위한 최고의 제품군입니다. 핵심 도메인 문제에 집중하고 필요에 따라 인프라 또는 통신 도구를 전환하십시오.

![image](../img/orderdomain.png)

>이 워크숍 데모에서는 주문 도메인 객체를 설계하고 AWS 서비스를 활용하여 지속적, http 요청 수락 및 처리를 위한 핸들러, 이벤트 전파를 수행하십시오.

## Amazon EventBridge Event를 통합 이벤트로 사용

람다 함수를 쉽게 내보내서(Export) 들어오는 명령을 받아들이고 몇 가지 작업을 수행 할 수 있습니다.
현재 도메인에서 교차 경계(Cross boundary) 이벤트가 발생한 경우 다른 도메인 서비스를 직접 호출하지 말고 교차 도메인 이벤트를 게시하십시오. AWS에서는 EventBridge Event를 사용하는 것이 가장 적절합니다. 거의 실시간에 가까운 이벤트, 고성능 및 확장 가능성이 뛰어난 이벤트입니다.

## 쓰기 모델/읽기 모델 영구 리포지토리로 DynamoDB 사용

도메인 전문가와 함께 모델을 캡처하면 먼저 모델 쓰기를 설계하고 쿼리 사용 모델 읽기를 생성 할 수 있습니다.


## 추가 정보

- Vernon, Vaughn. “Ch. 7, Event Storming.” Domain-Driven Design Distilled, Addison-Wesley, 2016. - https://www.amazon.com/Domain-Driven-Design-Distilled-Vaughn-Vernon/dp/0134434420
- Brandolini, Alberto. Introducing EventStorming. Leanpub, to be released, eventstorming.com/ - https://leanpub.com/introducing_eventstorming
- Brandolini, Alberto. “Ziobrando’s Lair.” Introducing Event Storming, Nov. 2013, ziobrando.blogspot.de/2013/11/introducing-event-storming.html.
- Brandolini, Alberto. Event Storming Recipes. SlideShare, 21 June 2014, de.slideshare.net/ziobrando/event-storming-recipes.
- Rayner, Paul. Event Storming. SlideShare, 26 May 2017, [www.slideshare.net/AgileDenver/event-storming-76390807](http://www.slideshare.net/AgileDenver/event-storming-76390807).
- Brandolini, Alberto. Model Storming. SlideShare, 19 Sept. 2013, [www.slideshare.net/ziobrando/model-storming](http://www.slideshare.net/ziobrando/model-storming).
- Brandolini, Alberto. 50.000 Orange Stickies Later, 7 November 2018, https://www.youtube.com/watch?v=NGXl1D-KwRI
- Business Rules, https://medium.com/plexiti/business-rules-367e430ee168
- How to use Example Mapping & Event Storming, https://hiptest.com/blog/bdd/how-to-use-example-mapping-event-storming/
- What is the Aggregate, https://twitter.com/mathiasverraes/status/1141242508892155904?s=20
- How to monitor Domain Events for Product management, https://xebia.com/blog/eventstorming-and-how-to-monitor-domain-events-for-product-management/



## 특별한 감사를 전합니다

**Jenson Lee** , plays the role as coffee shop owner

**Eason Kuo**, Core team member from Domain Driven Design Taiwan Community

**Arthur Chang** , collaborate design & run the workshop, Co-founder from Domain Driven Design Taiwan Community

**Kenny Baas-Schwegler** , discuss the aggregate definition and ES workshop running experience sharing

[다음 : 05 AWS CDK로 애플리케이션 배포 >](../05-deploy-applications-by-cdk/README.md)
