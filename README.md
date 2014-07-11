[![Build Status](https://travis-ci.org/TNG/junit-dataprovider.png?branch=master)](https://travis-ci.org/TNG/junit-dataprovider)
[![Coverage Status](https://coveralls.io/repos/TNG/junit-dataprovider/badge.png?branch=master)](https://coveralls.io/r/TNG/junit-dataprovider)


junit-dataprovider
==================

#### Table of Contents  
* [What is it](#what-is-it)
* [Motivation and distinction](#motivation-and-distinction)
* [Requirements](#requirements)
* [Download](#download)
* [Usage examples](#usage-examples)
	* [Array syntax](#array-syntax)
	* [String syntax](#string-syntax)
	* [List syntax](#list-syntax)
	* [Let ```@DataProvider``` directly providing test data](#let-dataprovider-directly-providing-test-data)
	* [Access ```FrameworkMethod``` within ```@DataProvider``` method](#access-frameworkmethod-within-dataprovider-method)
* [Release notes](#release-notes)
* [Eclipse template](#eclipse-template)
* [Contributing](#contributing)


What is it
----------

A [TestNG](http://testng.org/doc/index.html) like 
dataprovider (see [here](http://testng.org/doc/documentation-main.html#parameters-dataproviders)) 
runner for [JUnit][] having a simplified syntax 
compared to all the existing [JUnit features](https://github.com/junit-team/junit/wiki).

[JUnit]: https://github.com/junit-team/junit

Motivation and distinction 
--------------------------

#### What is the advantage compared to [JUnit Theories][]?

> Test cases for [JUnit Theories][] are built from all data points whose type matches 
> the method's argument – or even the cross product of all matching data points, 
> if the method takes several arguments. The junit-dataprovider, however, adresses 
> another use case: Its test cases may consist of multiple parameters that belong together, 
> which may contain test input values and/or expected values to assert the result.
> Furthermore, a test method using [JUnit Theories][] fails or succeeds entirely (for alle 
> data points), on the contrary the junit-dataprovider considers each row of the data provider
> as standalone test case.


#### Why can I not use [JUnit Theories][] and data points containing [DTO][]s for test cases?

> Of course, this is also a possible way to use [JUnit Theories][], constructing DTOs for 
> every single data point causes a lot of [boiler plate](http://en.wikipedia.org/wiki/Boilerplate_%28text%29)
> code and inconvenience. This is AFAIK also the case when you use the ParameterSupplier 
> feature of [JUnit Theories][], where you additionally need a custom Annotation and a class...

[DTO]: http://en.wikipedia.org/wiki/Data_transfer_object


#### But why does [JUnit][] not support data providers?

> They do, having another name for it, tough, just see [Parameterized][]. 
> The advantage of this concept is surely that it is completely 
> [typesafe](http://en.wikipedia.org/wiki/Type_safety). But unfortunatly one has to create
> a class per data provider or parameterized test, respectively, which is IMHO also overkill.
> The tests of a single unit (i.e. class) have to be divided into different classes, 
> which need to be maintained (renamed, moved etc.) separately.
> Furthermore, [Parameterized][] tests (even if there are more than a single test method within 
> one test class) can only be executed altoghter for a test class. A junit dataprovider test, though, 
> can be executed independent on test method level (even if the same data provider is reused for 
> more than one test method).

[Parameterized]: https://github.com/junit-team/junit/wiki/Parameterized-tests


#### Is it possible to execute a junit-dataprovider test method for a single test data row?

> Unfortunately this is not possible directly expect if the other test data rows are commented out 
> in the source code. The rerun of a single test data row, though, is working, if e.g. in Eclipse 
> you right click the test to be executed and choose run/debug.


[JUnit Theories]: https://github.com/junit-team/junit/wiki/Theories

#### May I move a junit-dataprovider into a separate class?

> Of course, just move it to any from the test case accessible class and annotate it as usual with 
> properly and use it with additionally specifying the location, see [Usage examples](#usage-examples).

#### Why must a ```@Dataprovider``` be static while similar [junitparams](https://code.google.com/p/junitparams/) does allow it non-static?

> To answer this question we have to look into the internal implementation of [JUnit][]. Its first step is to determine 
> all test methodes to run, before validating and filtering it. At this points of execution, though, no instance of the
> test class is instanciated.
> Then as a second step [JUnit][] creates a single instance of the class under test for every single test method.
> One possiblity, for sure, is that junit-dataprovider could create an instance and invoke the ```@DataProvider``` 
> as [junitparams][] does interally, but neither ```@Before``` nor ```MethodeRule``` is minded. 
> Therefore, I decided to disallow a ```@DataProvider``` instance methods that nobody using junit-dataprovider 
> believes accessing data of a test class instance is possible.

*Note*: Even ```@BeforeClass``` and ```@ClassRule``` are not executed currently (see [#22](/../../issues/22)) 
before the static ```@DataProvider``` method because (validation and) filtering of test methods is done 
immediately after creation of ```DataProviderRunner``` :-(


Requirements
-----------

This JUnit dataprovider requires JUnit in version 4.8.2+ (see 
[junit-dep-4.8.2](http://search.maven.org/#artifactdetails|junit|junit-dep|4.8.2|jar)
/ [junit-4.8.2](http://search.maven.org/#artifactdetails|junit|junit|4.8.2|jar)). 

If you are using a previous version and cannot upgrade, please let us know by opening an issue.

Download
--------

All released (= tagged) versions are available at 
[Maven Central Repository](http://search.maven.org/#search|ga|1|a%3A%22junit-dataprovider%22). 
Following this link you can choose a version. For more information about a certain version, see
[release notes](#release-notes). Now either download it manually or see 
the **Dependency Information** section how to integrate it with your dependency management tool.


Usage examples
--------------

### Array syntax 

For example using [Java](https://www.java.com/) and its array syntax:


```java
import static org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;

import com.tngtech.java.junit.dataprovider.DataProvider;
import com.tngtech.java.junit.dataprovider.DataProviderRunner;
import com.tngtech.java.junit.dataprovider.UseDataProvider;

@RunWith(DataProviderRunner.class)
public class DataProviderTest {

    @DataProvider
    public static Object[][] dataProviderAdd() {
        // @formatter:off
        return new Object[][] {
                { 0, 0, 0 },
                { 1, 1, 2 },
                /* ... */
        };
        // @formatter:on
    }

    @Test
    @UseDataProvider("dataProviderAdd")
    public void testAdd(int a, int b, int expected) {
        // Given:

        // When:
        int result = a + b;

        // Then:
        assertEquals(expected, result);
    }
    
    @Test
    @UseDataProvider(value = "dataProviderIsStringLengthGreaterTwo", location = StringDataProvider.class)
    public void testIsStringLengthGreaterThanTwo(String str, boolean expected) {

        // Given:

        // When:
        boolean isGreaterThanTwo = (str == null) ? false : str.length() > 2;

        // Then:
        assertThat(isGreaterThanTwo).isEqualTo(expected);
    }
}
```

```java
import com.tngtech.java.junit.dataprovider.DataProvider;

public class StringDataProvider {

    @DataProvider
    public static Object[][] dataProviderIsStringLengthGreaterTwo() {
        // @formatter:off
        return new Object[][] {
                { "",       false },
                { "1",      false },
                { "12",     false },
                { "123",    true },
                { "Test",   true },
            };
        // @formatter:on
    }
}
```

### String syntax

For example using [Java](https://www.java.com/) and its ```String``` syntax:


```java
import static org.junit.Assert.*;

import java.io.File;

import org.junit.Test;
import org.junit.runner.RunWith;

import com.tngtech.java.junit.dataprovider.*;

@RunWith(DataProviderRunner.class)
public class DataProviderTest {

    @DataProvider
    public static String[] dataProviderFileExistence() {
        // @formatter:off
        return new String[] {
                "src,             true",
                "src/main,        true",
                "src/main/java/,  true",
                "src/test/java/,  true",
                "test,            false",
        };
        // @formatter:on
    }

    @Test
    @UseDataProvider("dataProviderFileExistence")
    public void testFileExistence(File file, boolean expected) {
        // Expect:
        assertThat(file.exists()).isEqualTo(expected);
    }
}
```

### List syntax

For example using [Groovy](http://groovy.codehaus.org/) (if you are not able 
to use [Spock](http://spock-framework.readthedocs.org)) and its native ```List``` support:

```groovy
import static org.assertj.core.api.Assertions.assertThat

import org.junit.Test
import org.junit.runner.RunWith

import com.tngtech.java.junit.dataprovider.*

@RunWith(DataProviderRunner)
class DataProviderTest {

    @DataProvider
    static List<List<Object>> dataProviderBooleanLogicAnd() {
        // @formatter:off
        return [
            [ false,  false,  false ],
            [ true,   false,  false ],
            [ false,  true,   false ],
            [ true,   true,   true ],
        ]
        // @formatter:on
    }

    @Test
    @UseDataProvider('dataProviderBooleanLogicAnd')
    void 'test boolean logic for "and"'(op1, op2, expected) {
        // Expect:
        assert (op1 && op2) == expected
    }
}
```

### Let ```@DataProvider``` directly providing test data

Instead of using ```@UseDataProvider``` to point to a method providing test data, you can directly
pass test data using ```@DataProvider``` annotation and its ```#value()``` method to provide an
array of regex-separated ```String```s. Each regex-separated ```String``` is split and trimmed back by
spaces (= "``` ```"), tabs (= "```\t```) and line-separator (= "```\n```" or "```\r```"). The
resulting ```String``` is then parsed to its corresponding type in the test method signature. All primitive
types (e.g. ```char```, ```boolean```, ```int```), primitive wrapper types (e.g. ```Long```, ```Double```), ```Enum```s,
and ```String```s are supported.

*Note:* The ```String``` "null" will always be passed as ```null```.

```java
    // @formatter:off
    @Test
    @DataProvider({
            ",                 0",
            "a,                1",
            "abc,              3",
            "veryLongString,  14",
        })
    // @formatter:off
    public void testStringLength(String str, int expectedLength) {
        // Expect:
        assertThat(str.length()).isEqualTo(expectedLength);
    }

    @Test
    @DataProvider({
            "null",
            "",
        })
    public void testIsEmptyString2(String str) {
        // When:
        boolean isEmpty = (str == null) ? true : str.isEmpty();

        // Then:
        assertThat(isEmpty).isTrue();
    }
```

### Access ```FrameworkMethod``` within ```@DataProvider``` method

```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExternalFile {
        public enum Format {
            CSV,
            XML,
            XLS;
        }

        Format format();
        String value();
    }

    @DataProvider
    public static Object[][] loadFromExternalFile(FrameworkMethod testMethod) {
        String testDataFile = testMethod.getAnnotation(ExternalFile.class).value();
        // Load the data from the external file here ...
        return new Object[][] { { testDataFile } };
    }

    @Test
    @UseDataProvider("loadFromExternalFile")
    @ExternalFile(format = ExternalFile.Format.CSV, value = "testdata.csv")
    public void testThatUsesUniversalDataProvider(String testData) {
        // Expect:
        assertThat(testData).isEqualTo("testdata.csv");
    }
```

Further examples can be found in [DataProviderJavaAcceptanceTest.java](/src/test/java/com/tngtech/test/java/junit/dataprovider/DataProviderJavaAcceptanceTest.java).


Release notes
-------------

### tbd. (tbd.)

* ...

### v1.8.0 (11-Jul-2014)

* added ```splitBy```, ```convertNulls``` and ```trimValues```` parameter to ```@DataProvider``` ([#24](/../../issues/24))
* more fault tolerant filtering instead of explicitly maintain a black or white list ([#27](/../../issues/27))
* ```@DataProvider``` method can now optionally access corresponding ```FrameworkMethod``` via parameter ([#28](/../../issues/28))

### v1.7.0 (20-Jun-2014)

* implemented [#20](/../../issues/20) to use ```@DataProvider``` directly providing test data for test method
* support any type which have a singl-argument ```String``` constructor for ```String[]``` data provider ([#26](/../../issues/26))
* removed some internal technical debts by refactoring ([#23](/../../issues/23)) and ([#25](/../../issues/25))

### v1.6.0 (26-Apr-2014)

* fixed bug in fix [#16](/../../issues/16) using IntelliJ ([#18](/../../issues/18) with merge [#19](/../../issues/19))
* added support for ```List<List<Object>>``` besides ```Object[][]``` for languages with native ```List``` support ([#17](/../../issues/17)) 

### v1.5.2 (10-Mar-2014)

* fixed ```IllegalArgumentException``` if using maven with category filter ([#16](/../../issues/16)) 

### v1.5.1 (07-Mar-2014)

* adjusted depedencies of uploaded 'pom.xml' (especially for JUnit => 'provided') ([#15](/../../issues/15))
* fixed that ```DataProviderRunner``` is incompatible with ```Categories``` ([#14](/../../issues/14))
* added LICENSE.TXT with Apache License 2.0 ([#13](/../../issues/13))
* fixed MANIFEST.MF
* better error message for not parsable filter

### v1.5.0 (03-May-2013)

* added ability that data provider is located in a completely different class

### v1.4.1 (02-May-2013)

* fixed ```ClassCastException``` in ```formatParameter``` if array is of primitive type

### v1.4.0 (01-May-2013)

* fixed issue with ability to run single junit-dataprovider row/method ([#11](/../../issues/11))
*  fixed appearence of "unrooted test" in JUnit Eclipse plugin ([#7](/../../issues/7)) 

### v1.3.0 (28-Apr-2013)

* ability to run single junit-dataprovider row/method ([#9](/../../issues/9))
* always show all parameters of junit-dataprovider in test method name ([#8](/../../issues/8))
* set username/password for MavenCentral conditionally ([#6](/../../issues/6))

### v1.2.0 (07-Mar-2013)

* enabled compatibility to junit v4.8.2
* added conditional signing [#4](/../../issues/4)


### v1.1.0 (02-Mar-2013)

* removed flag whether last parameter is an expected value
* refactoring that junt-dataprovider is testable and added a lot of tests
* transfered code to [TNG](https://github.com/TNG) ([#1](/../../issues/1), [#2](/../../issues/2) and [#3](/../../issues/3))


### v1.0.0 (22-Feb-2013)

* initial release


Eclipse template
----------------

* Name:                     dataProvider
* Context:                  Java type members
* Automatically insert:     false
* Description:              Insert a junit dataprovider method
* Use code formatter:	    false (unfortunately, this is a global setting for all templates)
* Pattern:

```
@${dataProviderType:newType(com.tngtech.java.junit.dataprovider.DataProvider)}
public static Object[][] dataProvider${Name}() {
    // @formatter:off
	return new Object[][] {
		{ ${cursor} },
	};
	// @formatter:on
}
```

Contributing
------------

You are very welcome to contribute by providing a patch/pull request.
