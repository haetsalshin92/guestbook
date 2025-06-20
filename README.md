✅ 1. ./mvnw는 무엇인가?
./mvnw는 Maven Wrapper 실행 스크립트예요.

즉, Maven이 젠킨스 서버에 설치되어 있지 않아도, 프로젝트 내에 Maven Wrapper가 있으면 해당 버전의 Maven을 자동으로 내려받아 사용하게 해줘요.

Git 프로젝트에 ./mvnw, mvnw.cmd, .mvn/ 폴더가 있으면 Maven Wrapper가 설정된 프로젝트예요.

✅ 2. clean compile은 무엇을 의미하나?
clean: target/ 디렉토리를 지워서 이전 빌드 결과를 없앰.

compile: src/main/java 안의 소스 파일들을 .class 파일로 컴파일만 함 (JAR로 패키징은 안 함).

✅ 3. 어디서 실행되고, 뭘 빌드하나?
Jenkins 기준 흐름:
git clone → 프로젝트가 워크스페이스에 복사됨 (예: /home/ec2-user/.jenkins/workspace/guestbook)

Jenkins는 이 워크스페이스 디렉토리에서 ./mvnw clean compile 실행

해당 디렉토리의 pom.xml 파일을 기준으로 빌드 시작

🔹 즉, ./mvnw는 현재 디렉토리(보통은 checkout 받은 Git 루트)에 있는 pom.xml을 읽고,
src/main/java의 자바 소스들을 컴파일하여 target/classes에 .class 파일로 변환합니다.

✅ Maven이 없으면 안 되는가?
./mvnw를 쓰면 Maven이 로컬에 설치되어 있지 않아도 괜찮음

단, mvnw가 프로젝트에 포함되어 있어야 함

만약 ./mvnw가 없다면, 시스템에 Maven이 설치되어 있어야 하고 mvn clean compile처럼 사용해야 해요.

