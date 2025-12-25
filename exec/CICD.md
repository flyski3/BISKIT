# 1. Gitlab 소스 클론 이후 빌드 및 배포

## 1.1 사용 기술 및 버전 정보

### **A. Backend (Spring Boot)**

#### **JVM 및 빌드 도구**
- **Java**: OpenJDK 21
- **Spring Boot**: 3.5.5
- **Spring Dependency Management**: 1.1.7
- **Gradle**: 8.14.3
- **IDE**: IntelliJ IDEA 권장

#### **주요 라이브러리 버전**
```gradle
// Spring Framework
spring-boot-starter-data-jpa: 3.5.5
spring-boot-starter-data-redis: 3.5.5
spring-boot-starter-oauth2-client: 3.5.5
spring-boot-starter-security: 3.5.5
spring-boot-starter-validation: 3.5.5
spring-boot-starter-web: 3.5.5
spring-boot-starter-webflux: 3.5.5
spring-boot-starter-websocket: 3.5.5
spring-messaging: 3.5.5

// 보안
jjwt-api: 0.12.6
jjwt-impl: 0.12.6
jjwt-jackson: 0.12.6

// 개발 도구
spring-boot-devtools: 3.5.5

// 데이터베이스
mysql-connector-j: 9.4.0
h2database: runtime (테스트용)
```

---

### **B. Frontend (Next.js)**

#### **개발 환경**
- **next.js**: node:18-alpine

#### **SDK 버전**
- **compileSdk**: 36
- **minSdk**: 33 (Android 13)
- **targetSdk**: 36

#### **주요 라이브러리/도구(참고)**
- typescript
- jenkins
- docker
- nginx
- react query

---

### **C. AI Server (FastAPI)**

#### **Python 환경**
- **Python**: 3.10
- **FastAPI**: 0.115.4

#### **주요 라이브러리**
```python
# AI/ML
pytorch >= 2.5.1
numpy >= 2.2.6
pandas >= 2.3.2
graphSAGE
GNN explainer
```

---

## 1.2 빌드 및 실행 환경 변수

> ⚠️ 보안 주의  
> 아래 항목 중 **키/토큰/비밀번호/시크릿 값은 절대 Git에 커밋하지 마세요.**  
> repo에는 `.env.example`만 커밋하고, 실제 값은 `.env`에 넣고 `.gitignore` 처리하세요.

### (권장) `.gitignore`
```gitignore
.env
```

---

### **Frontend/Backend 환경 변수**

#### **필수 환경 변수**
> 아래는 예시이며, 실제 값은 `.env`에 넣어 사용하세요.

```yaml
# application.properties 또는 application.yml에 대응되는 값(예시)
spring:
  datasource:
    spring.datasource.username: ${SPRING_DATASOURCE_USERNAME}
    spring.datasource.password: ${SPRING_DATASOURCE_PASSWORD}

jwt:
  jwt.secret: ${JWT_SECRET}
```

#### **JWT_SECRET 설명(중요)**
- `JWT_SECRET`은 **Google OAuth와 무관**합니다.
- 서버가 JWT 토큰을 **서명/검증**하는 데 사용하는 비밀키로, **프로그램 실행자(운영자)가 임의로 생성**해서 넣어야 합니다.
- 가능한 한 **길고 랜덤한 문자열**을 권장합니다.

#### **AI API 연동 설정**
```yaml
python:
  api:
    ai.server.base-url: ${AI_SERVER_BASE_URL}  # AI 서버 주소
```

#### **Kakao Maps API 키**
- `KAKAO_MAP_KEY=${KAKAO_MAP_KEY}`

#### **GeoCoder API 키**
- `GEOCODER_TOKEN=${GEOCODER_TOKEN}`

---

### **AI Server 환경 변수**

#### **.env 파일 생성 필요**
> 직접 빌드 시 서버 루트 디렉토리에 `.env` 파일을 생성합니다.  
> ✅ repo에는 **`.env.example`만 커밋**하고 실제 `.env`는 커밋하지 않습니다.

