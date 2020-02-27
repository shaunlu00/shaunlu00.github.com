---
layout: post
title: "Jackson JSON Tutorial(1)"
subtitle: 'Jackson Annotations'
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

Let's first add the following dependencies to the pom.xml
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>[2.9.9,)</version>
</dependency>
```

Basic usage example:  
```java
ObjectMapper objectMapper = new ObjectMapper();
// write object to json string
Product product = new Product(...);
objectMapper.writeValueAsString(product);
// read object from json string
String productStr = "...";
Product p = objectMapper.readValue(productStr, Product.class);
```

---
## 1 Property Annotation
### 1\.1 @JsonIgnoreProperties, @JsonIgnore, @JsonIgnoreType
Specify the properties the jackson will ignore in serialization/deserialization.
```java
@JsonIgnoreProperties({"id", "socialIdentityNumber"})
public class Employee {
    private Long id;
    private String name;
    private int age;
    private String socialIdentityNumber;
    private Job job;
    // constructor getter setter

    @JsonIgnoreType
    public static class Job {
        private String jobDescription;
        // .. getter setter
    }   
}

@Test
public void testJsonIgnore() throws JsonProcessingException {
    Employee employee = new Employee(1000L, "peter", 35, "1x0192ks1921", new Employee.Job("job desc"));
    System.out.println(new ObjectMapper().writeValueAsString(employee));
}
```
The result is `{"name":"peter","age":35}`

#### 1\.2 @JsonProperty
Indicate the property name in JSON.
```java
public class Product {
    @JsonProperty("UUID")
    private String id;
    @JsonProperty("PRODUCT_NAME")
    private String name;
    // .. getter setter
}

@Test
public void testJsonProperty() throws JsonProcessingException { 
    Product product = new Product("k1001", "MacBook");
    System.out.println(objectMapper.writeValueAsString(product));
}
```
The result is `{"UUID":"k1001","PRODUCT_NAME":"MacBook"}`

#### 1\.3 @JsonFormat
Specify how to format fields/properties for JSON output
```java
public class Product {
    @JsonProperty("UUID")
    private String id;
    @JsonProperty("PRODUCT_NAME")
    private String name;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy@hh:mm:ss")
    private Date createTime = new Date();

    @JsonFormat(shape = JsonFormat.Shape.NUMBER)
    private Date updateTime = new Date();

    private ProductType productType = ProductType.COMPUTER;
    @JsonFormat(shape = JsonFormat.Shape.OBJECT)
    public enum ProductType {
        COMPUTER("computer", 9011),
        BOOK("book", 9012);
        String code;
        int number;
        ProductType(String code, int number) {
            this.code = code;
            this.number = number;
        }
        // .. getter setter
    }
    // .. getter setter
}

@Test
public void testJsonFormat() throws JsonProcessingException {
    Product product = new Product("k1001", "MacBook");
    System.out.println(objectMapper.writeValueAsString(product));
}
```
The result is `{"createTime":"20-10-2019@03:55:10","updateTime":1571543710307,"productType":{"code":"computer","number":9011},"UUID":"k1001","PRODUCT_NAME":"MacBook"}`
    
#### 1\.4 @JsonInclude
Indicates which kind of properties are to be serialized.  
**ALWAYS**: Properties are always included  
**NON_NULL**: Null properties are excluded  
**NON_ABSENT**: Null or "Absent"(Java 8 Optional) properties are excluded  
**NON_EMPTY**: Null, "Absent", Empty Collection/Map, Empty String are excluded  

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Product {
    private String name = null;
    private String type = "";
    // getter setter
}
@Test
public void testJsonInclude() throws JsonProcessingException { 
    Product product = new Product();
    System.out.println(objectMapper.writeValueAsString(product));
}
```
The result is `{"type":""}`

#### 1\.5 @JsonView
The most common use case is to control the visibility of properties in serialization/deserialization.
For example, we have a user object like below:
```java
public class User {
    private String id;
    private String birthday;
    private String name;
    private String email;
}
```
We only want to return 'name', 'email' in a public API but expose 'birthday' and 'id' for internal service.  
Firstly, we define two classes which indicates different views.
```java
public class UserJsonView {
    public static class Public {}
    public static class Internal extends Public {}
}

public class User {
    @JsonView(UserJsonView.Internal.class)
    private String id;
    @JsonView(UserJsonView.Internal.class)
    private String birthday;
    @JsonView(UserJsonView.Public.class)
    private String name;
    @JsonView(UserJsonView.Public.class)
    private String email;
}
```
If we want to use Public view to serialize
```java
User user = new User("111", "1988-10-10", "shaun", "shaun@test.com");
objectMapper.writerWithView(UserJsonView.Public.class)
            .writeValueAsString(user);
```
The result is `{"name":"shaun","email":"shaun@test.com"}`. And if we can use Internal view to show all properties
```java
User user = new User("111", "1988-10-10", "shaun", "shaun@test.com");
objectMapper.writerWithView(UserJsonView.Internal.class)
            .writeValueAsString(user);
```
The result is: `{"id":"111","birthday":"1988-10-10","name":"shaun","email":"shaun@test.com"}`

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
public class Company {
    private String name;
    private String location;
    private String manager;
    private Map<String, String> companyProperties = new HashMap<>();
    @JsonAnyGetter
    public Map<String, String> getCompanyProperties() {
        return companyProperties;
    }
}

