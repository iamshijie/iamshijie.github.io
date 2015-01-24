---
layout: post
title: "Consumer-Driven Contracts Test - Pact & Pacto"
date: 2015-01-21 19:06:36 +0800
comments: true
categories: 
---

实施microservices[^1] (小、松耦合、多服务、多语言、自动部署、易扩展、易集成、分布式...)架构后，测试变成为了一个难题，如何在多语言、多系统、甚至异步与动态消息的情况下进行测试？如何在build(构建)中进行自动化测试？如何进行集成测试[^2]？

很多人会想到一些名词/工具：stub, mock, fake, dummy, spy, [WebMock](WebMock), [vcr](https://github.com/vcr/vcr), [mountebank](http://www.thoughtworks.com/radar/tools/mountebank)...类似名词有很多，这些工具的思路也都类似，即在服务提供方和服务消费者之间，模拟/拦截消费者的请求，伪造一个结果请求结果返回给消费者，就如Martin Fowler说的

>use to stub out parts of a system for testing

他给这些名词一个通用的叫法： TestDouble

>Test Double is a generic term for any case where you replace a production object for testing purposes

replace，替代。即在测试中，对真实的对象进行替代。

但是，这种方式非常有局限性：

1. TestDouble最重要的使用场景之一就是，与一个外部服务(external service)进行交互。但是你使用的double是否真的精确地代表了外部服务？如果外部服务改变了，你的测试又会发生什么？

	一个解决办法是，在自己的测试中使用double，同时定期的运行另外一套测试集，调用真实的外部服务，来查看外部服务的返回，是否与double测试中的相同。在Martin Fowler的blog中，称之为[Tntegration Contract Test](http://martinfowler.com/bliki/IntegrationContractTest.html)。
	
	是不是觉得很麻烦？即使这样，还是需要在一部分测试中真实地调用外部服务。在复杂的多服务系统中(如microservices)，这样做不仅很困难，而且成本非常高(也许你得为这些测试准备一套完整的环境)。

2. TestDouble更适用于测试金字塔中的底层测试。对于UI层面的模拟用户的测试、集成测试，尤其是集成的自动化测试该怎么办呢？在microservices这种架构下，又该怎么办呢？

[Consumer-Driven Contracts](http://thoughtworks.github.io/pacto/patterns/cdc/) Test就应运而生。[Pact](https://github.com/realestate-com-au/pact)和[Pacto](https://github.com/thoughtworks/pacto)都是这种消费者驱动的契约测试的产物。

每个消费者在一个单独的契约中，描述对服务提供者的期望。服务提供方在自己的测试集中，对这些“期望”进行校验。服务提供方在做出任何改变的时候，只要能保证满足消费者不对其产生影响即可。一旦对任何消费者产生影响，在正式发布前就可以知晓，而且只需要与那些会产生的影响的消费者进行协商即可。

这里的Contracts分为三部分[^3]：

1. 提供者契约（Provider contracts）——提供者契约是我们最为熟悉的一种的服务契约，参考 WSDL+XML Schema+WS-Policy。顾名思义，提供者契约是以提供者为中心的。提供者规定了它要提供什么；然后，各消费者便将自己绑定到这个一成不变的契约上。不论消费者实际需要多少功能，消费者接受了提供者契约，就将自己与该提供者的全体功能耦合起来了。


2. 消费者契约（Consumer contracts）——另一方面，消费者契约是对一个消费者的需求更为精确的描述。消费者契约描述了，在一次具体交互场合下，提供者功能中消费者需要的特定部分。消费者契约可被用来标注一个现有的提供者契约，另外消费者契约也有助于发现一个现今尚未规定的提供者契约。

3. 消费者驱动的契约（Consumer-driven contracts）——消费者驱动的契约描述的是服务提供者向其所有当前消费者承诺遵守的约束。一旦各消费者把自己的具体期望告知提供者，消费者驱动的契约就被创建了。在提供者方面创建的约束，确定了一个消费者驱动的契约。若提供者接受了一个消费者驱动的契约，那么它只需保证已有约束仍能得到满足，即可自行改进与修改其服务。


现在我们看看[Pact](https://github.com/realestate-com-au/pact)和[Pacto](https://github.com/thoughtworks/pacto)这两个测试框架是如何实现的。



## Pact

Pact为服务消费者提供了API，用来定义消费者对服务提供方的http请求、以及期望的http响应。这些期望被用在消费者的spec测试中，来mock服务提供方。所有的交互都会被记录下，然后在服务提供方的spec测试中回放，来确保服务提供方确实提供了消费者所期望的响应。

这使得对于一个集成场景，契约的两方都可以快速、独立地进行单元测试。

工作原理[^4] ：

![pact-step1.png](/images/pact-step1.png)

![pact-step2.png](/images/pact-step2.png)

Pact以gem的形式提供，gem地址：[https://rubygems.org/gems/pact](https://rubygems.org/gems/pact)

使用方法：[https://github.com/realestate-com-au/pact#installation](https://github.com/realestate-com-au/pact#installation)

一些最佳实践：[Pact-best-practices](https://github.com/realestate-com-au/pact/wiki/Best-practices)


## Pacto

Pacto包含两部分，request clause和response clause。

Request clause定义了消费者发送给服务提供方的请求中必须包含的信息，这些信息将被用于一个渲染service。通常包含了http头、需要的参数、以及请求体（json形式）。

Response clause定义了服务提供方需要返回给消费者的信息，从而成功完成一次事务。通常包含请求头和请求体（json形式）。

通常通过产生、校验、服务打桩三部分。

使用方法：[https://github.com/thoughtworks/pacto#usage](https://github.com/thoughtworks/pacto#usage)或者这里[http://thoughtworks.github.io/pacto/usage/](http://thoughtworks.github.io/pacto/usage/)

##区别：

1. Pacto仅支持json服务，原生支持Ruby，对于非Ruby项目，提供了[Pacto Server](https://github.com/thoughtworks/pacto#pacto-server-non-ruby-usage)。

	Pact支持Ruby, JVM([pact-jvm](https://github.com/DiUS/pact-jvm)), .net的消费者，Ruby mock server使用了Javascript进行包装。对于非ruby的服务提供方，Pact提供了[Pact Provider Proxy](https://github.com/bethesque/pact-provider-proxy)

2. Pacto不支持多状态码验证（如200、201）。Pact允许在服务提供者不同状态下，生成同样的请求，允许在同样的endpoint上测试不同的http响应码，或者在不同状态下测试相同的资源。

3. Pacto通过录制与现有服务的交互，可以录制契约。不用手工编写，使得契约非常容易创建。

4. Pacto一旦录制完成，契约就是静态的，既可以校验服务消费者，也可以校验服务提供方。

	Pact的契约是动态生成的中间产物，更容易维护[why?](https://github.com/realestate-com-au/pact/wiki/FAQ#why-are-the-pacts-generated-and-not-static)。
	
5. Pact支持正则表达匹配。

6. Pact有[Pact Broker](https://github.com/bethesque/pact_broker)，提供了自动生成的文档、网络图，使得在产品环境和最新版本（消费者和提供者）间进行交叉测试成为可能，从而将消费者和提供者的发布循环解耦。

总结：

1. 契约录制功能，使得当需要对一个现有第三方服务打桩的时候，Pacto成了更好的选择。在这种情况下，服务提供方状态的缺失、正则匹配可能无关紧要，因为你不可能在服务方创建一套数据，而不使用当前测试的接口。

2. 对于服务提供方不还不存在的新项目来说，Pact可能更好，因为消费者的功能可以驱动出服务提供方的需求。





参考资料：

1. [Lightening Talk: Auckland Continuous Delivery Meetup [Sept 2014] - Pact: Consumer-Driven Contract Testing (Integrated Tests Are A Scam)](http://www.slideshare.net/catosplace/lightening-talk-agile-auckland-pact-consumerdriven-contract-testing-integrated-tests-are-a-scam)

2. [realestate.com.au:http://techblog.realestate.com.au/testing-interactions-with-web-services-without-integration-tests-in-ruby/](http://techblog.realestate.com.au/testing-interactions-with-web-services-without-integration-tests-in-ruby/)

3. 微服务测试
	
	[![microservice-testing-summary.png](/images/microservice-testing-summary.png)](http://martinfowler.com/articles/microservice-testing/#agenda)
	
4. [Pact and Pacto](https://github.com/realestate-com-au/pact/wiki/FAQ#how-does-pact-differ-from-pacto)


[^1]: 关于Microservices,参阅[ThoughtWorks技术雷达](http://www.thoughtworks.com/radar/techniques/microservices)和[Martin Fowler的blog](http://martinfowler.com/articles/microservices.html)

[^2]: Martin Flower的[微服务测试](http://martinfowler.com/articles/microservice-testing/#agenda)

[^3]: 这三种契约的描述，摘自InfoQ的文章[用消费者驱动的契约进行面向服务开发](http://www.infoq.com/cn/articles/consumer-driven-contracts)

[^4]: 这两张图片来自[Simplifying Micro-Service testing with Pacts](http://dius.com.au/2014/05/19/simplifying-micro-service-testing-with-pacts/)