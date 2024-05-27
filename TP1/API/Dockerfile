# TODO: Choose a java JRE
FROM openjdk:21-slim
# TODO:  Add the compiled java (aka bytecode, aka .class)
COPY ./Main.class /usr/src/myapp/
WORKDIR /usr/src/myapp
# TODO: Run the Java with: “java Main” command.
CMD ["java", "Main"]

# FROM eclipse-temurin:17-jdk-alpine
# # Build Main.java with JDK
# # TODO : in next steps (not now)

# FROM eclipse-temurin:17-jre-alpine
# # Copy resource from previous stage
# COPY --from=0 /usr/src/Main.class .
# # Run java code with the JRE
# # TODO : in next steps (not now)
