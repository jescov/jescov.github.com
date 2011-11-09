---
layout: default
title: JesCov - Examples
header: Examples
---

## Running from the command line

The easiest - but least flexible - way of running JesCov is on the command line. This will allow you to execute arbitrary JavaScript and have coverage be captured for everything executed. It will generate a file called `jescov.json.ser` that contains the coverage information. This file can later be used by the Ant tasks, for example.

Provided you have the JesCov jar file in the current directory you just run it using the Java command and give it all the JavaScript files to execute on the command line:

{% highlight console %}
$ java -jar jescov-0.0.1.jar one.js two.js three.js
{% endhighlight %}

## Using Ant to generate reports

In order to get useful reports out of JesCov, the current implementation has a project called JesCov-ant that exposes an Ant task. This Ant task can be used to either generate HTML or XML output. The HTML output will contain annotated source files as well as coverage information, while the XML format follows the Cobertura DTDs.

Provided you have the JesCov and JesCov-ant jar files in the current directory you can use a fragment like this to generate XML from a pre-existing `jescov.json.ser` file:

{% highlight xml %}
<project basedir="." default="coverage" name="example">
  <path id="jescov.classes">
    <pathelement path="jescov-0.0.1.jar" />
    <pathelement path="jescov-ant-0.0.1.jar" />
  </path>

  <taskdef classpathref="jescov.classes" resource="com/olabini/jescov/ant/tasks.properties"/>

  <target name="coverage">
    <jescov-report format="xml"/>
  </target>
</project>
{% endhighlight %}

This will generate a file called `coverage.xml` in the current directory. The xml format can be given several different attributes:

- `datafile`  -  if you have given your file a different name than `jescov.json.ser` or if it's in some other directory, you can specify it with this attribute.
- `destfile`  -  you can generate a file in another directory or with a different name by setting this attribute. The default is `coverage.xml` in the current directory.

You can also generate a HTML report based on the coverage information and source code:

{% highlight xml %}
<project basedir="." default="coverage" name="example">
  <path id="jescov.classes">
    <pathelement path="jescov-0.0.1.jar" />
    <pathelement path="jescov-ant-0.0.1.jar" />
  </path>

  <taskdef classpathref="jescov.classes" resource="com/olabini/jescov/ant/tasks.properties"/>

  <target name="coverage">
    <jescov-report format="html"/>
  </target>
</project>
{% endhighlight %}

The HTML format will create a directory called `coverage-report` in the current directory, by default. It will also assume that the full path to the source files that was used to generate the coverage information can be reached from the current directory. You can change these attributes:

- `datafile`  -  if you have given your file a different name than `jescov.json.ser` or if it's in some other directory, you can specify it with this attribute.
- `destdir`  -  you can generate the HTML report into a different directory by giving this attribute. All non-existing directories will be created. The default is `coverage-report`.
- `srcdir`  -  if the current directory is not the reference point for the source files, give this attribute to specify where to find the source. If you get exception about not being able to find JavaScript files, this is likely the attribute you need.

## Using the Jasmine JUnit runner

The first step in actually using JesCov to run Jasmine test is to set everything up so it works with the Jasmine JUnit test runner. Once that's up and running (I usually create one single JUnit class that runs all Jasmine specs) you need to replace the JasmineTestRunner class used with the one from JesCov. A typical file might look like this:

{% highlight java %}
import org.junit.runner.RunWith;

import be.klak.junit.jasmine.JasmineSuite;
import com.olabini.jescov.junit.JasmineTestRunner;

@RunWith(JasmineTestRunner.class)
@JasmineSuite(sources = {
                          "one.js",
                          "two.js",
                          "three.js",
                          "four.js"
                        }, 
               sourcesRootDir = "js", 
               jsRootDir = "jstest" , 
               specs = {
                         "oneSpec.js", 
                         "twoSpec.js", 
                         "fourSpec.js"
                       })
public class AllJavaScriptTest {
}

{% endhighlight %}

The big thing to notice here is that the `JasmineTestRunner` is coming from the JesCov package, and not from the `be.klak.junit.jasmine` package. Now, when defined like this, it will run the Jasmine tests without any coverage checking. This is because you usually want to be able to use the same source definition as part of your CI and when later running it through coverage metric tools. You can turn coverage collection on by using Java System properties. You can also configure other properties easily. By default, the coverage collection will ignore all the spec files and also the Jasmine core files. The goal is to give you coverage information for your business code only.

In order to actually collect coverage, you could do it by running from inside of Ant, something like this:

{% highlight xml %}
<junit>
  <sysproperty key="com.olabini.jescov.enabled" value="true" />
  <batchtest>
    <fileset dir="src/test">
      <include name="com/example/AllJavaScriptTest.java"/>
    </fileset>
  </batchtest>        
</junit>
{% endhighlight %}

There are several different system properties that can be used to further change the behavior of the collection of data:

- `com.olabini.jescov.enabled`  -  if set to true, will collect coverage information
- `com.olabini.jescov.ignoreFiles`  -  a comma separated list of files to ignore. You need to use the full file name exactly as it gets exposed in the JSON data file to ignore it
- `com.olabini.jescov.jsonOutputFile`  -  the name of the file to use for the output data. Defaults to `jescov.json.ser`.
- `com.olabini.jescov.jsonMerge`  -  if set to true, will read in a possibly existing data file and merge the current run with that data before writing it back. This is useful if you want to run several different suites separately but combine the results.

