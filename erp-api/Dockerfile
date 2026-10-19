# ============ 构建阶段：Maven 多模块编译打包 ============
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /build

# 先只拷贝 pom，让依赖层可缓存（源码变更不会重新下载依赖）
# Maven 多模块需要完整的 pom 层级结构
COPY pom.xml ./
COPY common/pom.xml common/
COPY erp-api/pom.xml erp-api/
COPY mapper/pom.xml mapper/
COPY model/pom.xml model/
COPY open-sdk/pom.xml open-sdk/
COPY security/pom.xml security/
COPY service/pom.xml service/

# 预下载依赖；个别插件解析失败不影响后续 package，故容忍失败
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -ntp dependency:go-offline -DskipTests || true

# 再拷贝源码
COPY common/src common/src
COPY erp-api/src erp-api/src
COPY mapper/src mapper/src
COPY model/src model/src
COPY open-sdk/src open-sdk/src
COPY security/src security/src
COPY service/src service/src

# 只构建 erp-api 及其依赖模块
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -ntp -pl erp-api -am clean package -DskipTests

# ============ 运行阶段：仅携带 jar ============
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=build /build/erp-api/target/erp-api-4.1.0.jar /app/erp-api.jar

ENV TZ=Asia/Shanghai
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -Duser.timezone=Asia/Shanghai -Dfile.encoding=UTF-8"

EXPOSE 6149

# exec 形式保证 java 为 PID 1，能收到 SIGTERM
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar /app/erp-api.jar"]