@Test
public void testJsonGetter() throws JsonProcessingException { 
    Company company = new Company("Siemens", "Germany", "John");
    company.getCompanyProperties().put("headquarter", "Munich");
    company.getCompanyProperties().put("Number of employee", "100000");
    System.out.println(objectMapper.writeValueAsString(company));
}
```
The result is `{"name":"Siemens","location":"Germany","manager":"John","headquarter":"Munich","Number of employee":"100000"}`

#### 2\.3 @JsonPropertyOrder
To specify the order of properties on serialization.
```java
@JsonPropertyOrder({"manager", "name", "location"})
public class Company {
    private String name;
    private String location;
    private String manager;
}
@Test
public void testJsonOrder() throws JsonProcessingException {
    Company company = new Company("Siemens", "Germany", "John");
    System.out.println(objectMapper.writeValueAsString(company));
}
```
The result is `{"manager":"John","name":"Siemens","location":"Germany"}`

#### 2\.4 @JsonRawValue
Serialize a property exactly as it is.
```java
public class Company {
    // ...
    private String id;
    @JsonRawValue
    private String companyEmployee;
}
@Test
public void testJsonRawValue() throws JsonProcessingException {
    Company company = new Company("Siemens", "Germany", "John");
    company.setCompanyEmployee("[{\"name\":\"peter\"}, {\"name\":\"john\"}]");
    System.out.println(objectMapper.writeValueAsString(company));
}
```
The result is `{"manager":"John","name":"Siemens","location":"Germany","companyEmployee":[{"name":"peter"}, {"name":"john"}]}
`

#### 2\.5 @JsonValue
Indicates a single method that the library will use to serialize the object.
```java
public class Student {
    private String name;
    private Major major = new Major("a001", "Mathematics");
    public static class Major {
        private String id;
        private String name;
        @JsonValue
        public String getValue(){
            return id + ":" + name;
        }
    }
}
@Test
public void testJsonValue() throws JsonProcessingException {
    Student student = new Student("Shaun");
    System.out.println(objectMapper.writeValueAsString(student));
}
```
The result is `{"name":"Shaun","major":"a001:Mathematics"}`

#### 2\.6 @JsonSerialize
This annotation indicates a custom serializer used for marshalling.
```java
public class MonkeyKing {
    private String name = "Sun Wukong";
    private Weapon weapon = new Weapon("1000m", "1000kg");
    @JsonSerialize(using = WeaponSerializer.class)
    public static class Weapon {
        private String length;
        private String weight;
    }

    public static class WeaponSerializer extends StdSerializer<Weapon> {

        public WeaponSerializer(Class<Weapon> t) {
            super(t);
        }

        public WeaponSerializer() {
            this(null);
        }

        @Override
        public void serialize(Weapon weapon, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
            String value = "Power Stick:" + weapon.getLength() + "-" + weapon.getWeight();
            jsonGenerator.writeString(value);
        }
    }
}

@Test
public void testJsonSerializer() throws JsonProcessingException {
    MonkeyKing monkeyKing = new MonkeyKing();
    System.out.println(objectMapper.writeValueAsString(monkeyKing));
}
```
The result is `{"name":"Sun Wukong","weapon":"Stick:1000m---{}1000kg"}`

---
## 3 Unmarshalling
#### 3\.1 @JsonCreator
Indicates the constructor/factory used in deserialization.
```java
public class Elephant {
    private String sex;
    private String livingPlace;

    public Elephant(){ }
    @JsonCreator
    public Elephant(@JsonProperty("SEEX") String sex,
                   @JsonProperty("LIVPLACE") String livingPlace) {
        this.sex = sex;
        this.livingPlace = livingPlace;
    }
}
@Test
public void testJsonCreator() throws JsonProcessingException {
    String str = "{\"SEEX\" : \"Male\", \"LIVPLACE\" : \"Africa\"}";
    Elephant elephant = objectMapper.readValue(str, Elephant.class);
}
```
#### 3\.2 @JsonAnySetter
Just like `@JsonAnyGetter`, this annotation is used to deserialize unmapped properties into a map.
```java
public class Tiger {
    private String name = "tiger";
    private Map<String, Object> properties = new HashMap();
    @JsonAnySetter
    public void add(String key, Object value) {
        properties.put(key, value);
    }
}

@Test
public void testJsonAnySetter() throws JsonProcessingException {
    String str = "{\"name\" : \"riji\", \"Other\":{\"Weight\":\"1T\"}}";
    Tiger tiger = objectMapper.readValue(str, Tiger.class);
    System.out.println(tiger.getProperties().get("Other").getClass());
    Map other = (Map) tiger.getProperties().get("Other");
    System.out.println(other.get("Weight"));
}   
```
The unmapped property `Other` is deserialized to `Map<String, Object>`. As the `Other` is an object, so the value is mapped as LinkedHashMap.
So the output is:  
*'Other' property is mapped as:class java.util.LinkedHashMap*  
*Other.Weight is '1T'*

#### 3\.3 @JsonSetter
Mark a method as setter method.

#### 3\.4 @JsonDeserialize
This annotation indicates a custom deserializer.
```java
public class Song {
    private String singer;
    @JsonDeserialize(using = WriterDeserializer.class)
    private Writer writer;
    
    public static class Writer {
        private String name;
        private String company;
    }
    
    public static class WriterDeserializer extends StdDeserializer<Writer> {
            public WriterDeserializer() {
                this(null);
            }
            public WriterDeserializer(Class<Writer> t) {
                super(t);
            }
            @Override
            public Writer deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException {
                String jsonStr = jsonparser.getText();
                String name = jsonStr.split(":")[0];
                String company = jsonStr.split(":")[1];
                return new Writer(name, company);
            }
    }
}
```
The result is *This song is written by Steve*

#### 3\.5 @JsonAlias
It defines one or more alternative names for a property during deserialization.
```java
public class Product {
    @JsonAlias({"pName", "product_name", "productName"})
    private String name;
 
    private String type;
}
```


