#项目所依赖的镜像
FROM java:8
# 将maven构建好的jar添加到镜像中
COPY ../../target/*.jar ./
ADD *.jar app.jar
# 暴露的端口号
EXPOSE 8072
# 镜像所执行的命令
ENTRYPOINT ["java","-jar","/app.jar"]
