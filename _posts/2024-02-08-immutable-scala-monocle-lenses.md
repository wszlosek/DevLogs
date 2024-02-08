---
layout: post
title: "Immutable Scala - Monocle library: Lenses"
description: "Using Lenses to simplify immutable data manipulation"
date: 2024-02-08 19:22:00 +0200
background: '/img/posts/post06.jpg'
image: 'https://ibb.co/zhMM1PG'
---

# Using Lenses to simplify immutable data manipulation

Hi everyone! Today, I want to delve into the topic of immutability in Scala.
Furthermore, I want to introduce you to library that can help you improve writing immutable code, Monocle.

It's a first part of my series about Monocle library. In this post, I'll focus on Lenses, 
which are a powerful tool for simplifying the manipulation of immutable data structures, especially when they are deeply nested.

&nbsp;
## Table of Contents

1. [Functional programming and immutability](#functional-programming-and-immutability)
    1. [The challenges of manipulating immutable data](#the-challenges-of-manipulating-immutable-data)
2. [Lenses](#lenses)
3. [Summary](#summary)

&nbsp;
## Functional programming and immutability

Diving into functional programming feels a bit like getting superpowers for your code-especially with Scala in your toolkit.
Imagine coding in a world where data doesn't play tricks on you by changing when you least expect it. 
Welcome to the world of immutability, where once you create something, it's set in stone (well, in code). 

This isn't just about keeping things tidy; it's about making our coding lives easier. 
No more unexpected changes means debugging is less of a headache, and our programs become more like well-oiled machines that we can trust. 

In Scala, immutability is embraced both in language design and library support, making it a go-to choice for functional programming. 
Here's a simple example to illustrate immutability in Scala:

```scala
val numbers = List(1, 2, 3)
val updatedNumbers = numbers.map(_ * 2) // List(2, 4, 6)
```

In this example, numbers is a list containing integers. When we want to double each number, we use the map function to apply a transformation, resulting in updatedNumbers. 
Notice how numbers remains unchanged after the operation. Instead of altering the original list, we create a new list with the desired changes. 
This approach typifies functional programming's immutability: data structures are not modified in place; new structures are created from existing ones with the necessary modifications.

&nbsp;
### The challenges of manipulating immutable data

Consider a case where you have a complex, deeply nested object, such as a user profile that includes nested objects for address, which in turn includes city and street details. 
In a mutable world, updating the user's street is straightforward—you directly navigate to the street field and change the value. 

However, the immutable Scala universe demands a different approach.

```scala
case class Street(name: String)
case class Address(city: String, street: Street)
case class UserProfile(name: String, age: Int, address: Address)
```

Updating the street in a UserProfile instance requires constructing a new UserProfile object, which includes a new Address object, which itself includes the new Street object. 
This operation, while preserving immutability, can quickly become error-prone:

```scala
val userProfile = UserProfile("Wojciech", 30, Address("ScalaCity", Street("123 Transient St.")))

val updatedUserProfile = userProfile.copy(
  address = userProfile.address.copy(
    street = Street("456 Wandering St.")
  )
)
```

As you can see, even a simple update necessitates a somewhat convoluted process of copying and updating nested structures. 
This verbosity not only obscures the intent of the operation but also increases the risk of errors, 
especially as the depth and complexity of the data structure grow.

This challenge sets the stage for the need for optics in Scala, tools designed to simplify the manipulation of immutable data structures.
Optics, such as Lenses, provide a more elegant and less error-prone way to update nested immutable structures, 
offering a direct line to the data you want to modify without the verbosity and the manual copying.

Popular library, which provides Lenses, is [Monocle](https://www.optics.dev/Monocle/). 
Monocle's documentation is a great place to start if you want to learn more about these concepts. In this post, I'll focus on basic usage of Lenses.


Monocle is published for Scala 2.13.x and 3.x. You can add it to your sbt build with:

```scala
libraryDependencies ++= Seq(
 "dev.optics" %% "monocle-core"  % "3.1.0",
 "dev.optics" %% "monocle-macro" % "3.1.0",
)
```

&nbsp;
## Lenses

Lens is a functional concept that allows you to zoom into a data structure, 
focusing on a particular field to get or update it, without disturbing the immutability of the whole structure

Consider the case of our person, who decided to move from "123 Transient St." to "456 Wandering St.".
Let's see how a Lens can simplify this operation:

```scala
import monocle.Lens
import monocle.macros.GenLens

case class Address(street: String, city: String)
case class Person(name: String, address: Address)

// create a Lens for the Address inside Person
val addressLens: Lens[Person, Address] = GenLens[Person](_.address)
// create a Lens for the Street inside Address
val streetLens: Lens[Address, String] = GenLens[Address](_.street)

val userProfile = UserProfile("Wojciech", 30, Address("ScalaCity", Street("123 Transient St.")))

val updatedProfile = addressLens.composeLens(streetLens).modify(_ => "456 Wandering St.")(userProfile)
```

This operation is quite straightforward. We compose the addressLens with the streetLens to focus directly on the street field of the Address,
and then we simply modify it. The beauty of this approach lies in its simplicity and elegance. 
No explicit copying or nested object construction is required. The lens handles the immutability under the hood,
allowing us to maintain the purity and integrity of our data structures while making our code more readable and maintainable.

Of course, real beauty of Lenses is revealed when you have deeply nested structures.

```scala
import monocle.Lens
import monocle.macros.GenLens

case class Email(value: String)
case class Phone(value: String)
case class ContactDetails(email: Email, phone: Phone)
case class Address(street: String, city: String)
case class AddressHistory(currentAddress: Address, previousAddresses: List[Address])
case class Person(name: String, age: Int, contactDetails: ContactDetails, addressHistory: AddressHistory)

val contactDetailsLens: Lens[Person, ContactDetails] = GenLens[Person](_.contactDetails)
val emailLens: Lens[ContactDetails, Email] = GenLens[ContactDetails](_.email)
val emailValueLens: Lens[Email, String] = GenLens[Email](_.value)

val addressHistoryLens: Lens[Person, AddressHistory] = GenLens[Person](_.addressHistory)
val currentAddressLens: Lens[AddressHistory, Address] = GenLens[AddressHistory](_.currentAddress)
val streetLens: Lens[Address, String] = GenLens[Address](_.street)

val personEmailLens = contactDetailsLens.composeLens(emailLens).composeLens(emailValueLens)
val personCurrentStreetLens = addressHistoryLens.composeLens(currentAddressLens).composeLens(streetLens)

val person = Person(
  "Wojciech",
  23,
  ContactDetails(Email("wojciech@example.com"), Phone("123-456-789")),
  AddressHistory(Address("123 Transient St.", "ScalaCity"), List(Address("101 Initial Way", "BeginnersTown")))
)

val updatedPersonEmail = personEmailLens.modify(_ => "wojciech@newdomain.com")(person)
val updatedPersonAddress = personCurrentStreetLens.modify(_ => "456 Wandering St.")(updatedPersonEmail)

// updatedPersonAddress = Person(
//    Wojciech,
//    23,
//    ContactDetails(Email(value=wojciech@abc.com), Phone(value=123-456-789)), 
//    AddressHistory(Address(456 Wandering St., ScalaCity), [Address(101 Initial Way, BeginnersTown)])
// )
```

&nbsp;
## Summary

The challenges of updating nested, immutable data structures can turn coding sessions into daunting treks. 
Yet, with optics' concept, these tasks transform into more manageable and elegant operations.

Lenses allow us to focus and modify specific parts of our data structures without unwrapping layer after layer manually.

In the next part of this series, I'll delve into the topic of Prisms, another powerful optic provided by Monocle. 
I recommend exploring the possibilities that Monocle offers and trying to incorporate them into your projects!
<br/><br/>

<script src="https://utteranc.es/client.js"
        repo="wszlosek/DevDawn"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
