---
layout: default
title: JesCov - JavaScript code coverage
header: JesCov - JavaScript code coverage
---

## What is it

JesCov is an open source tool for measuring code coverage of JavaScript. It makes it possible to find out both line and branch coverage from any ES3-compliant source code. It is not tied to any particular testing framework, but the only current integration point is for Jasmine. The general approach is based on a JSON data interchange format, which means some of the reporting tools could potentially be used against data collected in some other way than running the core JesCov project.

JesCov is a Java project and at the moment requires you to run your code using Rhino. If you use JesCov to figure out how well your unit tests cover your code, that should work well for many circumstances.

## How does it work

When running code, JesCov will use the Rhino debugger interface to hook into the loading of source files. It will then transform the original source file using an Antlr grammer that inserts coverage-collecting statements. This grammer is based on one that the js-test-driver uses, but heavily modified to accomodate collection of branch coverage as well as line coverage. After modifying the source code, everything will run as usual. At the end of the run, the data collected will be dumped to disk in a JSON-format. This format can then be converted into either Cobertura style XML or a HTML report which annotates source files with coverage information. This functionality exists in the core project, but currently the only way of accessing it is through an Ant-task.

## Future

There are lots of limitations in the current code. Some of the things I would like to do if people find this project useful is to make it possible to run using the V8 debugger interface as well as with Rhino. Getting this to work might be an interesting challenge. I would also like to make it possible to statically rewrite JavaScript files. Finally, integration points with other test frameworks would be a good idea. At some point it might be good to push all this stuff to a Maven repository too, but that's something I would love for someone else to contribute with!


