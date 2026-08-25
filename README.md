# VM 연결 테스트 앱

DB 없이 실행되는 최소 Spring Boot + Thymeleaf 애플리케이션입니다. Java 17 이상이 필요합니다.

## 빌드 및 실행

Windows:

```powershell
.\gradlew.bat clean bootJar
java -jar build\libs\vm-test-app-0.0.1-SNAPSHOT.jar
```

Linux VM:

```bash
chmod +x gradlew
./gradlew clean bootJar
java -jar build/libs/vm-test-app-0.0.1-SNAPSHOT.jar
```

백그라운드 실행:

```bash
nohup java -jar build/libs/vm-test-app-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

## 접속 확인

- 화면: `http://VM_PUBLIC_IP:8080/`
- API: `http://VM_PUBLIC_IP:8080/health`
- VM 내부 확인: `curl http://localhost:8080/health`

외부에서 접속하려면 클라우드 방화벽/보안 목록과 VM 방화벽에서 TCP 8080 포트를 허용해야 합니다.
