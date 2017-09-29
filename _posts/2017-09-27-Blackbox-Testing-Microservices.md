---
layout: post
title: Blackbox Testing Microservices with Kotlin and RestAssured
---

Learn how to set up a "[suzerain](https://www.merriam-webster.com/dictionary/suzerain)" project to author 
User Journeys for Blackbox Testing.

If you just want to dig into the code, start here: https://github.com/revelfire/blackboxmicroservicetesting

> Specific instructions for running this code are in the readme.md of that repo and will not be repeated here.

If you got here without reading [the companion article, click here](https://www.linkedin.com/pulse/blackbox-api-testing-kotlin-restful-microservices-chris-mathias/)

## Stack
* Gradle
* Kotlin
* KotlinTest
* Intellij Kotlin Plugin
* RestAssured

That's really about it. 

## Structure

We break the code up into folders that match structure-specific functionality.

### /functions
This folder is specifically for Functions that act as clients for
 the API. There should generally not be tests here, just Functions.

### /tests
This folder is specifically for single Tests/scenarios that validate a
 particular API Function invocation. These will include positive and negative
 test cases.

### /journeys
This folder is specifically for tests that composite multiple Functions
into User Journeys that should ideally mirror application use cases
that have documented acceptance criteria. 

## Pieces

### build.gradle

This file stitches all the pieces together and gives us the simple text execution.

```groovy
buildscript {
    ext.kotlin_version = "1.1.4"
    repositories {
        mavenLocal()
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

repositories {
    mavenLocal()
    mavenCentral()
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'maven'
apply plugin: 'kotlin'

group 'com.raken.test.api'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile group: 'org.jetbrains.kotlin', name: 'kotlin-stdlib', version: "$kotlin_version"

    //Include our app lib so we have access to models (not mandatory, but helpful)
    compile group: 'com.revelfire', name: 'car-service', version: '1.0-SNAPSHOT'

    compile group: 'com.fasterxml.jackson.module', name: 'jackson-module-kotlin', version: '2.9.0'

    compile group: 'io.rest-assured', name: 'rest-assured', version: '3.0.2'
    compile group: 'org.hamcrest', name: 'hamcrest-core', version: '1.3'
    compile group: 'org.jetbrains.kotlin', name: 'kotlin-stdlib', version: "$kotlin_version"
    compile group: 'io.kotlintest', name: 'kotlintest', version: "$kotlin_version"
    compile group: 'com.github.javafaker', name: 'javafaker', version: "0.13"
    
    testCompile group: 'io.kotlintest', name: 'kotlintest', version: '2.0.1'
}

tasks.test.doFirst {
    def includeTags = System.properties['includeTags']
    if(includeTags) {
        systemProperty 'includeTags', includeTags
    }
    testLogging.events = ['passed','failed','skipped']
}

tasks.test.outputs.upToDateWhen { false }
```

### TestBase.kt
This file has some basic underpinnings for test execution and hooks for 
overarching functionality.

```kotlin
package com.raken.test.api

//...imports omitted...

//Tags to help you compartmentalize your testing
object QuickRun : Tag()
object Integration : Tag()    //Represents a single unit of testing an API endpoint (various scenarios possibly)
object Journey : Tag()        //Represents a group of tests that mimic a user behavior/journey through the app

object TestBase {

    var props:Properties = Properties();

    init {
        props = readProperties()
        validateProperties()
    }

    fun carServiceHost():String = "${getProperty("service.car-service.url")}:${getProperty("service.car-service.port")}"
    fun foodServiceHost():String = "${getProperty("service.food-service.url")}:${getProperty("service.food-service.port")}"

    fun getPropertyAsInt(key:String):Int {
        val value = getProperty(key)
        value?.let {
            return value.toInt()
        }
        throw RuntimeException("Property ${key} not found!")
    }

    fun getPropertyAsBoolean(key:String):Boolean {
        val value = getProperty(key)
        value?.let {
            return value.toBoolean()
        }
        throw RuntimeException("Property ${key} not found!")
    }

    fun getProperty(key:String):String? {
        System.getProperty(key)?.let {
            return System.getProperty(key)
        }
        return props[key] as String
    }

    fun getRequiredProperty(key:String):String? {
        System.getProperty(key)?.let {
            return System.getProperty(key)
        }
        props[key]?.let {
            return props[key] as String
        }
        throw RuntimeException("Property ${key} marked as required but not present!")
    }

    fun readProperties():Properties = Properties().apply {
        FileInputStream("src/test/resources/testing.conf").use { fis ->
            load(fis)
        }
    }

}

//Kotlintest interceptors that fire before the entire suite, and after.
object GlobalTestSuiteInitializer : ProjectConfig() {

    private var started: Long = 0

    override fun beforeAll() {
        RestAssured.config = RestAssuredConfig.config().objectMapperConfig(ObjectMapperConfig.objectMapperConfig().jackson2ObjectMapperFactory(objectMapperFactory))
        started = System.currentTimeMillis()
    }

    override fun afterAll() {
        val time = System.currentTimeMillis() - started
        println("overall time [ms]: " + time)
    }
}
```

Note in particular the *GlobalTestSuiteInitializer* above which can be used for any
sort of suite-wide bootstrapping.

### RestAssured Configuration (RestAssuredSupport.kt)
We have a couple of twiddly bits we need to set up for JSON manipulation and other things.

```kotlin
package app.util

//...imports omitted...

interface RestAssuredSupport {
    fun RequestSpecification.When(): RequestSpecification {
        return this.`when`()
    }
    
    fun ResponseSpecification.When(): RequestSender {
        return this.`when`()
    }

}

//http://www.programcreek.com/java-api-examples/index.php?api=com.jayway.restassured.config.RestAssuredConfig

object ObjectMapperConfigurator {
    val objectMapper = ObjectMapper().registerModule(KotlinModule())
    init {
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
        objectMapper.configure(DeserializationFeature.FAIL_ON_MISSING_CREATOR_PROPERTIES, false)

        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY)
    }
    fun get():ObjectMapper {
        return objectMapper
    }
}

val objectMapperFactory = object : Jackson2ObjectMapperFactory {

    override fun create(cls: Class<*>?, charset: String?): ObjectMapper {
        return ObjectMapperConfigurator.get()
    }
}

```

## Finally

```commandline
gradle test
```

Done.
