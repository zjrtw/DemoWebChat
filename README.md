# 🎓 Real-time Avatar Online Study Platform

실시간으로 여러 사용자가 함께 공부하는 플랫폼입니다. 웹캠/음성 기반 아바타 반응, 실시간 문서 공동 편집, 마우스 좌표 공유 및 필기 기능을 지원합니다.

## 🌟 핵심 기능

### 1. **3가지 아바타 모드**
- **풀 아바타**: 웹캠 눈 깜빡임 + 마이크 음성 분석
- **라이트 아바타**: 마이크 음성 분석 + 자동 눈 깜빡임
- **보이스 전용**: 프로필 이미지 + 음성 신호만

### 2. **실시간 발언자 표시**
- Audio Analyzer로 마이크 입력 RMS 분석
- 임계값(VITE_VOLUME_THRESHOLD=0.05) 초과 시 Glow 효과
- 동시 발언 지원

### 3. **공동 편집 & 협업**
- Y.js 기반 OT(Operational Transform) 동기화
- 실시간 마우스 좌표 공유 (레이저 포인터)
- Canvas 기반 필기 기능

### 4. **데이터 영구 저장**
- 채팅 메시지: PostgreSQL 저장
- 문서 변경 이력: Y.js Update 저장
- 페이지 새로고침/늦은 입장 시 복구

## 📁 프로젝트 구조

```
webChat/
├── docker-compose.yml  # PostgreSQL Docker 설정
├── docs/
│   ├── prd.md          # 제품 요구사항 문서
│   ├── SETUP.md        # 개발 환경 설정 가이드
│   └── tasks.md        # 작업 체크리스트
├── frontend/           # React 프론트엔드
│   ├── index.html
│   ├── src/
│   │   ├── store/      # Zustand 상태 관리
│   │   ├── components/ # React 컴포넌트
│   │   ├── hooks/      # 커스텀 훅
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   └── package.json
└── backend/            # Node.js 백엔드
    ├── src/
    │   ├── server.js           # Express + Socket.io 메인
    │   ├── websocket.js        # WebSocket 신호 중개
    │   ├── yjs-binding.js      # Y.js 공동 편집
    │   ├── audio-analyzer.js   # 음성 분석 (RMS)
    │   ├── database.js         # PostgreSQL 연결
    │   ├── migrations.js       # 마이그레이션 관리
    │   ├── db-utils.js         # 데이터베이스 유틸리티
    │   ├── migrations/         # SQL 마이그레이션 파일
    │   │   └── 001_initial_schema.sql
    │   ├── seeds/              # 샘플 데이터 스크립트
    │   │   └── 001_sample_data.js
    │   └── routes/             # API 라우트
    │       ├── auth.js         # 인증 API
    │       ├── users.js        # 사용자 API
    │       ├── rooms.js        # 방 API
    │       ├── messages.js     # 메시지 API
    │       └── documents.js    # 문서 API
    ├── DATABASE.md             # 데이터베이스 가이드
    ├── API.md                  # API 문서
    ├── .env.example
    └── package.json
```

## 🚀 빠른 시작

### 0. **PostgreSQL 설정 (Docker)**

```bash
# 프로젝트 루트에서
docker-compose up -d

# 상태 확인
docker-compose ps

# 로그 확인
docker-compose logs postgres
```

### 1. **백엔드 설정**

```bash
cd backend
npm install

# .env 파일 생성
cp .env.example .env
```

`.env` 파일 내용:
```env
NODE_ENV=development
PORT=3001
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/webchat_dev
VOLUME_THRESHOLD=0.05
VOICE_ACTIVITY_TIMEOUT=1000
LOG_LEVEL=debug
```

```bash
# 데이터베이스 마이그레이션 (서버 시작 시 자동 실행)
npm run migrate

# 샘플 데이터 추가 (선택)
npm run seed

# 또는 한 번에
npm run db:reset

# 서버 실행 (마이그레이션 자동 포함)
npm run dev
```

### 2. **프론트엔드 설정**

```bash
cd ../frontend
npm install

# .env 파일 생성
cp .env.example .env

# 개발 서버 실행
npm run dev
```

브라우저에서 `http://localhost:5173` 자동 접속

## 🔧 환경 변수

### 백엔드 (.env)
- `NODE_ENV`: development | production
- `PORT`: 서버 포트 (기본값: 3001)
- `DATABASE_URL`: PostgreSQL 연결 문자열 (예: `postgresql://postgres:postgres@localhost:5432/webchat_dev`)
- `VOLUME_THRESHOLD`: 음성 감지 임계값 (기본값: 0.05, 범위: 0.01~0.1)
- `VOICE_ACTIVITY_TIMEOUT`: 음성 활동 타임아웃 (기본값: 1000ms)
- `LOG_LEVEL`: 로깅 레벨 (debug | info | warn | error)

