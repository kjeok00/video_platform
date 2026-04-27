영상 스트리밍 기반 아동 컨텐츠 플랫폼  

# 1. 프로젝트 링크

---
- 배포 주소
  - https://bootale.net/


# 2. 기술 스택

---
- Backend
  - Django
  - drf-spectacular
- Authentication
  - JWT
  - SMTP
- Database
  - MySQL
- Media
  - FFmpeg
  - HLS
- Realtime & Async
  - Daphne
  - Channels
  - WebSocket
- Task Queue / Cache
  - Celery
  - Redis
- DevOps
  - Docker
  - Watchdog
  - Gitlab CI
  - Nginx

---
# 3. 환경 개요

## 디렉토리 구조
~~~
/home/infant(root)
|
|-- infra_env
|   |-- docker-compose.yml
|   |-- nginx
|       |-- conf
|
|-- dev_env
|   |-- docker-compose.yml
|   |-- .env
|   |-- django
|   |   |-- Dockerfile
|   |   |-- .gitlab-ci.yml
|   |
|   |-- react
|   |-- volumes
|       |-- media
|       |-- logs
|
|-- qa_env
|   |-- ...
|
|-- prod_env
|   |-- ...
|
|-- Makefile    
~~~

## Container 및 요청 구조
~~~
client
↓
↓
|-- booktale.net / booktle.kr / booktale.co.kr
↓
↓
nginx
|
↓
↓
booktale.net
|
|-- /dev <dev_booktale_net>
|   |
|   |-- dev_django
|   |-- dev_mysql
|   |-- dev_celery
|   |-- dev_redis
|
|-- /qa <qa_booktale_net>
|   |
|   |-- qa_django
|   |-- qa_mysql
|   |-- qa_celery
|   |-- qa_redis
|
|-- / (운영) <prod_booktale_net>
    |
    |-- prod_django
    |-- prod_mysql
    |-- prod_celery
    |-- prod_redis
~~~

# 4. 환경 구성 가이드

1. docker-compose.yml 생성
2. 프로젝트 파일 clone
   1. 프로젝트 파일 내부가 아닌 같은 디렉토리에 .env 파일 생성
      1. 프로젝트 settings에 상위 디렉토리의 .env 파일을 참조하도록 설정되어 있음
3. docker-compose와 같은 디렉토리에서 docker compose build (--no-cache)
4. docker compose up -d
5. docker ps로 컨테이너 실행 확인

- 한번에, root에서 qa, dev, prod 환경의 docker-compose에 명령 내리기
  1. Makefile이 있는 root 디렉토리로 이동
  2. make build-all, make up-all, make down-all
  3. make [build or up or down]-[qa or dev or prod]

## 5. HLS 구성
- 인코딩: FFmpeg로 240p~1080p 다중 해상도 HLS(m3u8) 생성.
- 서빙
  - /media/episode_xx/master.m3u8 경로로 접근하여 파일 전달
  - /media/episode_xx/master.m3u8 경로로 접근, JWT 인증 후 Nginx가 X-Accel-Redirect로 파일 전달.  
- 보안: 인증된 사용자만 미디어 접근 가능

# 6. 인증 및 보안
- JWT 인증: Django REST Framework와 djangorestframework-simplejwt로 구현
- 이메일 인증: SMTP를 통해 회원가입 및 비밀번호 재설정 이메일 전송. 
- 추가 보안
  - HTTPS 강제
  - CSRF/XSS 방지 설정
  - IP 화이트 리스트 설정
    - dev, qa 환경은 **220.124.223.3** 에서만 접근 가능
- 미디어 보호: X-Accel-Redirect로 Nginx가 인증된 요청만 처리.  


# 7. API 문서
- Swagger/OpenAPI: drf-spectacular로 자동 생성된 API 문서 제공.  
- 주요 엔드포인트:
  - POST /api/auth/register/: 사용자 등록
  - POST /api/auth/login/: JWT 토큰 발급
  - GET /api/contents/: 콘텐츠 목록 조회
  - POST /api/transactions/: 결제 처리

# 8. 이메일 인증
- SMTP 설정: Gmail SMTP 이용
- 환경변수 (.env)
~~~
# Notion 참조
EMAIL_HOST_USER=
EMAIL_HOST_PASSWORD=
EMAIL_HOST=
EMAIL_BACKEND=
EMAIL_PORT=
~~~

# 9. 패키지 설치
~~~
cd platform-infant-backend
pip install -r requirements.txt
~~~

# 10. 기타
- 파일 감지: watchdog으로 미디어 파일 생성/삭제 감지  
- 영상 처리: moviepy, ffmpeg-python으로 동영상 편집 및 변환  
- 캐싱: django-redis로 쿼리 캐싱 최적화
