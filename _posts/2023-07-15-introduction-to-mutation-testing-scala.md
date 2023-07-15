---
layout: post
title: "Introduction to mutation testing in Scala"
description: "Get to know basic concepts of mutation testing"
date: 2023-07-15 11:51:00 +0200
background: '/img/posts/post03.jpg'
image: 'https://ibb.co/qD7vCqz'
---

# Get to know basic concepts of mutation testing

Hello! In this post, I'd like to introduce you to mutation testing in the Scala programming language.
Of course, the methods and frameworks described in this post can be used in other JVM-based languages, such as Java and Kotlin.
There are several mutation testing frameworks available for JVM-based languages. Today, I'll be using the PITest framework.
But not framework itself is the most important. The most important is the idea of mutation testing.

This topic is special to me because I'm interested in mutation testing from a scientific research perspective.

Please keep in mind that this post is just an introduction to the topic. 
In the future, I will be writing about more advanced areas related to mutation testing.

&nbsp;
## Table of Contents

1. [Mutation testing concept](#mutation-testing-concept)
    1. [What is mutation?](#what-is-mutation)
    2. [For and against mutation testing](#for-and-against-mutation-testing)
2. [Mutation testing in JVM-based languages](#mutation-testing-in-jvm-based-languages)
    1. [Sample mutations types](#sample-mutations-types)
    2. [Example of mutation testing in Scala](#example-of-mutation-testing-in-scala)
3. [Summary](#summary)

&nbsp;
## Mutation testing concept

&nbsp;
### What is mutation?
Program-based mutation testing is a technique used to test software units, 
such as classes, methods, and functions, by modifying their source code. 
The modified code, known as a mutant, is a copy of the original code with a small alteration. 
It's important to note that program-based mutation testing is just one type of mutation testing employed in practical programming. 

&nbsp;
### For and against mutation testing
Advantages of mutation testing:
* **enhanced fault detection**: mutation testing can identify faults in the code that may be missed by traditional testing techniques. By introducing small changes (mutations) into the code, it helps uncover potential weaknesses and vulnerabilities.
* **evaluation of test quality**: mutation testing provides a quantitative measure of the effectiveness of the test suite. By measuring the percentage of "killed" mutants (i.e., mutants that are detected as faulty), it helps assess the quality and adequacy of the tests. This information can guide developers in improving their test cases.
* **integration with existing testing and CI/CD practices**: mutation testing can be integrated into existing testing and development practices, including continuous integration and continuous delivery pipelines. This integration allows for automated mutation testing as part of the development workflow, enhancing the overall software quality.

Disadvantages of mutation testing:
* **high computational cost**: mutation testing can be computationally expensive. The process involves generating a large number of mutants and executing the test suite against each mutant. This can require significant computing power and time, especially for large codebase.
* **time-consuming analysis**: analyzing the results of mutation testing can be time-consuming. The process involves identifying surviving mutants (mutants not detected by the test suite) and distinguishing between meaningful mutations and trivial ones. This manual review and interpretation of results can be labor-intensive.
* **limited mutation coverage**: creating mutations that represent all possible faults in the code can be challenging. The effectiveness of mutation testing relies on the ability to generate diverse and meaningful mutations. However, it may not always be possible to create mutations that cover all potential faults, leading to limited mutation coverage.


Optimization of mutation testing is a area of still active research. 
Techniques like machine learning and Bayesian inference are being explored to make mutation testing more efficient and effective. 
As advancements continue, mutation testing may become a more cost-effective and widely adopted technique, particularly in commercial projects.

&nbsp;
## Mutation testing in JVM-based languages
JVM-based languages have a lot of mutation testing frameworks. I would like to mention the most popular of them:
* [PITest](https://pitest.org/)
* [MuJava](https://cs.gmu.edu/~offutt/mujava/)
* [Javalanche](https://github.com/david-schuler/javalanche/)
* [Jester](https://jester.sourceforge.net/)

In relation to the topic of this post, I will show how to use mutation testing in Scala programming language. 
To do this, I will use the PITest framework. It's the most popular mutation testing framework for JVM-based languages. 
PiTest is a fast, easy to use mutation testing system, it's a still active project and it has a lot of features.

A big challenge in mutation testing is creating mutants for operators from the functional programming paradigm. 
Fortunately, PITest supports (at least partially) operators from the functional programming paradigm, 
such as those found in the Scala programming language.

&nbsp;
### Sample mutations types

PITest generates various types of mutations. 
Here are some example types of mutations that PITest can generate:

* **conditional operator replacement**: changes conditional operators, such as `==`, `!=`, `<`, `>`, etc., to other operators, such as `<` -> `<=`, `==` -> `!=`, to see if the tests respond correctly to different cases.
* **constant value replacement**: changes constant values, such as numbers, strings, or boolean values, to other values to see if the tests are sensitive to different input data.
* **arithmetic operator replacement**: changes arithmetic operators, such as `+`, `-`, `*`, `/`, to other operators, such as `++`, `--`, to see if the tests handle different computations.
* **statement removal**: removes a single statement from the code to see if the tests are sensitive to the absence of a specific functionality.
* **statement reordering**: changes the order of statements in a code block to see if the tests are resilient to changes in execution order.

In each of reports, PITest provides a detailed description of the mutations that were generated, for example:
```text
replaced boolean return with true for mutation/testing/gradle/init/HotelReservationSystem$$anonfun$findAvailableRooms$1::isDefinedAt
```
or 
```text
negated conditional
```
and many more.

&nbsp;
### Example of mutation testing in Scala
In this section, I will show you how to use PITest in example code written in Scala.
Before you begin, make sure you have the PITest framework installed in your project. 
If you're using Gradle as your build tool, you can add the following lines to your `build.gradle` file (with [gradle-pitest-plugin](https://github.com/szpak/gradle-pitest-plugin)):

```groovy
plugins {
    id 'info.solidsoft.pitest' version '1.9.1' // the latest version
}

pitest {
    targetClasses = ['com.example.*'] // specify the package or class to be tested
    targetTests = ['com.example.*'] // specify the package or class containing your test cases
    // You can configure other options and settings here
}
```

And voilà! Now you can generate PITest reports for your project by running the Gradle task pitest. Simply execute the following command:

```shell
./gradlew pitest
```
This will initiate the mutation testing process using PITest and generate the corresponding reports. 
The reports will provide insights into the effectiveness of your test suite in detecting mutations and highlight any areas where your tests could be improved.
You can find reports in the `build/reports/pitest` directory.

&nbsp;
#### Ex. 1
Let's start with a simple example. 

```scala
class BankAccount(var balance: Double) {
  def deposit(amount: Double): Unit = {
    balance += amount
  }

  def withdraw(amount: Double): Boolean = {
    if (amount > 0 && amount <= balance) {
      balance -= amount
      true
    } else {
      false
    }
  }

  def transfer(amount: Double, recipient: BankAccount): Boolean = {
    if (withdraw(amount)) {
      recipient.deposit(amount)
      true
    } else {
      false
    }
  }

  def checkBalance(): Double = balance
}
```
```scala
class BankAccountSpec extends AnyWordSpec with Matchers {

  "The BankAccount" should {
    "deposit money correctly" in {
      val account = new BankAccount(100.0)
      account.deposit(50.0)
      account.checkBalance() shouldEqual 150.0
    }

    "withdraw money correctly" in {
      val account = new BankAccount(100.0)
      account.withdraw(50.0) shouldEqual true
      account.checkBalance() shouldEqual 50.0
    }

    "return false for invalid withdrawal" in {
      val account = new BankAccount(100.0)
      account.withdraw(150.0) shouldEqual false
      account.checkBalance() shouldEqual 100.0
    }

    "transfer money correctly to another account" in {
      val sender = new BankAccount(100.0)
      val recipient = new BankAccount(50.0)
      sender.transfer(75.0, recipient) shouldEqual true
      sender.checkBalance() shouldEqual 25.0
      recipient.checkBalance() shouldEqual 125.0
    }
  }
}
```

As you can see, I have prepared a simple BankAccount class for performing basic banking operations, along with a set of unit tests to ensure its functionality. 

Below you can see part of the PITest report specifically related to the BankAccount class.

<a href="https://ibb.co/C8GThg1"><img src="https://i.ibb.co/wgmxMV0/Zrzut-ekranu-2023-07-12-o-22-09-19.png" alt="Zrzut-ekranu-2023-07-12-o-22-09-19" border="0"></a>

In the generated report, the green color indicates that the mutant was killed by the test suite. 
This means that the test suite successfully detected the fault introduced by the modified code. 

On the other hand, the red color signifies that the mutant survived, indicating that either the test suite failed to detect the fault or there was no test coverage for that particular case.
In this specific example, two mutants survived, which suggests that the corresponding faults were not detected by the test suite. 

Additionally, you mentioned that there is a case where there is no test coverage, specifically when the amount of money to transfer is greater than the balance of the sender's account or when the amount is less than or equal to zero. 
This implies that there may be potential issues or untested scenarios in those cases.

So let's add new test.

```scala
"return false for invalid transfer" in {
  val sender = new BankAccount(100.0)
  val recipient = new BankAccount(50.0)
    
  sender.transfer(150.0, recipient) shouldEqual false
  sender.transfer(0.0, recipient) shouldEqual false
    
  sender.checkBalance() shouldEqual 100.0
  recipient.checkBalance() shouldEqual 50.0
}
```

Now, let's run `gradle pitest` task again and check the results:

<a href="https://ibb.co/vYw4W8f"><img src="https://i.ibb.co/ZmLzrCv/Zrzut-ekranu-2023-07-12-o-22-17-52.png" alt="Zrzut-ekranu-2023-07-12-o-22-17-52" border="0"></a>

Firstly, now we have full test coverage. Additionally, we killed one more mutant (from line 11) by adding a test case with exactly zero amount of money.
However, we still have one surviving mutant for this line of code because we haven't tested the scenario when the amount of money to transfer is equal to the sender's account balance.
To address this issue, we need to add test cases that cover this scenario.

```scala
val sender = new BankAccount(100.0)
val recipient = new BankAccount(50.0)
sender.transfer(100.0, recipient) shouldEqual true
```

<a href="https://ibb.co/k0BJqyT"><img src="https://i.ibb.co/xGLYzsb/Zrzut-ekranu-2023-07-12-o-22-19-11.png" alt="Zrzut-ekranu-2023-07-12-o-22-19-11" border="0"></a>

Beautiful green color! Now we have full coverage in tests, and we killed all generated mutants.

&nbsp;
#### Ex. 2
Let's explore a more complex example in Scala that involves functional elements.

```scala
class HotelReservationSystem {
  private var bookings: Map[String, Int] = Map.empty

  def makeReservation(roomNumber: String, numberOfGuests: Int): Unit = {
    bookings += (roomNumber -> numberOfGuests)
  }

  def cancelReservation(roomNumber: String): Unit = {
    bookings -= roomNumber
  }

  def getNumberOfGuests(roomNumber: String): Option[Int] = {
    bookings.get(roomNumber)
  }

  def findAvailableRooms(capacity: Int): List[String] = {
    bookings
      .collect {
        case (roomNumber, guests) if guests <= capacity => {
          roomNumber
        }
      }
      .toList
  }

  def processBookings(bookingFunction: (String, Int) => Unit): Unit = {
    bookings.foreach {
      case (roomNumber, guests) => bookingFunction(roomNumber, guests)
    }
  }

  def getRoomDetails(roomNumber: String): Option[(String, Int)] = {
    bookings.get(roomNumber).map(guests => (roomNumber, guests))
  }

  def getTotalNumberOfGuests: Int = {
    bookings.values.sum
  }
}
```

```scala
class HotelReservationSystemSpec extends AnyWordSpec with Matchers with OptionValues {
  
  "HotelReservationSystem" should {
    "make a reservation correctly" in {
      val reservationSystem = new HotelReservationSystem()
      reservationSystem.makeReservation("101", 2)
      reservationSystem.makeReservation("102", 1)

      reservationSystem.getNumberOfGuests("101").value shouldEqual 2
      reservationSystem.getNumberOfGuests("102").value shouldEqual 1
    }

    "cancel a reservation correctly" in {
      val reservationSystem = new HotelReservationSystem()
      reservationSystem.cancelReservation("101")
      reservationSystem.getNumberOfGuests("101") shouldEqual None
    }

    "find available rooms correctly" in {
      val reservationSystem = new HotelReservationSystem()
      reservationSystem.makeReservation("201", 3)
      reservationSystem.makeReservation("202", 2)

      val availableRooms = reservationSystem.findAvailableRooms(2)
      availableRooms should contain only ("202")
    }

    "process bookings correctly" in {
      val reservationSystem = new HotelReservationSystem()
      var totalGuests = 0
      reservationSystem.makeReservation("201", 3)
      reservationSystem.makeReservation("202", 2)
      reservationSystem.processBookings { (_, guests) =>
        totalGuests += guests
      }

      totalGuests shouldEqual 5
    }

    "get room details correctly" in {
      val reservationSystem = new HotelReservationSystem()
      reservationSystem.makeReservation("301", 4)

      val roomDetails = reservationSystem.getRoomDetails("301")
      roomDetails.value shouldEqual ("301", 4)
    }

    "get total number of guests correctly" in {
      val reservationSystem = new HotelReservationSystem()
      reservationSystem.makeReservation("401", 2)
      reservationSystem.makeReservation("402", 3)

      val totalGuests = reservationSystem.getTotalNumberOfGuests
      totalGuests shouldEqual 5
    }
```

<a href="https://ibb.co/Mp49QZW"><img src="https://i.ibb.co/2PQ5HWm/Zrzut-ekranu-2023-07-15-o-12-33-26.png" alt="Zrzut-ekranu-2023-07-15-o-12-33-26" border="0"></a>

The mutation in the cancelReservation method is easy to eliminate with the following test because there are currently no tests for canceling a reservation.


```scala
"cancel a reservation correctly" in {
  val reservationSystem = new HotelReservationSystem()
  reservationSystem.makeReservation("401", 2)
  reservationSystem.makeReservation("501", 2)

  reservationSystem.cancelReservation("402")
  reservationSystem.findAvailableRooms(3) shouldEqual List("401", "501")

  reservationSystem.cancelReservation("401")
  reservationSystem.findAvailableRooms(5) shouldEqual List("501")
}
```

Not bad! We still have five cases remaining, each related to the findAvailableRooms method.

Currently, we haven't considered the conditional `if guests <= capacity` in the `findAvailableRooms` method, so we need to add some test cases to cover it.
Let's fix `find available rooms correctly` test case:
```scala
"find available rooms correctly" in {
  val reservationSystem = new HotelReservationSystem()
  reservationSystem.makeReservation("201", 3)
  reservationSystem.makeReservation("202", 2)

  var availableRooms = reservationSystem.findAvailableRooms(2)
  availableRooms should contain only ("202")

  reservationSystem.makeReservation("603", 10)
      
  availableRooms = reservationSystem.findAvailableRooms(1)
  availableRooms should be (empty)
      
  availableRooms = reservationSystem.findAvailableRooms(9)
  availableRooms should contain only ("201", "202")

  availableRooms = reservationSystem.findAvailableRooms(10)
  availableRooms should contain only ("201", "202", "603")
}
```

<a href="https://ibb.co/DkKwKnV"><img src="https://i.ibb.co/FJ787tx/Zrzut-ekranu-2023-07-15-o-13-11-03.png" alt="Zrzut-ekranu-2023-07-15-o-13-11-03" border="0"></a>

Currently, we have full test coverage and have successfully killed all generated mutants. Great job!

&nbsp;
## Summary

As you can see, mutation testing is a powerful tool that can help you improve the quality of your tests and code. While it's not a silver bullet, it can be a valuable addition to your test suite. Analyzing all generated mutants may not be easy, 
but it often leads to discovering interesting cases that you may not have considered before.
I have presented mutation testing in the context of Scala, 
but it's again worth noting that you can use PITest in other JVM languages as well, not necessarily limited to projects built with Gradle. 

This post serves as a simple introduction to the topic, and I hope it encourages you to explore mutation testing in your own projects.

Please feel free to share your thoughts and questions in the comments section below.

<br/><br/>

<script src="https://utteranc.es/client.js"
        repo="wszlosek/DevDawn"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
