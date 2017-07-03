---
title: "JVM 스크립트 엔진 테스트"
tags: ["JVM", "scripting"]
slug: "JVM_dynamic_scripting"
date: 2017-07-01T16:15:35+09:00
---
<!--more-->
# 개요
Java Scripting Engine의 성능 측정

# 목적
JVM내의 비즈니스 로직을 서버 재시작 없이 변경하려 함

## 방법
kotlin 웹서버에 더미 서비스, javascript(ecmascript)런타임으로 구동되는 서비스, kotlin런타임으로 구동되는 서비스를 올린 후 load generator로 성능 측정

# 코드
* build.gradle

```prettyprint
...
dependencies {
    compile group: 'org.jetbrains.kotlin', name: 'kotlin-stdlib', version: kotlinVersion
    compile (group: 'org.jetbrains.kotlin', name: 'kotlin-script-util', version: kotlinVersion) {
        exclude group: 'org.jetbrains.kotlin', module: 'kotlin-stdlib'
    }
...
```

* src/main/resources/META-INF/services/javax.script.ScriptEngineFactory

```prettyprint
org.jetbrains.kotlin.script.jsr223.KotlinJsr223JvmLocalScriptEngineFactory
```

* src/main/kotlin/Main.kt

```prettyprint
import spark.Spark.get
import javax.script.Invocable
import javax.script.ScriptEngine
import javax.script.ScriptEngineManager

class DSLClass {
    val manager = ScriptEngineManager()
    val kotlinEngine: ScriptEngine = manager.getEngineByExtension("kts")
    val jsEngine: ScriptEngine = manager.getEngineByExtension("js")

    init {
        get("/dummy", { _, _ ->
            "hello there"
        })

        get("/kotlin", { _, _ ->
            val script = """fun sum(a: Int, b: Int): Int = a + b"""
            kotlinEngine.eval(script)

            val inv = kotlinEngine as Invocable
            val result = inv.invokeFunction("sum", 1, 2)

            "<h1>kotlin result: ${result}</h1>"
        })

        get("/js", { _, _ ->
            val script = """function sum(a, b) { return a + b; }"""
            jsEngine.eval(script)

            val inv = jsEngine as Invocable
            val result = inv.invokeFunction("sum", 1, 2)

            "<h1>js result: ${result}</h1>"
        })
    }
}

fun main(args: Array<String>) {
    DSLClass()
}
```

# 결과
kotlin dynamic scripting은 아직 쓸 수준이 못됨.

## dummy
```prettyprint
ls@mark-win10:~$ echo "GET http://localhost:4567/dummy" | vegeta attack -duration=5s | tee results.bin | vegeta report
Requests      [total, rate]            250, 50.20
Duration      [total, attack, wait]    4.9817705s, 4.9799993s, 1.7712ms
Latencies     [mean, 50, 95, 99, max]  2.246649ms, 2.1812ms, 2.8538ms, 3.9128ms, 5.0676ms
Bytes In      [total, mean]            2750, 11.00
Bytes Out     [total, mean]            0, 0.00
Success       [ratio]                  100.00%
Status Codes  [code:count]             200:250
Error Set:
```

## javascript
```prettyprint
ls@mark-win10:~$ echo "GET http://localhost:4567/js" | vegeta attack -duration=5s | tee results.bin | vegeta report
Requests      [total, rate]            250, 50.20
Duration      [total, attack, wait]    4.982487s, 4.9799993s, 2.4877ms
Latencies     [mean, 50, 95, 99, max]  2.538705ms, 2.4975ms, 3.0654ms, 4.2255ms, 8.6948ms
Bytes In      [total, mean]            5750, 23.00
Bytes Out     [total, mean]            0, 0.00
Success       [ratio]                  100.00%
Status Codes  [code:count]             200:250
Error Set:
```

## kotlin
```prettyprint
ls@mark-win10:~$ echo "GET http://localhost:4567/kotlin" | vegeta attack -duration=5s | tee results.bin | vegeta report
Requests      [total, rate]            250, 50.20
Duration      [total, attack, wait]    34.9811253s, 4.9799993s, 30.001126s
Latencies     [mean, 50, 95, 99, max]  29.799915256s, 30.0014174s, 30.0019209s, 30.0022531s, 30.0026295s
Bytes In      [total, mean]            50, 0.20
Bytes Out     [total, mean]            0, 0.00
Success       [ratio]                  0.80%
Status Codes  [code:count]             200:2  0:248
Error Set:
Get http://localhost:4567/kotlin: net/http: timeout awaiting response headers
```

# 참조
* Java Scripting Programmer's Guide : http://docs.oracle.com/javase/7/docs/technotes/guides/scripting/programmer_guide/
* Java 8 Nashorn Tutorial : http://winterbe.com/posts/2014/04/05/java8-nashorn-tutorial/
* vegeta / HTTP load testing tool and library : https://github.com/tsenart/vegeta
