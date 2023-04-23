# maven-plagin-example

Пример создания своего Maven-плагина

## Содержание:

1. [Установка Java/Maven](#установка-java--maven);
2. [Создаем Maven-Plagin Project](#создаем-maven-плагин-проект);
3. [Пишем свой логгер](#пишем-свой-логгер);
4. [Используем свой плагин в Maven-проекте](#используем-свой-плагин-в-maven-проекте);
5. [Расширяем плагин Example](#расширяем-плагин-example);
6. [Плагины с параметрами](#плагины-с-параметрами);

## [Установка Java / Maven](#содержание)

В данном примере используется Java 11 - amazon version, и Maven:latest

Ссылки для быстрой загрузки используемых ресурсов

| Софт    |                                     Ссылка                                     | Версия |
| ------- | :----------------------------------------------------------------------------: | :----: |
| Java 11 | https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/downloads-list.html | amazon |
| Maven   |                     https://maven.apache.org/download.cgi                      | latest |

Далее добавляем установленные ссылки на распакованные архивы в System или User PATH переменную

Для проверки работоспособности, используем команды:

```ps
java --version
mvn --version
```

> В случае успеха, увидите сообщения об установленных версиях

## [Создаем Maven-плагин проект](#содержание)

Создаем проект с архитектурой плагина:

```ps
mvn archetype:generate -DgroupId=ru.cmx -DartifactId=example-maven-plugin -Dversion=0.1.0-SNAPSHOT -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-plugin
```

## [Пишем свой логгер](#содержание)

Далее выпиливаем все ненужное для примера
В данном кейсе не рассматриваются инструменты тестирования, поэтому чтобы не вставлять себе палки в колеса, после правок - выпилим их из проекта:

- example-maven-plugin/src/it - удаляем
- example-maven-plugin/src/test - тоже удаляем

Приберёмся в pom.xml - убрав оттуда лишнее - не нужное для минимального билда плагина, оставляем только:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>ru.cmx</groupId>
  <artifactId>example-maven-plugin</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <packaging>maven-plugin</packaging>

  <name>example-maven-plugin Maven Plugin</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-plugin-api</artifactId>
      <version>3.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.maven.plugin-tools</groupId>
      <artifactId>maven-plugin-annotations</artifactId>
      <version>3.4</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
</project>
```

Далее переделываем плагин под самый простой вариант - обычный логгер инфы о том, что "очистка прошла успено", далее привяжем в clean phase в другом проекте.

Поэтому, переходим в MyMojo.java, и редактируем содержимое до следующего результата:

```java
package ru.cmx;

import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;

@Mojo(name = "log-clean")
public class LogCleanMojo extends AbstractMojo {
    public void execute()throws MojoExecutionException {
        System.out.println("CLEAN IS DONE! CONGRATULATIONS!");
    }
}

```

И ренеймим сам файл, если переименовали в нем класс MyMojo.java -> LogCleanMojo.java

Готово, осталось только разместить плагин в репозитории - локальном или удаленном. Для примера достаточно локального, поэтому пишем команду:

```ps
mvn clean install
```

И можем проверить, появившуюся папку в локальном репозитории ../m2/repository/ru/cmx/example-maven-plugin

## [Используем свой плагин в Maven-проекте](#содержание)

Для того, чтобы использовать новый плагин в своем проекте, необходимо его подключить в билд. В pom.xml добавляем следующее:

```xml
  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>ru.cmx</groupId>
          <artifactId>example-maven-plugin</artifactId>
          <version>0.1.0-SNAPSHOT</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
```

Готово, для того чтобы воспользоваться плагином, можно написать команду:

```ps
mvn example:log-clean
```

Если все успешно, в консоли увидим сообщение, которое указали в плагине в функции `execute()`

## [Расширяем плагин Example](#содержание)

Возвращаемся в проект с плагином. Добавим еще один goal. Делается это очень просто, создаем еще один Mojo класс в проекте - LogCompileMojo.java. Можно скопировать LogCleanMojo.java и переименовать класс и файл.

Содержимое нового файла:

```java
package ru.cmx;

import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;

@Mojo(name = "log-compile")
public class LogCompileMojo extends AbstractMojo {
    public void execute()throws MojoExecutionException {
        System.out.println("COMPILE IS DONE! CONGRATULATIONS!");
    }
}

```

Готово, снова собираем билд, и публикуем в локальный репозиторий:

```ps
mvn clean install
```

Далее переходим в Maven проект.

Попробуем привязать плагины к фазам жизненного цикла Maven проекта:

Для этого добавляем в `pom.xml` в тег `<build>` следующее:

```xml
    <plugins>
      <plugin>
        <groupId>ru.cmx</groupId>
        <artifactId>example-maven-plugin</artifactId>
        <executions>
          <execution>
            <phase>clean</phase>
            <goals>
              <goal>log-clean</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>

```

Тестируем, запуская стандартную фазу очистки Maven:

```ps
mvn clean
```

В случае успеха, после основных плагинов очистки - увидим наше сообщение об успехе.

Хорошо, попробуем добавить еще второй `goal` - `log-compile` на фазу компиляции

Для этого добавляем в тег `<executions>` нашего плагина новый <execution>, по аналогии:

```xml
          <execution>
            <phase>compile</phase>
            <goals>
              <goal>log-compile</goal>
            </goals>
          </execution>

```

Пробуем запустить:

```ps
mvn clean
```

Итого - успеха нет. Мало того, что мы новый goal не проверили, так еще и старый перестал работать с следующей ошибкой:

```ps
[INFO] Scanning for projects...
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] 'build.plugins.plugin.[ru.cmx:example-maven-plugin].executions.execution.id' must be unique but found duplicate execution with id default @ line 61, column 22
```

Почему? Потому что Maven'у нужно определить каждый execution - как уникальный, чтобы не было дубликатов. Артефакты, формуируемые у них, строятся из параметра `<id>`, который мы не переопределили. Поэтому у них использован дефолтный параметр, и у обоих он совпал. Из декларативного подхода, нам не нужно путать сборщик двусмысленностями действий, поэтому добавляем к каждому `execution` по `id`.

Итоговый `<build>` выглядит так:

```xml
  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>ru.cmx</groupId>
          <artifactId>example-maven-plugin</artifactId>
          <version>0.1.0-SNAPSHOT</version>
        </plugin>
      </plugins>
    </pluginManagement>
    <plugins>
      <plugin>
        <groupId>ru.cmx</groupId>
        <artifactId>example-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>logging-clean-after-phase</id>
            <phase>clean</phase>
            <goals>
              <goal>log-clean</goal>
            </goals>
          </execution>
          <execution>
            <id>logging-compile-after-phase</id>
            <phase>compile</phase>
            <goals>
              <goal>log-compile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

```

Можно `id` обозначить как 1 и 2, но лучше туда написать что-то осмысленное, чтобы потом не забыть а что происходит на фазе, с этим плагином.

Тестируем что получилось:

```ps
mvn clean compile
```

В случае успеха, мы увидим оба сообщения и все запустится без ошибок.

## [Плагины с параметрами](#содержание)

И так, статичные плагины это хорошо, но как насчет динамически подстраивающихся под ситуацию?

Выпилим дубликат получающихся целей log-compile и log-clean, объеденив их в один плагин с параметром, который назовем log

Создаем новый Mojo класс из любого уже существующего в нашем плагине, например LogMojo.java с содержимым:

```java
package ru.cmx;

import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;
import org.apache.maven.plugins.annotations.Parameter;

@Mojo(name = "log")
public class LogMojo extends AbstractMojo {

    @Parameter( property = "example.phase", defaultValue = "NO PHASE" )
    private String phase;

    public void execute()throws MojoExecutionException {
        System.out.println(phase + " IS DONE! CONGRATULATIONS!");
    }
}

```

Готово. Билдим - деплоим в локальный репозиторий:

```ps
mvn clean install
```

Далее, переходим в Maven проект.

Правим `<executions>`, добавляя нашу новую переменную:

```xml
        <executions>
          <execution>
            <id>logging-clean-after-phase</id>
            <phase>clean</phase>
            <goals>
              <goal>log</goal>
            </goals>
            <configuration>
              <phase>CLEAN</phase>
            </configuration>
          </execution>
          <execution>
            <id>logging-compile-after-phase</id>
            <phase>compile</phase>
            <goals>
              <goal>log</goal>
            </goals>
            <configuration>
              <phase>COMPILE</phase>
            </configuration>
          </execution>
        </executions>

```

И финальный штрих - тестируем:

```ps
mvn clean compile
```

В случае успеха, результат будет такой же, как и до нашего "рефакторинга".