# 영유아플랫폼
> 영상 스트리밍 기반 아동 프리미엄 콘텐츠 플랫폼

![Django](https://img.shields.io/badge/Django-4.2.20-092E20?style=flat-square&logo=django)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql)
![Celery](https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker)

## 📖 프로젝트 소개
**영유아용 교육 및 엔터테인먼트 콘텐츠를 제공하는 미디어 플랫폼입니다.
정기 구독 및 포인트 결제 시스템을 기반으로 영상 콘텐츠를 서비스하며, FFmpeg와 HLS를 활용하여 안정적인 스트리밍 환경을 제공합니다.
Django Channels와 Celery를 도입하여 실시간 알림과 대용량 미디어 파일의 백그라운드 비동기 처리를 지원합니다.

---

## 🛠 1. 기술 스택
### Backend Framework & Language
- **Django**, **drf-spectacular**

### Database & Cache
- **MySQL**
- **Redis**

### Real-time & Async Processing
- **Celery**, **Redis**
- **Django Channels**, **Daphne**, **WebSocket**

### Media & Authentication
- **FFmpeg**, **HLS**, **moviepy**, **ffmpeg-python** 
- **JWT** (SimpleJWT), **SMTP** 

### DevOps & Infrastructure
- **Docker**, **Docker Compose**
- **Nginx**, **Watchdog** 
- **Gitlab CI**

---

## ⚙️ 2. 환경 개요

### 디렉토리 구조
본 프로젝트는 Dev, QA, Prod 환경을 각각 독립적으로 구성하여 운영됩니다.
```text
/home/ (Root)
 ├── infra_env/        # 공통 인프라
 │    ├── docker-compose.yml
 │    └── nginx/conf
 ├── dev_env/          # 개발 환경
 │    ├── docker-compose.yml
 │    ├── .env
 │    ├── django/ (Dockerfile, .gitlab-ci.yml)
 │    ├── react/
 │    └── volumes/ (media, logs)
 ├── qa_env/           # QA 환경
 ├── prod_env/         # 운영 환경
 └── Makefile          # 전체 환경 통합 빌드/실행 스크립트
```

### Container 및 요청 구조
클라이언트 요청은 Nginx를 거쳐 각 환경(dev, qa, prod)의 Django, MySQL, Celery, Redis 컨테이너로 라우팅됩니다.
```text
client
 ↓
domain
 ↓
nginx
 ↓
 ├── /dev 
 │    ├── dev_django, dev_mysql, dev_celery, dev_redis
 ├── /qa
 │    ├── qa_django, qa_mysql, qa_celery, qa_redis
 └── /prod
      └── prod_django, prod_mysql, prod_celery, prod_redis
```

---

## 🚀 3. 환경 구성 가이드
1. `docker-compose.yml` 생성
2. 프로젝트 파일 clone
   - 프로젝트 파일 내부가 아닌 같은 디렉토리에 `.env` 파일 생성
3. `docker-compose.yml`이 있는 디렉토리에서 `docker compose build (--no-cache)` 실행
4. `docker compose up -d`
5. `docker ps`로 컨테이너 정상 실행 확인

**💡 통합 환경 관리 (Makefile 활용):**
- Makefile이 있는 root 디렉토리에서 각 환경을 일괄 제어 가능
  - `make build-[qa|dev|prod]`
  - `make up-[qa|dev|prod]`
  - `make down-[qa|dev|prod]`
  - 전체 제어: `make build-all`, `make up-all`, `make down-all`

---

## 🎬 4. HLS 구성 및 미디어 처리
- **인코딩:** FFmpeg를 활용하여 240p~1080p 다중 해상도 HLS(m3u8) 포맷으로 변환.
- **안전한 서빙:** 
  - `/media/episode_xx/master.m3u8` 경로로 접근
  - JWT 인증을 거친 후 **Nginx `X-Accel-Redirect`**를 통해서만 실제 파일 전달.
- **보안:** 인증된 사용자만 프리미엄 미디어 리소스에 접근 가능.

---

## 🔒 5. 인증 및 보안
- **JWT 인증:** Django REST Framework와 `djangorestframework-simplejwt` 활용.
- **이메일 인증:** SMTP를 통해 회원가입 및 비밀번호 재설정 이메일 전송.
- **추가 보안 정책:**
  - HTTPS 통신 강제
  - CSRF/XSS 방지 설정 적용
  - IP 화이트리스트 설정: Dev, QA 환경은 특정 IP에서만 접근 가능하도록 제한.

---

## 📚 6. API 문서 및 주요 엔드포인트
- **Swagger/OpenAPI:** `drf-spectacular`를 통해 자동 생성.
- **주요 API 엔드포인트:**
  - `POST /register/` : 사용자 등록
  - `POST /login/` : JWT 토큰 발급
  - `GET  /contents/` : 콘텐츠 목록 조회
  - `POST /transactions/` : 구독 및 포인트 결제 처리

---

## 📧 7. 이메일 인증 (환경 변수)
Gmail SMTP 서버를 활용합니다. 
```properties
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
EMAIL_HOST=smtp.gmail.com
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_PORT=587
```

---

## 📦 8. 패키지 수동 설치
로컬 개발 시, 아래 명령어로 의존성을 설치합니다.
```bash
$ cd ~
$ pip install -r requirements.txt
```

---

## 💡 9. 기타 아키텍처 특징
- **파일 감지:** `watchdog` 라이브러리를 활용하여 미디어 파일의 생성 및 삭제 이벤트를 실시간 감지.
- **영상 처리:** `moviepy`, `ffmpeg-python`을 활용한 백그라운드 동영상 편집 및 인코딩.
- **캐싱 최적화:** `django-redis`를 도입하여 빈번한 DB 쿼리 부하 최소화.
