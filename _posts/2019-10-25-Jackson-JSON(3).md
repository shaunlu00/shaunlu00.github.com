---
layout: post
title: "Jackson JSON Tutorial(3)"
subtitle: 'Enum, Date, JsonNode'
author: "Shaun"
header-style: text
tags: ["Java Utils"]
---

Jackson is a very popular java library to serialize/deserialize JSON. 
This tutorial illustrates common solutions while marshalling and unmarshalling JSON.

[-> (1) Jackson Annotations]({{ site.baseurl }}/2019/09/26/Jackson-JSON(1)/)  
[-> (2) Collection/Array, Polymorphic Type]({{ site.baseurl }}/2019/10/12/Jackson-JSON(2)/)  
[-> (3) Enum, Date, JsonNode]({{ site.baseurl }}/2019/10/25/Jackson-JSON(3)/)   

You can find all examples at [https://github.com/shaunlu00/java-tutorial-jackson](https://github.com/shaunlu00/java-tutorial-jackson)

---
## 1 Enumeration Handling
By default, Jackson will represent Java Enums as String, for example:
```java
public enum Status {
    TODO("todo", 1),
    IN_PROGRESS("in progress", 2),
    COMPLETED("completed", 3);
    private String key;
    private int value;
    Status(String key, int value) { 
        this.key = key;
        this.value = value;
    }
}
@Test
public void testJsonEnum() throws JsonProcessingException {
    Status status = Status.IN_PROGRESS;
    System.out.println(objectMapper.writeValueAsString(status));
}
```
The result is `IN_PROGRESS`

Jackson can serialize enum as json object with @JsonFormat annotation:
```java
@JsonFormat(shape=JsonFormat.Shape.OBJECT)
public enum Status {}
```
Then the output is `{"key":"in progress","value":2}`

Wen can also fully control enum marshalling/unmarshalling by @JsonCreator and @JsonValue
```java
public enum  State {

    ACTIVE("active", 0),
    DEAD("dead", 1),
    SUSPEND("suspend", 2),
    UNKNOW("unknow", 3);

    private String key;
    private int value;

    State(String key, int value) {
        this.key = key;
        this.value = value;
    }

    @JsonCreator
    public static State createState(int value) {
        for(State state : values()){
            if (state.getValue() == value) {
                return state;
            }
        }
        return State.UNKNOW;
    }

    @JsonValue
    public String getStateValue(){
        return key;
    }
}
@Test
public void testJsonEnum2() throws JsonProcessingException {
    String str = "2";
    State state = objectMapper.readValue(str, State.class);
    System.out.println(objectMapper.writeValueAsString(state));
}
The result is 'suspend'
```

---
## 2 Date
We can set date format on `ObjectMapper`
```java
@Test
public void testJsonDate() throws JsonProcessingException {
    Date date = new Date();
    objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm"));
    System.out.println(objectMapper.writeValueAsString(date));
}
```
Or we use `@JsonFormat` to format date
```java
class Temp {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "hh:mm:ss dd-MM-yyyy")
    Date date = new Date();
}
```
To serialize Java 8 DateTime, we need to import `jackson-datatype-jsr310` module
```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>[2.9.9,)</version>
</dependency>
```
Then register JavaTime module
```java
@Test
public void testJsonJava8Date() throws JsonProcessingException{
    LocalDateTime dateTime = LocalDateTime.of(2019, 05, 01, 13, 31, 0);
    objectMapper.registerModule(new JavaTimeModule());
    objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    System.out.println(objectMapper.writeValueAsString(dateTime));
}
```

---
## 3 JsonNode
JsonNode is a tree model used for low level parsing.
```java
@Test
public void testJsonNode() throws JsonProcessingException{
    String str = "{\"name\":\"peter\", \"age\":\"20\"}";
    JsonNode jsonNode = objectMapper.readTree(str);
    System.out.println(jsonNode.get("name").asText());
    System.out.println(jsonNode.get("age").asInt());
}
```