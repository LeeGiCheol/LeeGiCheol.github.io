---
title : "Jar파일 패키징 시 파일 객체 에러"
categories : 
    - java
tag :
    - etc
toc : true
---

```java
public void fileCallFail() throws IOException {
    Resource resource = new ClassPathResource(this.NUMBER_PATH);
    String path = resource.getFile().getAbsolutePath();
}
```

```java
Caused by: java.io.FileNotFoundException: class path resource [number.txt] cannot be resolved to absolute file path because it does not reside in the file system: jar:file:/workspace/file-exam/target/file-exam-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/number.txt
```

스프링 부트 환경에서 FileInputStream을 사용해서 개발을 하고 있었다.  
분명 로컬 환경에서는 정상적으로 File을 불러왔는데  
이상하게도 Jar 파일로 만들어서 실행을 시키면 getFile 호출 시 위 에러가 발생했다.  

```java
package org.springframework.util;

public abstract class ResourceUtils {

    public static File getFile(URL resourceUrl, String description) throws FileNotFoundException {
        Assert.notNull(resourceUrl, "Resource URL must not be null");
        if (!"file".equals(resourceUrl.getProtocol())) {
            throw new FileNotFoundException(description + " cannot be resolved to absolute file path because it does not reside in the file system: " + resourceUrl);
        } else {
            try {
                return new File(toURI(resourceUrl).getSchemeSpecificPart());
            } catch (URISyntaxException var3) {
                return new File(resourceUrl.getFile());
            }
        }
    }
 
    ...

}
```

내가 사용한 코드중 resource.getFile의 구현체이다.  
if문을 보면 protocol이 file이 아니라면 FileNotFoundException을 발생시킨다.  

IDE를 통해 실행을 했을때는  
**file://** 으로 시작하는 프로토콜을 사용하기 때문에 정상적으로 실행하게 된다.  

그러나 스프링부트를 통해 Jar 파일로 만들게 될 경우  
class file, resource 등을 하나로 합친 Jar 파일 한 개가 생성된다.  

이때 Jar 파일의 프로토콜은 **jar:** 로 시작하게 된다.  

에러 메시지를 다시 확인해본다면 쉽게 알 수 있다.  

```java
jar:file:/workspace/file-exam/target/file-exam-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/number.txt
```

**jar:file:경로** 로 실행하게 되어 FileNotFoundException 이 발생한 것을 알 수 있다.  


그렇다면 Jar 파일로 패키징 했을 때 Resource를 File로 받아올 수 없을까?  

이러한 경우 Resource를 InputStream으로 리턴받아서 사용할 수 있다.  

```java
public void fileCallSuccess() throws IOException {
    Resource resource = new ClassPathResource(this.NUMBER_PATH);
    InputStream is = resource.getInputStream();
    Reader reader = new InputStreamReader(is, StandardCharsets.UTF_8);
    BufferedReader br = new BufferedReader(reader);
}
```

아래와 같이 File 객체로도 받을 수 있다.  

```java
public void fileCallToFile() throws IOException {
    Resource resource = new ClassPathResource(this.NUMBER_PATH);
    InputStream is = resource.getInputStream();

    File numberFile = File.createTempFile("number", ".txt");
    
    try (FileOutputStream fos = new FileOutputStream(numberFile)) {
        int read;
        byte[] bytes = new byte[1024];

        while ((read = is.read(bytes)) != -1) {
            fos.write(bytes, 0, read);
        }
    }
}
```

조금 번거롭지만 배포 환경에 따라 장애가 발생할 수 있으니 기억해두자.  

[소스코드](https://github.com/LeeGiCheol/file-not-found-exception-exam)    

[참고한 글](https://sonegy.wordpress.com/2015/07/23/spring-boot-executable-jar%EC%97%90%EC%84%9C-file-resource-%EC%B2%98%EB%A6%AC/)  

---