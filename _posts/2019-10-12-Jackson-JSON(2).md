---
layout: post
title: "Jackson JSON Tutorial(2)"
subtitle: 'Collection/Array, Polymorphic Type'
author: "Shaun"
header-style: text
tags: ["Java Utils"]
---

Jackson is a very popular java library to serialize/deserialize JSON. 
This tutorial illustrates common solutions while marshalling and unmarshalling JSON.

[-> (1) Jackson Annotations](http://shaunlu.com/2019/09/26/Jackson-JSON(1)/)  
[-> (2) Collection/Array, Polymorphic Type](http://shaunlu.com/2019/10/12/Jackson-JSON(2)/)  
[-> (3) Enum, Date, JsonNode](http://shaunlu.com/2019/10/25/Jackson-JSON(3)/)  

You can find all examples at [https://github.com/shaunlu00/java-tutorial-jackson](https://github.com/shaunlu00/java-tutorial-jackson)

---
## 1 Collection, Array
Firstly let's define a class
```java
public class UserPermission {
    private String id;
}
```
Read json string to array
```java
@Test
public void testJsonCollection() throws JsonProcessingException {
    String str = "[{\"id\":\"w\"}, {\"id\":\"r\"}]";
    UserPermission[] permissions = objectMapper.readValue(str, UserPermission[].class);
    System.out.println("This first permission is " + permissions[0].getId());
}
```
Read json string to collection
```java
@Test
public void testJsonCollection() throws JsonProcessingException {
    String str = "[{\"id\":\"w\"}, {\"id\":\"r\"}]";
    CollectionType collectionType = objectMapper.getTypeFactory().constructCollectionType(List.class, UserPermission.class);
    List<UserPermission> permissions = objectMapper.readValue(str, collectionType);
    System.out.println("This first permission is " + permissions.get(0).getId());
}
```

---
## 2 Polymorphic Type Handling
- @JsonTypeInfo - indicates details of what type information to include in serialization
- @JsonSubTypes - indicates sub-types of the annotated type
- @JsonTypeName - defines a logical type name to use for annotated class

We can use `@JsonTypeInfo` to add a new property about class type in serialization. And in deserialization, 
this class type can be used for polymorphic handling:  
```java
// Include Java class name as JSON property "@class"
@JsonTypeInfo(use=Id.CLASS, include=As.PROPERTY, property="@class")
// Include logical type name defined in impl classes as JSON property "@type"  
@JsonTypeInfo(use=Id.NAME, include=As.PROPERTY, property="@type")
@JsonSubTypes({
        @JsonSubTypes.Type(value = Subclass1.class, name = "subclass1"),
        @JsonSubTypes.Type(value = Subclass2.class, name = "subclass2")
})
class Mainclass {}

@JsonTypeName("subclass1")
class Subclass1 extends Mainclass {}

@JsonTypeName("subclass2")
class Subclass2 extends Mainclass {}
```


Let's take a look at the detailed example:
```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "@type", visible = true)
@JsonSubTypes({
        @JsonSubTypes.Type(value = Basketball.class, name = "basketball"),
        @JsonSubTypes.Type(value = Swim.class, name = "swim")
})public class Sports {
    private String name;
}
public class Basketball extends Sports {}
public class Swim extends Sports {}

@Test
public void testJsonTypeInfo() throws JsonProcessingException {
    String str1 = objectMapper.writeValueAsString(new Swim("breaststroke"));
    String str2 = objectMapper.writeValueAsString(new Basketball("Micheal"));
    System.out.println(str1 + str2);
}
```
The result is 
`{"@type":"swim","name":"Swim","swimType":"breaststroke"}`
`{"@type":"basketball","name":"Basketball","pg":"Micheal"}`. 
Now we have type identifier `@type` for  deserialization, but at first `@JsonIgnoreProperties(ignoreUnknown = true)
` need to be added.
```java
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "@type", visible = true)
@JsonSubTypes({
        @JsonSubTypes.Type(value = Basketball.class, name = "basketball"),
        @JsonSubTypes.Type(value = Swim.class, name = "swim")
})
public class Sports {...}

@Test
public void testJsonTypeInfoDeserialization() throws JsonProcessingException {
    String str = "{\"@type\":\"swim\",\"name\":\"Swim\",\"swimType\":\"breaststroke\"}";
    Sports sports = objectMapper.readValue(str, Sports.class);
    System.out.println("Sports is" + sports.getClass().getName());
    // The result is "Sports iscom.github.shaunlu.tutorial.jackson.Swim"
}
```
`JsonTypeInfo` is very useful when we need to deserialize polymorphic array
```java
@Test
public void testJsonTypeInfoArrayDeserialization() throws JsonProcessingException {
    String str = "[{\"@type\":\"swim\",\"name\":\"Swim\",\"swimType\":\"breaststroke\"}, " +
               "{\"@type\":\"basketball\",\"name\":\"Basketball\",\"pg\":\"Micheal\"}]";
    List<Sports> sportsList = objectMapper.readValue(str, objectMapper.getTypeFactory().constructCollectionType(List.class, Sports.class));
    System.out.println("Sports[0] is " + sportsList.get(0).getClass().getName() + "\n"
                     + "Sports[1] is " + sportsList.get(1).getClass().getName());
}
```
The result is
```
Sports[0] is com.github.shaunlu.tutorial.jackson.Swim
Sports[1] is com.github.shaunlu.tutorial.jackson.Basketball
```