---
layout: post
title:  "How to build Fluent Interfaces in Java [english]"
date:   2015-01-01 16:00
categories: fluent interfaces java
---

<center>*Hey there! I'll show here how to build [Fluent Interfaces][fluent-interface] in Java language. Let's explore this pattern with some examples and why this can be useful in some situations.*</center>

#### What is Domain Specific Languages (DSLs)?

<div class="citacao">
  DSLs are small languages, focused on a particular aspect of a software system. You can't build a whole program with a DSL, but you often use multiple DSLs in a system mainly written in a general
  purpose language
</div>

<div class="citacao-autor-right">
  <a href="http://www.martinfowler.com/books/dsl.html">Martin Fowler</a>
</div>

<br/>
Cool. Let's explore this a little bit more. First of all, there are two kinds of DSL: internal and external. An internal DSL is created with the main language of an application. There is no need of custom compilers and interpreters. A external DSL have a specific syntax and you have to write a parser to process them. For example, you can create a external DSL to someone interact with you application and this person not necessarily have to be a developer.

#### Fluent Interfaces and Method Chaining

Fluent Interfaces is a way to implement internal DSLs. Let's start with this example:

Tradicional code:

{% highlight java %}

  var cars = new List<String>();
  cars.add("Honda Civic");
  cars.add("New beatle");
  cars.add("Ferrari");

{% endhighlight %}

Fluent Interface style:

{% highlight java %}

  var cars = CarsList()
          .add("Honda Civic")
          .add("New beatle")
          .add("Ferrari");

{% endhighlight %}

As you saw, is all about *semantic*. I mean, is just a code that call another code, but in a easier and readably way. Probably you already used some API with this style.

JMock is a classical example:

{% highlight java %}

public interface SomeInterface {
    public abstract List<Client> findAllClients();
}

{% endhighlight %}

Unit test method:

{% highlight java %}

@Test
void findlAllClientsTest()  {
    List<Client> allClientsMock = ...
    jmockObject.expects(once()) // 1
      .method("findAllClients")  // 2
      .withNoArguments() // 3
      .will(returnValue(allClientsMock) // 4
     );
 }

{% endhighlight %}

1. the method will be called only once
2. specify the findAllClients() method
3. there is no argument for this method
4. specify the return value

Sometimes your API or service layer become complex. You should think if a Fluent Interface pattern can be useful for you or your team.

Tradicional style:

{% highlight java %}

void makeAOrder()  {
    Customer customer = new Customer();
    Car car1 = new Car("blue");
    Car car2 = new Car("yellow");
    Car car3 = new Car("red");

    OrderRequest orderRequest = new OrderRequest();
    orderRequest.setCustomer(customer);
    orderRequest.addCar(car1);
    orderRequest.addCar(car2);
    orderRequest.addCar(car3);
    orderRequest.processRequest();
    orderRequest.sendRequestByEmail();
 }

{% endhighlight %}

Fluent Interface style:

{% highlight java %}

void makeAOrder()  {
      Customer customer = ...
      ArrayList<Car> carList = ...

      new OrderRequest()
        .buyTheseCars(carList)
        .forThisCustomer(customer)
        .processRequest()
        .sendRequestByEmail();
 }

{% endhighlight %}

Sometime ago I created a Java program to get a list of flights in the brazilian airports. I used a Fluent Interface style and the calls were:

{% highlight java %}

public class AppClient {
  public static void main(String[] args) throws Exception {
      new StatusBuilder()
         .giveMeAllFlightsOf(AirlineUtil.TAM_AIRLINE)
         .inTheAirport(AirportUtil.SP_CONGONHAS)
         .arrivingNow()
         .andShowInConsole();
        }
}

{% endhighlight %}

#### Building your own Fluent Interface

*Step 1*: create your domain model

{% highlight java %}
public class Car {
    private String color;
    public Car(String color) {
      this.color = color;
    }
    public String getColor() {
      return color;
    }
    public void setColor(String color) {
      this.color = color;
    }
}

public class Customer {
    private String name;
    public Customer(String name) {
      this.name = name;
    }
    public String getName() {
      return name;
    }
    public void setName(String name) {
      this.name = name;
    }
    @Override
    public String toString() {
      return "Customer [name=" + name + "]";
    }
}
{% endhighlight %}

*Step 2*: Now, your service:

{% highlight java %}
public class MakeOrderService {
    public void processRequest(Customer client, List<Car> carList) {
      System.out.println("Validating the order...OK!");
    }
    public void sendRequestByEmail(Customer client) {
      System.out.println("REQUEST SENT FOR " + client.toString());
    }
}
{% endhighlight %}

*Step 3*: Finally, your fluent API. It's like a [Facade][facade-pattern]. A *semantic facade*.

{% highlight java %}
public class OrderRequest {
    
    private MakeOrderService makeOrderService = new MakeOrderService();
    private Customer customer;
    private List<Car> carList;

    public OrderRequest() { }
    
    public OrderRequest forThisCustomer(Customer customer) {
      this.customer = customer;
      return this;
    }
    
    public OrderRequest buyTheseCars(List<Car> carList) {
      this.carList = carList;
      return this;
    }
    
    public OrderRequest processRequest() {
      makeOrderService.processRequest(customer, carList);
      return this;
    }

    public void sendRequestByEmail() {
      makeOrderService.sendRequestByEmail(customer);
    }
{% endhighlight %}

*Step 4*: Now, you can use this:

{% highlight java %}
    public static void main(String[] args) {

      Customer customer = new Customer("JOAO SILVA");
      Car car1 = new Car("blue");
      Car car2 = new Car("yellow");
      Car car3 = new Car("red");
      ArrayList<Car> carList = new ArrayList<Car>();
      carList.add(car1); carList.add(car2); carList.add(car3);

      new OrderRequest()
        .buyTheseCars(carList)
        .forThisCustomer(customer)
        .processRequest()
        .sendRequestByEmail();
  }
{% endhighlight %}

Well, that's it. You can get the [full code on Github.][fluent-code]

[fluent-code]:https://github.com/acneto/fluent-interface-in-java-tutorial
[fluent-interface]: http://www.martinfowler.com/bliki/FluentInterface.html
[facade-pattern]: http://en.wikipedia.org/wiki/Facade_pattern

