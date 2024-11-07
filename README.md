# micronaut-jdbc-tracing
Reproduce bug when adding Micronaut tracing JDBC instrumentation

I've faced an issue after adding micronaut tracing JDBC instrumentation
(`implementation('io.micronaut.tracing:micronaut-tracing-opentelemetry-jdbc:6.9.0')`) to a Micronaut application.
Locally it all works well, but when I deploy the application to a Kubernetes cluster, it does not start any more.

To verify that the application without the dependency starts, comment out the dependency in `build.gradle`, then:
- build a docker container with `./gradlew dockerBuild`
- tag the container image according to the docker repository that you will push it to, keeping in mind that the repository should be accessible from the Kubernetes
  ```bash
  docker tag jdbc-tracing:latest <your-docker-repository>/jdbc-tracing:latest
  ```
- push the container image to the docker repository `docker push <your-docker-repository>/jdbc-tracing:latest`
- update the pod.yaml manifest with the new image name
- apply the manifest to the Kubernetes cluster `kubectl apply -f pod.yaml`
- check the logs of the pod `kubectl logs <pod-name>` where you should see that the application does start
- You should see a log output like this
  ```
     __  __ _                                  _   
    |  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_
    | |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
    | |  | | | (__| | | (_) | | | | (_| | |_| | |_
    |_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
    09:45:28.274 [main] INFO  i.m.c.DefaultApplicationContext$RuntimeConfiguredEnvironment - Established active environments: [k8s, cloud]
    09:45:29.427 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Starting...
    09:45:30.049 [main] INFO  com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn0: url=jdbc:h2:mem:devDb user=SA
    09:45:30.053 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Start completed.
    09:45:30.961 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 4743ms. Server Running: http://jdbc-tracing:8080
  ```

To reproduce the issue, uncomment the dependency in `build.gradle`, then:
- build a docker container with `./gradlew dockerBuild`
- tag the container image according to the docker repository that you will push it to, keeping in mind that the repository should be accessible from the Kubernetes
  ```bash
  docker tag jdbc-tracing:latest <your-docker-repository>/jdbc-tracing:latest-with-bug
  ```
- push the container image to the docker repository `docker push <your-docker-repository>/jdbc-tracing:latest-with-bug`
- update the pod.yaml manifest with the new image name
- apply the manifest to the Kubernetes cluster `kubectl apply -f pod.yaml`
- check the logs of the pod `kubectl logs <pod-name>` where you should see that the application does not start
- You should see a log output like this
  ```
    __  __ _                                  _   
    |  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_
    | |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
    | |  | | | (__| | | (_) | | | | (_| | |_| | |_
    |_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
    09:44:25.329 [main] INFO  i.m.c.DefaultApplicationContext$RuntimeConfiguredEnvironment - Established active environments: [k8s, cloud]
    09:44:25.519 [main] ERROR io.micronaut.runtime.Micronaut - Error starting Micronaut server: No bean of type [io.micronaut.context.event.ApplicationEventPublisher<io.micronaut.context.event.StartupEvent>] exists.
    io.micronaut.context.exceptions.NoSuchBeanException: No bean of type [io.micronaut.context.event.ApplicationEventPublisher<io.micronaut.context.event.StartupEvent>] exists.
    at io.micronaut.context.DefaultBeanContext.newNoSuchBeanException(DefaultBeanContext.java:2798)
    at io.micronaut.context.DefaultApplicationContext.newNoSuchBeanException(DefaultApplicationContext.java:329)
    at io.micronaut.context.DefaultBeanContext.resolveBeanRegistration(DefaultBeanContext.java:2761)
    at io.micronaut.context.DefaultBeanContext.getBean(DefaultBeanContext.java:1745)
    at io.micronaut.context.DefaultBeanContext.getBean(DefaultBeanContext.java:842)
    at io.micronaut.context.BeanLocator.getBean(BeanLocator.java:97)
    at io.micronaut.context.DefaultBeanContext.publishEvent(DefaultBeanContext.java:1831)
    at io.micronaut.context.DefaultBeanContext.start(DefaultBeanContext.java:360)
    at io.micronaut.context.DefaultApplicationContext.start(DefaultApplicationContext.java:216)
    at io.micronaut.runtime.Micronaut.start(Micronaut.java:75)
    at io.micronaut.runtime.Micronaut.run(Micronaut.java:334)
    at io.micronaut.runtime.Micronaut.run(Micronaut.java:320)
    at jdbc.tracing.Application.main(Application.java:8)
  ```


## Micronaut 4.6.3 Documentation

- [User Guide](https://docs.micronaut.io/4.6.3/guide/index.html)
- [API Reference](https://docs.micronaut.io/4.6.3/api/index.html)
- [Configuration Reference](https://docs.micronaut.io/4.6.3/guide/configurationreference.html)
- [Micronaut Guides](https://guides.micronaut.io/index.html)
---

- [Shadow Gradle Plugin](https://plugins.gradle.org/plugin/com.github.johnrengelman.shadow)
- [Micronaut Gradle Plugin documentation](https://micronaut-projects.github.io/micronaut-gradle-plugin/latest/)
- [GraalVM Gradle Plugin documentation](https://graalvm.github.io/native-build-tools/latest/gradle-plugin.html)
## Feature tracing-opentelemetry-exporter-otlp documentation

- [Micronaut OpenTelemetry Exporter OTLP documentation](http://localhost/micronaut-tracing/guide/index.html#opentelemetry)

- [https://opentelemetry.io](https://opentelemetry.io)


## Feature micronaut-aot documentation

- [Micronaut AOT documentation](https://micronaut-projects.github.io/micronaut-aot/latest/guide/)


## Feature data-jdbc documentation

- [Micronaut Data JDBC documentation](https://micronaut-projects.github.io/micronaut-data/latest/guide/index.html#jdbc)


## Feature serialization-jackson documentation

- [Micronaut Serialization Jackson Core documentation](https://micronaut-projects.github.io/micronaut-serialization/latest/guide/)


## Feature jdbc-hikari documentation

- [Micronaut Hikari JDBC Connection Pool documentation](https://micronaut-projects.github.io/micronaut-sql/latest/guide/index.html#jdbc)
