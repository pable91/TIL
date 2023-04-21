
# 현상
- 최근에 프로젝트에 Spring-Rest-Docs를 추가하였다. 하지만 로컬에서 개발 완료 및 테스트 후 github action에서 build 하는데 다음과 같은 에러가 발생했다.
![b1](https://user-images.githubusercontent.com/22884224/233582610-33050736-f88a-4ef2-a030-e50e24c9e6a8.png)
- 빨간색 부분이 에러의 원인이다. 끝에 부분을 해석해보면 generated-snippets 디렉토리가 없다고 한다. generated-snippets 디렉토리는 프로젝트에서 작성한 test code들이 성공해야 생성하고 그 안에 개별 snippet들도 생성된다. 

# 삽질1
- 나는 맨 처음에 그 윗부분에 에러로그로 나온 "Deprecated Gradle features were use in this~~~~~~~ Gradle 8.0"을 보고, gradle 버전 호환이 안맞아서 디렉토리를 못찾는다고 생각했다.
당시 gradle 버전은 7.6이었고 구글링을 해도 버전호환으로 인한 에러 발생이 많다는걸 읽었기 때문이다. 하지만 gradle 버전을 변경해봐도 해당 에러는 계속 발생하였다.

# 해결?
- 버전은 해결책이 아니니 다른 방법으로 접근해봐야했다. 그러고 드는 생각이 혹시 test자체가 실행이 안되는건 아닐까? test 실행이 안되면 당연히 성공을 못할 것이고 generated-snippets 디렉토리는 생성되지않는다. 위와 같은 생각을 하고 gradle-publish.yml에서 빌드하는 명령어를 확인했다.   
![b2](https://user-images.githubusercontent.com/22884224/233587756-4b4b175d-6ae2-49ab-8433-0ad67ef27d20.png)
- -x라는 명령어는 해당 순서를 제외하라는 의미다. 즉, test 단계를 건너 뛰라는 명령어다. 해당 부분이 오류라고 생각하고 삭제하고 다시 빌드하였다.

# 다시 에러
- 이번에는 다른 에러가 발생했다.   
![b3](https://user-images.githubusercontent.com/22884224/233588769-b20137ec-0021-450f-b744-be1bd25d0a56.png)
- 빨간 박스 위에 test가 실행되는걸 보니 -x 명령어가 성공적으로 제거되었다는것은 알 수 있다. 근데 해당 테스트 코드들은 왜 에러가 발생하는하는거지? 
- 에러가 발생한 테스트 코드들은 모두 spring-rest-docs를 위핸 mvc 테스트 코드다. 하지만 다른 테스트 코드(ex.도메인)들은 에러를 발생시키지 않았다. 
- mvc 또는 spring-rest-docs쪽에서 문제가 발생했다는것을 유추할 수 있다.

# 이번에는
- 해당 에러 라인으로 가서 확인해봤다.
![b4](https://user-images.githubusercontent.com/22884224/233589974-2e203681-6988-4f65-8d72-fb6043b315f1.png)
- .andDo에서 에러가 발생한다. 다른 테스트 코드들도 동일한 지점에서 에러가 발생했다. 
- .andDo는 mvc 테스트 성공 후에 동작하고 snippets을 생성하는 부분이다. 즉, mvc와는 무관하고 spring-rest-docs가 유력한 에러의 원인이라고 생각했다.
- "혹시 build할때 spring-rest-docs 관련 라이브러리를 못찾아오나?" 라는 생각이 들었다.

# 드디어 해결
```groovy


ext {
    snippetsDir = file('build/generated-snippets')
}

test {
    outputs.dir snippetsDir
    useJUnitPlatform()
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExt'
    dependsOn test
}
task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}
bootJar {
    dependsOn copyDocument
}
```
- 무엇이 문제일까? 
- build 될때 로그를 보면 test가 실행하고 있을때 spring-rest-docs 에러가 발생한다. 즉, test 스코프 내부의 useJUnitPlatform() 부분에서 에러가 발생한다고 유추가능하다.
- 에러가 발생하는 이유는 useJUnitPlatform전에 spring-rest-docs 관련 라이브러리들이 설정되야지만 useJUnitPlatform안에 spring-rest-docs 테스트 들이 통과할 것이다. 하지만 test스코프안에는 우리가 필요로 하는 라이브러리 설정이 없다. 그래서 에러가 발생한 것이다.
- 해당 설정은 asciidoctor 스코프 안에 configurations 'asciidoctorExt'에서 해준다. 
- 요약하자면 test 스코프에서는 test 관련 코드(=useJUnitPlatform)을 실행시키면 관련 라이브러리가 없어서 에러가 난다. 해당 라이브러리를 설정해주고 test 스코프를 실행시키자.
- 수정한 코드다.
``` groovy
ext {
    snippetsDir = file('build/generated-snippets')
}

test {
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExtensions'
    dependsOn test
}

task copyDocument(type: Copy) {
    dependsOn asciidoctor

    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}

bootJar {
    dependsOn copyDocument
}
```

