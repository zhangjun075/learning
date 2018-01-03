# startup remind
java -jar -DhttpPort=8080 -Dhudson.util.ProcessTree.disable=true  jenkins.war

# app startup command
BUILD_ID=dontKillMe nohup java -jar  /Users/junzhang/Downloads/jenkins-shell/demo.jar  > server.log 2>&1 &