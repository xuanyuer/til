# maven.config is Annoying

Since version `3.3.1` maven instroduce the `maven.config` file. This allows users to specify a set of arguments/options to be applied whenever then run maven on a particular project.

Say for example we have the following project:

```
.
├── .mvn
│   ├── maven.config
│   └── wrapper
|       ├── maven-wrapper.properties
├── pom.xml
└── src
    └── main
        └── java
            └── Application.java
```

When you run maven commands like `mvn clean install` on the root of this project, maven would look at the `.mvn/maven.config` file and apply all the specified options to your maven command. For instance, if you have a custom `settings.xml` you can place it in this file and maven will apply it for you.

## Why is it annoying?

Let's save `maven.config` has the following content:

```
-s /some/path/settings.xml -Dmaven.repo.local=/some/path/.m2/repository --batch-mode --Dmy.custom.param=hello
```

Based on the above you'd probably think, that when you run `mvn clean install` it would basically just do:

```
mvn -s /some/path/settings.xml -Dmaven.repo.local=/some/path/.m2/repository --batch-mode --Dmy.custom.param=hello clean install
```

Right?

No.

What you effectively get is maven trying to apply settings like:

```
mvn -s "/project/root/path/ /some/path/settings.xml -Dmaven.repo.local=/some/path/.m2/repository --batch-mode --Dmy.custom.param=hello" clean install
```

## Why does this happen?

According to [MNG-7684](https://issues.apache.org/jira/browse/MNG-7684), there was a breaking change that treated every line in the `maven.config` file as a separate argument. This only happens if you're using maven version **3.9** and above. Prior to this version, this wouldn't happen.

As for why `/project/root/path` gets included, I have no idea. It's probably due to how maven is parsing the line, and seeing that `-s` points to blank it automatically assumes the context of execution as the project root.

## How should we write `maven.config`?

The above example should be written like this for it to work

```
-s
/some/path/settings.xml
-Dmaven.repo.local=/some/path/.m2/repository
--batch-mode
--Dmy.custom.param=hello
```

You can see that `-s` and `/some/path/settings.xml` are on separate lines. That's because, as mentioned, each **line** is an **argument**. When you do `mvn -s /some/path/settings.xml clean install`, `-s` and `/some/path/settings.xml` and 2 separate arguments even though they're meant to work together.

You can work around this for clarity though. `-s` is short of `--settings`, and you can use the `=` sign to specify key value pairs in one argument. Therefore the `maven.config` could be written like so:

```
--settings=/some/path/settings.xml
-Dmaven.repo.local=/some/path/.m2/repository
--batch-mode
--Dmy.custom.param=hello
```

Anyhow, annoying breaking change aside, it's even more annoying that I was not able to find any official documentation (or any documentation) pertaining to `maven.config`. After hours of searching I finally found my answer in a [Microsoft Azure GitHub issue](https://github.com/Azure/reliable-web-app-pattern-java/issues/35#issuecomment-1660163133).

I hope this helps somebody some day, even if that somebody is future me.
