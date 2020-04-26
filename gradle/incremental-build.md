# Incremental Build

## Task inputs, outputs, and dependencies

[JavaCompile](https://docs.gradle.org/2.5/dsl/org.gradle.api.tasks.compile.JavaCompile.html) 과 같은 Built-in 태스크는 입력 셋(Java 소스 파일)과 출력 셋(Class 파일)을 선언하고 있다. Gradle
은 이 정보를 사용해서 태스크가 up-to-date(최신 상태)인지 확인하고 태스크를 수행해야하는지 여부를 결정한다. **입력 혹은 출력이 변경되지 않은 경우 Gradle은 해당 태스크를 skip할 수 있다.** 이것을 Gradle의 `Incremental Build`라고 부른다.

Incremental Build를 활용하려면 Gradle에게 태스크의 입력 및 출력에 대한 정보를 제공해야 한다. (출력만 갖도록 태스크를 구성할 수도 있다.) 태스크를 실행하기 전에, Gradle은 출력을 확인하고 출력이 변경되지 않은 경우 태스크 실행을 skip 한다. 실제 빌드에서 태스크는 보통 입력으로 source file(소스 파일), resource(리소스), properties(속성)를 포함한다. Gradle은 작업을 실행하기 전에 입력이나 출력이 변경되지 않았는지 확인한다. ([14.9. Skipping tasks that are up-to-date](https://docs.gradle.org/2.5/userguide/more_about_tasks.html#sec:up_to_date_checks)

**종종 태스크의 출력이 다른 태스크의 입력으로 사용되기도 한다. 이 태스크들 사이의 순서를 정하는 것은 중요하다.** 그렇지 않으면 태스크가 잘못된 순서로 실행되거나 전혀 실행되지 않을 수도 있다. Gradle은 build script
에서 태스크가 정의된 순서에 의존하지 않는다. 새로운 태스크의 순서가 정해지지 않으므로 실행 순서가 빌드때마다 바뀔 수 있다. `consumer.dependsOn producer`를 이용해서 서로 다른 태스크 간에 종속성을 선언해서 두 태스크 간의 순서를 명시적으로 지정할 수도 있다.

## Declaring explicit task dependencies

일반적인 패턴을 포함한 예제 프로젝트를 살펴보자. 이 프로젝트의 경우 `generator` 태스크의 출력을 zip 파일로 만들어낸다. `generator` 태스크는 숫자가 증가하는 파일을 생성한다.

**build.gradle**
```groovy
apply plugin: 'base'

task generator() {
    doLast {
        def generatedFileDir = file("$buildDir/generated")
        generatedFileDir.mkdirs()
        for (int i=0; i<10; i++) {
            new File(generatedFileDir, "${i}.txt").text = i
        }
    }
}

task zip(type: Zip) {
    dependsOn generator
    from "$buildDir/generated"
}
```

빌드는 작동하지만 build script에는 몇 가지 문제가 있다. `generator` 태스크의 출력 디렉토리는 `zip` 태스크에서 반복되며 `zip` 태스크의 종속성은 `dependOn`으로 명시적으로 설정되어 있다. Gradle은 `generator` 태스크를 매번 실행한다. Gradle의 up-to-date 체크 검사가 Make와 같은 다른 tool과 어떻게 다른지 살펴보자.

Gradle은 파일의 timestamp 대신 입력 및 출력의 checksum을 비교한다. `generator` 태스크가 매번 실행되고 모든 출력 파일이 overwrite 되더라도 내용은 변경되지 않기 때문에 `zip` 태스크는 다시 실행할 필요가 없다. `zip` 태스크의 입력 checksum은 변경되지 않았다. up-to-date 태스크를 skip하면 Gradle은 불필요한 작업을 피함으로써 속도를 향상 시킬 수 있다.

## Declaring task inputs and outputs

`generator` 태스크가 매번 실행되는 이유를 알아보자. `-info` 옵션과 함께 빌드를 해보면 이유를 알 수 있다.

```
Executing task ':generator' (up-to-date check took 0.0 secs) due to:
Task has not declared any outputs.
```

기본적으로 태스크에 출력이 없으면 오래된(out-of-date)것으로 간주한다. 출력은 [TaskOutputs](https://docs.gradle.org/2.5/javadoc/org/gradle/api/tasks/TaskOutputs.html) 으로 선언된다. 태스크 출력은 파일 혹은 디렉토리가 될 수 있다.

**build.gradle**
```groovy
task generator() {
    def generatedFileDir = file("$buildDir/generated")
    outputs.dir generatedFileDir
    doLast {
        generatedFileDir.mkdirs()
        for (int i=0; i<10; i++) {
            new File(generatedFileDir, "${i}.txt").text = i
        }
    }
}
```

빌드를 두 번 더 실행하면 `generator` 태스크가 첫 번째 실행 후 최신 상태임을 알 수 있다.

```
Skipping task ':generator' as it is up-to-date (took 0.007 secs).
```

그러나 새로운 문제가 발생한다. 생성되는 파일의 수를 늘려도 `generator` 태스크는 다시 실행되지 않는다. clean 후 빌드를 실행해서 이 문제를 해결할 수는 있지만, 이 해결 방법은 오류가 발생하기 쉽다.

Gradle에게 `generator` 태스크에 영향을 줄 수 있는 것을 알려주고 필요시 다시 실행하도록 할 수 있다. [TaskInputs](https://docs.gradle.org/2.5/javadoc/org/gradle/api/tasks/TaskInputs.html) 을 사용해서 입력 파일 뿐만 아니라 태스크에 대한 입력으로 properties를 선언할 수도 있다. 이러한 입력 중 하나라도 변경되면 Gradle은 태스크를 실행하게 될 것이다. 

**build.gradle**
```groovy
task generator() {
    def fileCount = 10
    inputs.property "fileCount", fileCount
    def generatedFileDir = file("$buildDir/generated")
    outputs.dir generatedFileDir
    doLast {
        generatedFileDir.mkdirs()
        for (int i=0; i<fileCount; i++) {
            new File(generatedFileDir, "${i}.txt").text = i
        }
    }
}
```

```
Executing task ':generator' (up-to-date check took 0.007 secs) due to:
Value of input property 'fileCount' has changed for task ':generator'
```
