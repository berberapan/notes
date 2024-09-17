In IntelliJ you choose the Spring Boot option in the left column under the Generators category.

Spring recommends using **Gradle** as the build tool.

For the language you're using choose a stable version. Newer ones might not have all tools updated to support them yet.

![[springbootstart.png]]
##### Dependencies
After clicking next you'll get an option to choose dependencies to start with. You can add them later on too with Gradle. Repositories can be found on Maven central.

![[springdependencies.png]]

![[gradledependencies.png]]

##### Main class

The main method will be created with a Spring Boot Application annotation. This is needed to be able to run a Spring application.

![[springmainmethod.png]]