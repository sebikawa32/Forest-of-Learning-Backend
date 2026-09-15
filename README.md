# 🌳 Forest of Learning Backend

혼자 공부하는 사용자가 **스터디 · 습관 · 집중 · 포인트**를 기록하는 서비스의 API 서버입니다.

👉 [Frontend](https://forest-of-learning-frontend.vercel.app/) · Backend: Render · DB: PostgreSQL

---

## Tech Stack

Node.js · Express 5 · Prisma · PostgreSQL · express-session · argon2 · express-rate-limit

<p>
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white"/>
  <img src="https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white"/>
  <img src="https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white"/>
</p>

---

## My Role — 원세빈 (Detail)

스터디 **상세 페이지**에 필요한 API를 맡았습니다. 한 화면에서 스터디 정보, 응원 이모지, 주간 습관 기록이 보이도록 응답을 맞췄습니다.

- **응원 이모지** `POST/GET /emojis`  
  클릭마다 row를 만들지 않고 `(studyId, emoji)` unique 기준으로 **upsert + count increment**. 조회는 count 내림차순.
- **주간 습관 기록** `GET /habits/:studyId/records`  
  `habitId` 단건 조회를 **studyId + weekStart/weekEnd**로 바꿔, 프론트가 바로 그릴 수 있는 날짜 맵으로 반환.

```js
{ habitId, habitName, dates: { "2026-04-14": true, "2026-04-15": false } }
```

---

## 주요 기능

| 도메인 | 내용 |
| --- | --- |
| Study | CRUD, 검색·정렬·페이지네이션, argon2 해싱, 세션 비밀번호 인증 |
| Habit | CRUD, 날짜별 체크 upsert, 주간 기록 조회 |
| Focus | 일시정지 제외 실집중 시간 계산, 포인트 적립을 트랜잭션으로 처리 |
| Point | 스터디별 누적 포인트 + PointLog |
| Emoji | 스터디별 응원 이모지 (동일 이모지는 count 증가) |

---

## 트러블슈팅

### 1. 같은 이모지를 누를 때마다 행이 늘어남

응원은 “🔥 3”처럼 **종류별 카운트**인데, 처음엔 `create()`라서 클릭 1회 = 레코드 1개였습니다.

`(studyId, emoji)`에 `@@unique`를 걸고 `upsert`로 count만 올리도록 바꿨습니다.  
저장 단위와 화면 집계 단위를 맞춰야 프론트 가공과 중복 데이터를 줄일 수 있었습니다.

### 2. 습관 기록이 상세 주간 그리드에 안 맞음

기존 API는 `GET /habits/:habitId/records`라서, 그리드를 만들려면 습관 수만큼 요청이 필요했습니다.  
쓰기는 habitId, 읽기는 **스터디 + 주간**이 맞아서 조회 단위를 분리했습니다.

`weekStart`/`weekEnd`로 기간을 자르고 `{ habitId, habitName, dates }`로 내려 N+1과 프론트 가공을 줄였습니다.

### 3. 배포 후 세션 인증이 풀림 (팀)

로컬에선 됐지만 Vercel + Render에선 다음 요청이 401이었습니다. 도메인이 달라 쿠키가 안 붙고, HTTPS에서 `SameSite=Lax`면 cross-site 쿠키가 막혔습니다.

CORS `credentials`, 프로덕션 `sameSite: 'none'` / `secure` / `trust proxy`, 인증 후 `session.save()`로 맞췄습니다.  
인증은 백엔드 로직만이 아니라 CORS·쿠키·프론트 fetch가 한 세트라는 걸 배웠습니다.

---

## Team

| GitHub | 담당 |
| --- | --- |
| [@juengseulki](https://github.com/juengseulki) | PM, ERD, 포인트 로직, 번역 API |
| [@DevGangMin](https://github.com/DevGangMin) | Prisma, 입력 검증, 세션 인증, rate limit |
| [@sojeong0302](https://github.com/sojeong0302) | 스터디 CRUD, 검색·정렬·페이지네이션 |
| **원세빈** ([@sebikawa32](https://github.com/sebikawa32)) | **상세 페이지 데이터, 응원 이모지, 주간 습관 기록** |
| [@Crong-dev](https://github.com/Crong-dev) | 습관 CRUD, 체크/해제 |
| [@karrum5692](https://github.com/karrum5692) | 집중 세션 흐름 |

---

## 실행

```bash
npm install
# .env: DATABASE_URL, SESSION_SECRET, ALLOWED_ORIGINS, API_BASE_URL
npx prisma migrate dev
npm run seed
npm run dev
```