#### `.env.example` (커밋 O / 실제 값은 비워두거나 플레이스홀더로 유지)
```env
# === 기본 설정 ===
SPRING_PROFILES_ACTIVE=dev
APP_PORT=8080

# === 데이터베이스 설정 ===
# [CHANGE] 아래 4개는 반드시 채우세요 (도커 mysql 컨테이너 기동/접속에 필요)
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=

# [CHANGE] 보통 MYSQL_USER / MYSQL_PASSWORD와 동일하게 맞춥니다.
SPRING_DATASOURCE_USERNAME=
SPRING_DATASOURCE_PASSWORD=
# [KEEP] docker-compose에서 mysql 서비스명(mysql)로 접근하는 형태는 유지
SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/${MYSQL_DATABASE}?serverTimezone=Asia/Seoul&characterEncoding=UTF-8

# === Redis 설정 ===
# [KEEP] docker-compose에서 redis 서비스명(redis)로 접근
SPRING_DATA_REDIS_HOST=redis
# [CHANGE] compose에서 redis도 이 값을 참조하므로 변경 시 함께 변경 필요
REDIS_PORT=6379
SPRING_DATA_REDIS_PASSWORD=
SPRING_DATA_REDIS_TIMEOUT=2000

# === JWT 설정 ===
# [CHANGE] 반드시 채우세요 (긴 랜덤 문자열 권장)
JWT_SECRET=

# === OAuth 설정 ===
# [CHANGE] 실제 값 입력 (Secret은 절대 커밋 금지)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
# [KEEP] 로컬 기준 (운영 배포 시 도메인에 맞게 변경)
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/callback

# === 외부 API 설정 ===
# [CHANGE] 실제 값 입력
KT_API_KEY=
PUBLIC_DATA_API_KEY=
KAKAO_MAP_KEY=
GEOCODER_TOKEN=

# === AI 서버 연동 ===
# [KEEP] docker-compose에서 ai 서비스명(ai) + 포트 8000
AI_SERVER_BASE_URL=http://ai:8000

# === AI GMS 설정 (새로 추가) ===
# [CHANGE] GMS 연동을 사용할 경우 반드시 설정
#         (아래 AI_GMS_* 및 GMS_*는 서로 연관된 "GMS 연동" 설정입니다)
AI_GMS_API_KEY=
AI_GMS_BASE_URL=

# === GMS 설정 ===
# [CHANGE] 위 AI_GMS_* 와 동일하게 GMS 연동에 사용됩니다(프로젝트 구성에 따라 둘 다/하나만 사용)
GMS_KEY=
GMS_BASE_URL=

# 데이터/모델 파일이 있는 폴더 (파일이 위치한 곳)
DATA_DIR=./data
MODEL_PATH=survival_gnn.pt
META_PATH=survival_meta.json

# === 로깅 ===
LOGGING_LEVEL_ROOT=INFO
LOGGING_LEVEL_APP=DEBUG
LOG_LEVEL=INFO

# === LLM 옵션(사용 시) ===
LLM_ENABLE=true
LLM_MODEL=gpt-5-nano
LLM_TIMEOUT=60
```

---

## 1.3 빌드 및 배포

### **빌드 아키텍처**
- git clone 이후 `.env` 파일을 루트에 생성 (`.env.example`을 복사해 생성)
- `docker compose up --build`를 통해 실행
- `http://localhost:3000` 으로 접근

### **배포 아키텍처**
1. **개발자가 release 브랜치에 push**
2. 젠킨스가 git clone 워크스페이스에 복사
3. 젠킨스 파일을 읽고 credential 환경변수에 주입
4. 도커 컴포즈 빌드
5. 로드 실패 시 과거 성공버전으로 배포

---

## 1.4 배포시 특이사항

> nginx 컨테이너 / jenkins 컨테이너를 EC2에서 직접 실행하는 구성(예시)

### nginx 실행 예시
```bash
docker run \
  -d \
  --name nginx \
  --network my-network \
  --restart unless-stopped \
  -p 80:80 \
  -p 443:443 \
  -v /home/ubuntu/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /etc/letsencrypt:/etc/letsencrypt:ro \
  nginx:latest
```

### jenkins 실행 예시
```bash
docker run \
  -d \
  --name jenkins \
  --network my-network \
  --restart unless-stopped \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e JENKINS_OPTS="--prefix=/jenkins" \
  --group-add $(stat -c '%g' /var/run/docker.sock) \
  my-jenkins:latest
```

---

## 1.5 주요 설정 파일 목록

### **Backend**
- `backend/src/main/resources/application.properties` - 메인 설정
- `backend/build.gradle` - 의존성 및 빌드 설정

### **Frontend**
- `frontend/yarn.lock` - 라이브러리 버전 관리
- `frontend/package.json` - 앱 빌드 설정

### **Env**
- `.env.example` (커밋 O)
- `.env` (커밋 X)
