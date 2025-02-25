# A different approach Spring properties

The Spring framework is probably the most popular framework used for Java applications. When defining a Spring application, we usually create configuration properties that allow us to define different sets of properties for each environment.

My usual configuration definition looks like this:

#### MyConfiguration.java

```java
@ConfigruationProperties("foobar")
public class MyConfiguration {

    private final String foo;
    private final String bar;

    public MyConfiguration(String foo, String bar) {
        this.foo = foo;
        this.bar = bar;
    }

    // Getters and Setters
}
```

#### application.yml

```yaml
foobar:
  foo: Foo
  bar: Bar
```

An issue with this however, is that sometimes, we don't always get the values thae way we want them. For instance, your organisation might have a setup where you are not able to determine the format of your environment variables. For some reason, the values given to you are represented in JSON but you only need one of the JSON attributes.

Before today I would have probably taken the entire JSON as a String, and then parse that JSON in every class that requires it. That does sound a little inefficient, doesn't it? What if there was a way we could this _within_ the configuration class?

#### MyConfiguration.java

```java
@ConfigruationProperties("foobar")
public class MyConfiguration {

    private final String foo;
    private final String bar;

    public MyConfiguration(String json) throws JsonProcessingException {
        FooBar fooBar = new ObjectMapper().readValue(json, FooBar.class);

        this.foo = fooBar.foo();
        this.bar = fooBar.bar();
    }

    // foo and bar Getters
    // ...

    private record FooBar(String foo, String bar) {}
}
```

#### application.yml

```yaml
foobar:
  json: |
    {
      "foo": "Foo",
      "bar": "Bar"
    }
```

So what did we learn here? Whilst the example above shows how we could parse a JSON string, the key takeway is that you can define your configuration properties simply **based on your class constructor arguments**. There's no need to specifically create getters/setters. This is wonderful, because our configuration class properties should effectively be **final**. Having setters allows modification, and we don't want that. This also unlocks so many different possibilities for initialising your properties class, which is extremely useful.
