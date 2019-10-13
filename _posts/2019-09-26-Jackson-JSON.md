---
layout: post
title: "Jackson JSON Tutorial"
subtitle: 'Jackson usage examples'
author: "Shaun"
header-style: text
tags: ["Java Utils"]
---

Jackson is a very popular java library to serialize/deserialize JSON. This tutorial will demonstrate common solutions when marshalling/unmarshalling.

---
## 1 Property Annotation
### 1\.1 @JsonIgnoreProperties, @JsonIgnore, @JsonIgnoreType
Specify the properties the jackson will ignore in serialization/deserialization.
```java
@JsonIgnoreProperties({"id", "name"})
public class Employee {
    private String id;
    private String name;
    private int age;
}

@JsonIgnore("id")
public class Employee {
    private String id;
    private String name;
    private int age;
}

public class Employee {
    private String id;
    private String name;
    private Collection collection;
    @JsonIgnoreType
    public static class Collection {
        ...
    }   
}
```
#### 1\.2 @JsonProperty
Indicate the property name in JSON.
```java
public class Product {
    @JsonProperty("uuid")
    private String id;
    @JsonProperty("p_name")
    private String name;
}
```
#### 1\.3 @JsonFormat
Specify a format when serializing Date/Time properties;
```java
public class Product {
    private String id;
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    private Date createTime;
}
```
#### 1\.4 @JsonInclude
Exclude properties with empty/null/default values.
```java
@JsonInclude(Include.NON_NULL)
public class Product {
    private String name;
    private String type;
}
```
#### 1\.5 @JsonView
This annotation is used to define which property will be included in serialization/deserialization in a view.
For example, we have a user object like below:
```java
public class User {
    private String id;
    private String name;
    private String email;
    private String birthday;
}
```
We want to serialize `id`,`name`,`email` in most API response but only show `birthday` property in a specified API response.
In this situation, defining a view is a good practise.
Firstly, we define two classes which indicates different visibility.
```java
public interface UserJsonCommonView {}
public interface UserJsonFullView extends UserJsonCommonView {}

public class User {
    @JsonView(UserJsonCommonView.class)
    private String id;
    @JsonView(UserJsonCommonView.class)
    private String name;
    @JsonView(UserJsonCommonView.class)
    private String email;
    @JsonView(UserJsonFullView.class)
    private String birthday;
}
```
So this time, `birthday` property is only for view `UserJsonFullView`.
  
---
## 2 Marshalling Annotation
### 2\.1 @JsonGetter
Mark a method as a getter method.
```java
public class Employee { 
    private String name;
    @JsonGetter("name")
    public String getTheName() { 
        return name;
    }
}
```

#### 2\.2 @JsonAnyGetter
Serialize key/values paris in a map as standard properties.
```java
public class Employee { 
    private String name;
    private Map<String, String> props;
    @JsonAnyGetter 
    public Map<String, String> getProps() {return props;}
}
```
For example:
```java
Employee employee = new Employee();
employee.setName("shaun");
employee.getProps.put("birthday", "1988");
new ObjectMapper().writeValueAsString(employee);
```

Then the result is `{"name" : "shaun", "birthday" : "1988"}`

#### 2\.3 @JsonPropertyOrder
To specify the order of properties on serialization.
```java
@JsonPropertyOrder({"name", "gender", "birthday"})
public class Employee { 
    private String birthday;
    private String gender;
    private String name;
}
```

#### 2\.4 @JsonRawValue
Serialize a property exactly as it is.
```java
public class Product {
    private String id;
    @JsonRawValue
    private String jsonObject;
}
```
For example:
```java
product.setJsonObject("{\"name\":\"apple\", \"type\":\"fruit\"}");
```
Then the result looks like
```javascript
{
    "id": "1",
    "jsonObject" : {
        "name": "apple",
        "type": "fruit"
    }
}
```
#### 2\.5 @JsonValue
Indicates a single method that the library will use to serialize the object.
```java
public enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type 2");
    private Integer id;
    private String name;
    @JsonValue
    public String getName() {
        return name;
    }
}
```
#### 2\.6 @JsonSerialize
This annotation indicates a custom serializer used for marshalling.
```java
public class ProductDetail{
...
}
public class Product {
    private String id;
    @JsonSerialize(using = ProductDetailSerializer.class)
    private ProductDetail detail;
}
public class ProductDetailSerializer extends StdSerializer<ProductDetail> { 
    public ProductDetailSerializer() { 
        this(null); 
    } 
    public ProductDetailSerializer(Class<ProductDetail> t) {
        super(t); 
    }
    @Override
    public void serialize(ProductDetail value, JsonGenerator gen, SerializerProvider arg2) throws IOException, JsonProcessingException { 
        String sValue="...";
        gen.writeString(sValue);
    }
}
```

---
## 3 Unmarshalling
#### 3\.1 @JsonCreator
Indicates the constructor/factory used in deserialization.
```java
public class Product {
    private String id;
    private String name;
    @JsonCreator
    public Product(@JsonProperty("uuid") String id,
                   @JsonProperty("pName") String name) {
        this.id = id;
        this.name = name;
    }
}
```
#### 3\.2 @JsonAnySetter
Just like `@JsonAnyGetter`, this annotation is used to deserialize properties into a map.
```java
public class Employee { 
    private String name;
    private Map<String, String> props;
    @JsonAnyGetter 
    public Map<String, String> getProps() {return props;}
    @JsonAnySetter 
    public void add(String key, String values) {this.props.put(key, value);}
}
```
#### 3\.3 @JsonSetter
Mark a method as setter method.

#### 3\.4 @JsonDeserialize
This annotation indicates a custom deserializer.
```java
public class ProductDetail{
...
}
public class Product {
    private String id;
    @JsonSerialize(using = ProductDetailSerializer.class)
    @JsonDeserialize(using = ProductDetailDeserializer.class)
    private ProductDetail detail;
}
public class ProductDetailDeserializer extends StdDeserializer<ProductDetail> { 
    public ProductDetailDeserializer() { 
        this(null); 
    } 
    public ProductDetailDeserializer(Class<ProductDetail> t) {
        super(t); 
    }
    @Override
    public ProductDetail deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException {
        String jsonStr = jsonparser.getText();
        ProductDetail pd = ...;
        return pd; 
    }
}
```
#### 3\.5 @JsonAlias
It defines one or more alternative names for a property during deserialization.
```java
public class Product {
    @JsonAlias({"pName", "product_name", "productName"})
    private String name;
 
    private String type;
}
```

---
## 4 Polymorphic Type Handling
- @JsonTypeInfo - indicates details of what type information to include in serialization
- @JsonSubTypes - indicates sub-types of the annotated type
- @JsonTypeName - defines a logical type name to use for annotated class

We can use @JsonTypeInfo to add a new property about class type in serialization. And in deserialization, 
this class type can be used for polymorphic handling.
```java
@JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME, 
      include = As.PROPERTY, 
      property = "fish_type")
public class Fish {
    ...
}

public class GoldenFish extends Fish {...}

public class Shark extends Fish {...}

public class Test {
    public static void main() {
        ObjectWriter objWriter = new ObjectWriter();
        List<Fish> fishes = new ArrayList<>();
        fishes.add(new GoldenFish());
        fishes.add(new Shark());
        String result = objWriter.writeValuesAsString(fishes);
        List<Fish> retList = objWriter.readValue(result, objectMapper.getTypeFactory().constructCollectionType(List.class, Fish.class));
    }
}
```
