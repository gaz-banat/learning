
-----------------------
JIB
-----------------------

https://cloud.google.com/java/getting-started/jib


Jib builds containers without using a Dockerfile or requiring a Docker installation. You can use Jib in the Jib plugins for Maven or Gradle, or you can use the Jib Java library.

Jib organizes your application into distinct layers; dependencies, resources, and classes; and utilizes Docker image layer caching to keep builds fast by only rebuilding changes. 
Jib's layer organization and small base image keeps overall image size small which improves performance and portability.


Maven Plugin for JIB
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>2.8.0</version>
  <configuration>
    <to>
      <image>gcr.io/PROJECT/IMAGE</image>
    </to>
  </configuration>
</plugin>