### 프론트엔드 (.env)
- `VITE_API_URL`: 백엔드 서버 주소 (기본값: http://localhost:3001)
- `VITE_VOLUME_THRESHOLD`: 음성 감지 임계값 (기본값: 0.05)
- `VITE_BLINK_INTERVAL`: 자동 눈 깜빡임 간격 (기본값: 4500ms)

## 📡 웹소켓 이벤트

### 클라이언트 → 서버
- `room:join`: 방 입장
- `message:send`: 채팅 메시지 전송
- `audio:volume`: 마이크 음량 데이터
- `editor:cursor`: 마우스 좌표
- `editor:draw`: 필기 좌표
- `y:update`: Y.js 문서 업데이트 (공동 편집)
- `y:sync`: Y.js 동기화 프로토콜
- `y:awareness`: 사용자 인식 정보

### 서버 → 클라이언트
- `room:joined`: 방 입장 완료
- `room:participants`: 현재 참여자 목록
- `user:joined`: 새 사용자 입장
- `user:left`: 사용자 퇴장
- `user:speaking`: 발언 상태 변경
- `message:received`: 채팅 메시지 수신
- `editor:cursor`: 원격 사용자 마우스 좌표
- `editor:draw`: 원격 사용자 필기
- `y:update`: 원격 Y.js 업데이트

## 📄 공동 편집 (Y.js) 아키텍처

```
클라이언트 1              클라이언트 2
    ↓                       ↓
useYjsBinding          useYjsBinding
    ↓                       ↓
Y.Text (로컬)          Y.Text (로컬)
    ↓                       ↓
socket.emit('y:update')
    ↓
백엔드 WebSocket
    ↓
Y.Doc (서버 메모리)
    ↓
DB에 저장 (document_updates)
    ↓
socket.broadcast('y:update')
    ↓
다른 클라이언트들에게 전달
    ↓
Y.applyUpdate()
    ↓
UI 자동 동기화 ✨
```

### 문서 복구 프로세스
```
사용자가 페이지 새로고침 또는 늦은 입장
    ↓
useYjsBinding 훅 실행
    ↓
initializeDocument() 호출
    ↓
GET /api/documents/:roomId
    ↓
백엔드에서 document_updates 테이블 조회
    ↓
모든 Y.js 업데이트 반환
    ↓
클라이언트에서 Y.applyUpdate() 적용
    ↓
문서 완전히 복구됨 ✅
```

## 🎯 현재 구현 상태

### ✅ 완료
- ✅ 프로젝트 구조 및 초기 설정 (Docker 포함)
- ✅ Zustand 상태 관리 (전체 상태)
- ✅ WebSocket 기본 연결 (Socket.io)
- ✅ Audio Analyzer (RMS 기반 음성 감지)
- ✅ 발언자 표시 (Glow 효과)
- ✅ 기본 UI 레이아웃 (좌:우 = 6:4)
- ✅ Canvas 기반 필기 기능 (로컬)
- ✅ **PostgreSQL 스키마 설계**
- ✅ **마이그레이션 시스템**
- ✅ **샘플 데이터 생성 (Seeding)**
- ✅ **데이터베이스 유틸리티**
- ✅ **Y.js 공동 편집 통합** (클라이언트 + 서버)
- ✅ **문서 복구 기능** (API + 클라이언트)
- ✅ **전체 REST API** (인증, 사용자, 방, 메시지, 문서)
- ✅ **API 문서** (자세한 명세서)

### 🔄 진행 중 / 예정
- [ ] PostgreSQL 데이터 저장 (채팅 메시지 DB 저장 로직)
- [ ] 마우스 레이저 포인터 (서버 중개 테스트)
- [ ] 풀 아바타 모드 (웹캠 분석, face-api.js)
- [ ] ChatPanel 컴포넌트
- [ ] 사용자 인증 (JWT 토큰)
- [ ] 에러 핸들링 및 로깅 개선
- [ ] 성능 최적화 및 테스트

## 🔐 보안 고려사항

- [ ] WebSocket 인증 (JWT 토큰)
- [ ] 마이크/웹캠 권한 명시적 요청
- [ ] 채팅 메시지 XSS 필터링
- [ ] CORS 정책 설정
- [ ] Rate limiting
- [ ] 민감한 환경변수 .env 미포함

## 📊 데이터 흐름

```
사용자 입력 (음성/마우스)
    ↓
useAudioAnalyzer / EditorPanel
    ↓
Socket.io emit (audio:volume, editor:cursor)
    ↓
백엔드 WebSocket Handler
    ↓
Audio Analyzer 분석 → user:speaking 브로드캐스트
마이크 → DB 저장
    ↓
다른 사용자들에게 broadcast
    ↓
AvatarGrid / EditorPanel 업데이트
    ↓
UI 렌더링 (Glow, 입 움직임 등)
```

## 🛠️ 개발 팁

### 로컬 테스트 (2명 이상)
```bash
# 터미널 1: PostgreSQL (Docker)
docker-compose up -d
docker-compose logs -f postgres  # 상태 모니터링

# 터미널 2: 백엔드
cd backend && npm run dev

# 터미널 3: 프론트엔드
cd frontend && npm run dev

# 터미널 4: 추가 탭/브라우저
# Chrome, Firefox 등에서 http://localhost:5173 동시 열기
```

### Docker 자주 쓰는 명령어
```bash
# PostgreSQL 시작/중지
docker-compose up -d       # 시작
docker-compose down        # 중지 (데이터 유지)
docker-compose down -v     # 중지 및 초기화 (데이터 삭제)

# 상태 확인
docker-compose ps
docker-compose logs -f postgres

# PostgreSQL 직접 접속
docker-compose exec postgres psql -U postgres -d webchat_dev
```

### WebSocket 디버깅
- Chrome DevTools → Network → WS 필터
- Socket.io 이벤트 확인

### 음성 임계값 조정
- `.env`에서 `VOLUME_THRESHOLD` 수정 (0.01~0.1)
- 낮을수록 민감, 높을수록 둔감

## 📚 참고 자료

### 프로젝트 문서
- [개발 환경 설정](docs/SETUP.md)
- [데이터베이스 가이드](backend/DATABASE.md)
- [API 문서](backend/API.md)
- [작업 체크리스트](docs/tasks.md)
- [제품 요구사항](docs/prd.md)

### 외부 문서
- [Y.js Documentation](https://docs.yjs.dev/)
- [Socket.io Documentation](https://socket.io/docs/)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

## 📝 라이선스

MIT

## 👨‍💻 작성자

Claude Code - 초기 구조 및 핵심 로직 구현
# DemoWebChat
