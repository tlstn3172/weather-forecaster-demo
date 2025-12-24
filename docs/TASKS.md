# Weather Forecaster 개발 작업 목록

## Phase 1: 프로젝트 초기 설정 (Week 1)

### 1.1 개발 환경 구축
- [ ] Next.js 14 프로젝트 초기화
  - [ ] `npx create-next-app@latest weather-forecaster-demo` 실행
  - [ ] TypeScript 선택
  - [ ] Tailwind CSS 선택
  - [ ] App Router 선택
  - [ ] ESLint 선택
- [ ] 프로젝트 구조 생성
  - [ ] `app/` 폴더 확인
  - [ ] `components/ui/` 폴더 생성
  - [ ] `components/weather/` 폴더 생성
  - [ ] `components/layout/` 폴더 생성
  - [ ] `components/map/` 폴더 생성
  - [ ] `lib/api/` 폴더 생성
  - [ ] `lib/hooks/` 폴더 생성
  - [ ] `lib/utils/` 폴더 생성
  - [ ] `lib/types/` 폴더 생성
  - [ ] `lib/interfaces/` 폴더 생성
  - [ ] `lib/cache/` 폴더 생성
  - [ ] `lib/transformers/` 폴더 생성
  - [ ] `__tests__/unit/` 폴더 생성
- [ ] 테스팅 환경 설정 (코어 로직만)
  - [ ] Jest 설치: `npm install -D jest @types/jest`
  - [ ] ts-jest 설치: `npm install -D ts-jest`
  - [ ] `jest.config.js` 생성 및 설정
  - [ ] `jest.setup.js` 생성
  - [ ] MSW 설치: `npm install -D msw`
  - [ ] MSW 핸들러 설정 파일 생성
  - [ ] package.json에 테스트 스크립트 추가
- [ ] Prettier 설정
  - [ ] Prettier 설치: `npm install -D prettier`
  - [ ] `.prettierrc` 파일 생성
  - [ ] `.prettierignore` 파일 생성
  - [ ] ESLint-Prettier 통합: `npm install -D eslint-config-prettier`
- [ ] Git 설정
  - [ ] `.gitignore` 확인 및 업데이트 (.env.local, .next, out 등)
  - [ ] 초기 커밋: `git add .` && `git commit -m "chore: Initial Next.js setup"`

### 1.2 기본 설정 파일 작성
- [ ] `next.config.js` 작성
  - [ ] Static export 설정
  - [ ] basePath 설정
  - [ ] 이미지 최적화 설정
- [ ] `tailwind.config.ts` 작성
  - [ ] 커스텀 색상 정의
  - [ ] 커스텀 폰트 설정
  - [ ] 커스텀 border-radius 설정
- [ ] `tsconfig.json` 최적화
  - [ ] Path alias 설정 (`@/`)
- [ ] 환경 변수 설정
  - [ ] `.env.local.example` 생성
  - [ ] API 키 변수 정의

### 1.3 디자인 시스템 기초
- [ ] `styles/globals.css` 작성
  - [ ] Tailwind 임포트
  - [ ] 커스텀 유틸리티 클래스 정의
  - [ ] 스크롤바 숨김 클래스
  - [ ] 온도 그라데이션 클래스
- [ ] Google Fonts (Inter) 설정
- [ ] Material Symbols 아이콘 설정

---

## Phase 2: 타입 시스템 및 유틸리티 (Week 1-2)

### 2.1 TypeScript 타입 정의 (TDD)
- [ ] `lib/types/weather.ts` 작성
  - [ ] `Location` 인터페이스
  - [ ] `CurrentWeather` 인터페이스
  - [ ] `HourlyForecast` 인터페이스
  - [ ] `DailyForecast` 인터페이스
  - [ ] `WeatherDetails` 인터페이스
  - [ ] `WeatherResponse` 인터페이스
- [ ] `lib/types/utils.ts` 작성
  - [ ] `WeatherCondition` 타입
  - [ ] `TemperatureUnit` 타입
  - [ ] `WindDirection` 타입
  - [ ] `ApiError` 인터페이스

### 2.2 유틸리티 함수 구현 (TDD 필수)
- [ ] `lib/utils/temperature.ts`
  - [ ] 테스트 작성: `celsiusToFahrenheit`
  - [ ] 구현: `celsiusToFahrenheit`
  - [ ] 테스트 작성: `fahrenheitToCelsius`
  - [ ] 구현: `fahrenheitToCelsius`
- [ ] `lib/utils/formatters.ts`
  - [ ] 테스트 작성: `formatTemperature`
  - [ ] 구현: `formatTemperature`
  - [ ] 테스트 작성: `formatDate`
  - [ ] 구현: `formatDate`
  - [ ] 테스트 작성: `formatTime`
  - [ ] 구현: `formatTime`
- [ ] `lib/utils/wind.ts`
  - [ ] 테스트 작성: `getWindDirection`
  - [ ] 구현: `getWindDirection`
  - [ ] 테스트 작성: `degreesToDirection`
  - [ ] 구현: `degreesToDirection`
- [ ] `lib/utils/weatherIcons.ts`
  - [ ] 테스트 작성: `getWeatherIcon`
  - [ ] 구현: `getWeatherIcon`
  - [ ] 테스트 작성: `mapConditionToIcon`
  - [ ] 구현: `mapConditionToIcon`
- [ ] `lib/utils/validators.ts`
  - [ ] 테스트 작성: `isValidTemperature`
  - [ ] 구현: `isValidTemperature`
  - [ ] 테스트 작성: `isValidCoordinates`
  - [ ] 구현: `isValidCoordinates`

---

## Phase 3: API 통합 (Week 2)

### 3.1 API 인터페이스 정의 (SOLID - DIP)
- [ ] `lib/interfaces/WeatherProvider.ts`
  - [ ] `WeatherProvider` 인터페이스 정의
  - [ ] `getCurrentWeather` 메서드 시그니처
  - [ ] `getForecast` 메서드 시그니처
- [ ] `lib/interfaces/CacheStorage.ts`
  - [ ] `CacheStorage` 인터페이스 정의
  - [ ] `get`, `set`, `clear` 메서드 시그니처

### 3.2 OpenWeatherMap API 클라이언트 (TDD 필수)
- [ ] `lib/api/openweathermap.ts`
  - [ ] 테스트 작성: API 호출 로직
  - [ ] 구현: `fetchCurrentWeather`
  - [ ] 테스트 작성: 응답 파싱
  - [ ] 구현: `fetchOneCallAPI`
  - [ ] 테스트 작성: 에러 핸들링
  - [ ] 구현: 에러 핸들링 로직
- [ ] `lib/api/geocoding.ts`
  - [ ] 테스트 작성: `searchCity`
  - [ ] 구현: `searchCity`
  - [ ] 테스트 작성: `reverseGeocode`
  - [ ] 구현: `reverseGeocode`

### 3.3 데이터 변환 레이어 (TDD 필수)
- [ ] `lib/transformers/weatherTransformer.ts`
  - [ ] 테스트 작성: OpenWeatherMap → 내부 타입 변환
  - [ ] 구현: `transformCurrentWeather`
  - [ ] 테스트 작성: 시간별 예보 데이터 변환
  - [ ] 구현: `transformHourlyForecast`
  - [ ] 테스트 작성: 일별 예보 데이터 변환
  - [ ] 구현: `transformDailyForecast`
  - [ ] 테스트 작성: 날씨 상세 정보 변환
  - [ ] 구현: `transformWeatherDetails`

### 3.4 Next.js API Routes
- [ ] `app/api/weather/route.ts`
  - [ ] GET 핸들러 구현
  - [ ] 환경 변수에서 API 키 로드
  - [ ] 에러 응답 처리
- [ ] `app/api/geocoding/route.ts`
  - [ ] GET 핸들러 구현
  - [ ] 도시 검색 프록시

---

## Phase 4: 캐싱 및 상태 관리 (Week 2-3)

### 4.1 캐싱 구현 (TDD 필수, SOLID - OCP, DIP)
- [ ] `lib/cache/LocalStorageCache.ts`
  - [ ] 테스트 작성: `get`, `set`, `clear`
  - [ ] 구현: `CacheStorage` 인터페이스 구현
  - [ ] 테스트 작성: TTL 만료 처리
  - [ ] 구현: TTL 로직
- [ ] `lib/cache/MemoryCache.ts`
  - [ ] 테스트 작성: 메모리 캐시 동작
  - [ ] 구현: `CacheStorage` 인터페이스 구현

### 4.2 커스텀 훅 (TDD 필수)
- [ ] `lib/hooks/useGeolocation.ts`
  - [ ] 테스트 작성: 위치 권한 요청
  - [ ] 구현: Geolocation API 사용
  - [ ] 테스트 작성: 에러 처리
  - [ ] 구현: 에러 상태 관리
- [ ] `lib/hooks/useWeather.ts`
  - [ ] 테스트 작성: SWR 통합
  - [ ] 구현: 날씨 데이터 페칭
  - [ ] 테스트 작성: 캐싱 동작
  - [ ] 구현: 캐시 통합
- [ ] `lib/hooks/useLocation.ts`
  - [ ] 테스트 작성: 위치 상태 관리
  - [ ] 구현: 위치 변경 로직

### 4.3 Context API 설정
- [ ] `lib/contexts/WeatherContext.tsx`
  - [ ] Context 생성
  - [ ] Provider 컴포넌트 작성
  - [ ] 커스텀 훅 `useWeatherContext` 작성

---

## Phase 5: UI 컴포넌트 개발 (Week 3-4)

### 5.1 기본 UI 컴포넌트 (수동 테스트)
- [ ] `components/ui/Card.tsx`
  - [ ] Props 인터페이스 정의 (children, className, variant)
  - [ ] 카드 컴포넌트 구현
  - [ ] 다크 테마 스타일 적용
  - [ ] 반투명 배경 및 backdrop-blur 효과
  - [ ] 브라우저에서 수동 테스트
- [ ] `components/ui/Button.tsx`
  - [ ] Props 인터페이스 정의 (onClick, children, variant, disabled)
  - [ ] 버튼 컴포넌트 구현
  - [ ] 호버 효과 및 트랜지션
  - [ ] 비활성화 상태 스타일
  - [ ] 브라우저에서 수동 테스트
- [ ] `components/ui/Icon.tsx`
  - [ ] Props 인터페이스 정의 (name, size, color, filled)
  - [ ] Material Symbols 래퍼 구현
  - [ ] 크기 variant (sm, md, lg, xl)
  - [ ] 색상 props 처리
  - [ ] 브라우저에서 수동 테스트

### 5.2 레이아웃 컴포넌트
- [ ] `components/layout/Container.tsx`
  - [ ] 최대 너비 제한 (448px)
  - [ ] 중앙 정렬
- [ ] `components/layout/Header.tsx`
  - [ ] 메뉴 버튼
  - [ ] 위치 표시
  - [ ] 설정 버튼
- [ ] `components/layout/TabBar.tsx`
  - [ ] 하단 네비게이션
  - [ ] 4개 탭 (Weather, Cities, Radar, Profile)
  - [ ] 활성 상태 표시

### 5.3 날씨 컴포넌트
- [ ] `components/weather/CurrentWeather.tsx`
  - [ ] 대형 온도 표시
  - [ ] 날씨 아이콘
  - [ ] 날씨 상태 텍스트
  - [ ] 최고/최저 온도
- [ ] `components/weather/HourlyForecast.tsx`
  - [ ] 수평 스크롤 컨테이너
  - [ ] 시간별 카드 컴포넌트
  - [ ] "Now" 강조 표시
- [ ] `components/weather/DailyForecast.tsx`
  - [ ] 10일 예보 리스트
  - [ ] 온도 범위 바
  - [ ] 강수 확률 표시
- [ ] `components/weather/WeatherDetails.tsx`
  - [ ] UV 지수 카드
  - [ ] 습도 카드
  - [ ] 바람 카드 (나침반 포함)
  - [ ] 가시거리 카드

### 5.4 지도 컴포넌트
- [ ] `components/map/PrecipitationMap.tsx`
  - [ ] Google Maps 통합
  - [ ] 다크 테마 스타일
  - [ ] 강수량 오버레이
  - [ ] "Open Map" 버튼

---

## Phase 6: 페이지 구현 (Week 4)

### 6.1 메인 페이지
- [ ] `app/layout.tsx`
  - [ ] 루트 레이아웃
  - [ ] 폰트 설정
  - [ ] SWR Config Provider
  - [ ] Weather Context Provider
- [ ] `app/page.tsx`
  - [ ] 현재 날씨 섹션
  - [ ] 시간별 예보 섹션
  - [ ] 10일 예보 섹션
  - [ ] 상세 정보 섹션
  - [ ] 강수량 지도 섹션

### 6.2 추가 페이지
- [ ] `app/cities/page.tsx`
  - [ ] 도시 검색 입력 필드 UI
  - [ ] 검색 결과 리스트 표시
  - [ ] 저장된 도시 목록 표시
  - [ ] 도시 추가 기능 (localStorage 저장)
  - [ ] 도시 삭제 기능 (localStorage 업데이트)
  - [ ] 도시 선택 시 메인 페이지로 이동
  - [ ] 빈 상태 UI (저장된 도시 없을 때)
- [ ] `app/radar/page.tsx`
  - [ ] 전체 화면 레이아웃
  - [ ] Google Maps 전체 화면 통합
  - [ ] 레이더 레이어 오버레이
  - [ ] 뒤로 가기 버튼
  - [ ] 지도 컨트롤 (확대/축소)
- [ ] `app/profile/page.tsx`
  - [ ] 설정 섹션 레이아웃
  - [ ] 온도 단위 토글 (Celsius/Fahrenheit)
  - [ ] 설정 localStorage 저장
  - [ ] 테마 설정 UI (Phase 3 대비)
  - [ ] 앱 정보 섹션 (버전, 라이선스)

---

## Phase 7: 테스팅 (Week 5)

### 7.1 단위 테스트 완성 (코어 로직만)
- [ ] 테스트 커버리지 도구 설정
  - [ ] `jest --coverage` 스크립트 추가
  - [ ] `coverageThreshold` 설정 (jest.config.js)
  - [ ] 커버리지 리포트 폴더 gitignore 추가
- [ ] 유틸리티 함수 테스트
  - [ ] `lib/utils/temperature.ts` 커버리지 100% 확인
  - [ ] `lib/utils/formatters.ts` 커버리지 100% 확인
  - [ ] `lib/utils/wind.ts` 커버리지 100% 확인
  - [ ] `lib/utils/weatherIcons.ts` 커버리지 100% 확인
  - [ ] `lib/utils/validators.ts` 커버리지 100% 확인
- [ ] API 클라이언트 테스트
  - [ ] `lib/api/openweathermap.ts` 커버리지 85% 이상 확인
  - [ ] `lib/api/geocoding.ts` 커버리지 85% 이상 확인
- [ ] 커스텀 훅 테스트
  - [ ] `useGeolocation` 테스트 완료
  - [ ] `useWeather` 테스트 완료
  - [ ] `useLocation` 테스트 완료
- [ ] 커버리지 목표 달성 확인
  - [ ] 코어 로직 전체 90% 이상
  - [ ] 유틸리티 함수 100%
  - [ ] API 클라이언트 85% 이상

### 7.2 UI 수동 테스트
- [ ] 메인 페이지 수동 테스트
  - [ ] 현재 날씨 표시 확인
  - [ ] 시간별 예보 스크롤 확인
  - [ ] 10일 예보 표시 확인
  - [ ] 상세 정보 카드 확인
  - [ ] 반응형 디자인 확인 (모바일/데스크톱)
- [ ] Cities 페이지 수동 테스트
  - [ ] 도시 검색 기능 확인
  - [ ] 도시 추가/삭제 확인
  - [ ] 도시 선택 시 이동 확인
- [ ] Radar 페이지 수동 테스트
  - [ ] 지도 표시 확인
  - [ ] 레이더 레이어 확인
- [ ] Profile 페이지 수동 테스트
  - [ ] 설정 변경 확인
  - [ ] 온도 단위 토글 확인

---

## Phase 8: 최적화 및 배포 준비 (Week 5-6)

### 8.1 성능 최적화
- [ ] 이미지 최적화
  - [ ] next/image 사용
  - [ ] WebP 포맷 적용
- [ ] 코드 스플리팅
  - [ ] 지도 컴포넌트 동적 임포트
  - [ ] 라우트별 코드 분할
- [ ] 메모이제이션
  - [ ] React.memo 적용
  - [ ] useMemo, useCallback 최적화
- [ ] 번들 크기 분석
  - [ ] `@next/bundle-analyzer` 설치
  - [ ] 불필요한 의존성 제거

### 8.2 접근성 개선
- [ ] 시맨틱 HTML 적용
- [ ] ARIA 속성 추가
- [ ] 키보드 네비게이션 테스트
- [ ] 스크린 리더 테스트
- [ ] 색상 대비 검증

### 8.3 SEO 최적화
- [ ] 메타 태그 추가
  - [ ] title, description
  - [ ] Open Graph 태그
  - [ ] Twitter Card 태그
- [ ] robots.txt 생성
- [ ] sitemap.xml 생성

---

## Phase 9: CI/CD 및 배포 (Week 6)

### 9.1 GitHub Actions 설정
- [ ] `.github/workflows/deploy.yml` 작성
  - [ ] Workflow 이름 및 트리거 설정 (push to main)
  - [ ] Permissions 설정 (contents: read, pages: write, id-token: write)
  - [ ] Concurrency 설정
  - [ ] Build job 작성
    - [ ] Checkout 단계 (actions/checkout@v4)
    - [ ] Node.js 설정 단계 (actions/setup-node@v4, node 18)
    - [ ] 의존성 설치 단계 (npm ci)
    - [ ] 린트 단계 (npm run lint)
    - [ ] 테스트 단계 (npm run test)
    - [ ] 빌드 단계 (npm run build, 환경 변수 주입)
    - [ ] Upload artifact 단계 (actions/upload-pages-artifact@v3)
  - [ ] Deploy job 작성
    - [ ] Environment 설정 (github-pages)
    - [ ] Deploy 단계 (actions/deploy-pages@v4)
- [ ] GitHub Secrets 설정
  - [ ] Repository Settings → Secrets and variables → Actions 이동
  - [ ] `NEXT_PUBLIC_OPENWEATHER_API_KEY` 추가
  - [ ] `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` 추가
  - [ ] Secrets 값 확인

### 9.2 GitHub Pages 설정
- [ ] Repository Settings 설정
  - [ ] Pages 활성화
  - [ ] Source: GitHub Actions 선택
- [ ] 첫 배포 테스트
- [ ] 배포 URL 확인

### 9.3 문서화
- [ ] README.md 작성
  - [ ] 프로젝트 소개
  - [ ] 설치 방법
  - [ ] 실행 방법
  - [ ] 환경 변수 설정
  - [ ] 라이선스
- [ ] CONTRIBUTING.md 작성
- [ ] 코드 주석 추가

---

## Phase 10: Phase 2 기능 (추가 기능)

### 10.1 알림 기능
- [ ] 날씨 알림 설정 UI
- [ ] 브라우저 알림 권한 요청
- [ ] 알림 스케줄링

### 10.2 위젯
- [ ] 미니 위젯 컴포넌트
- [ ] 위젯 커스터마이징

### 10.3 라이트 모드
- [ ] 라이트 테마 색상 정의
- [ ] 테마 토글 기능
- [ ] 시스템 테마 감지

---

## 체크리스트 규칙

### TDD 규칙 준수
- 모든 코어 로직은 테스트 먼저 작성
- Red → Green → Refactor 사이클 준수
- 테스트 커버리지 목표 달성

### SOLID 원칙 준수
- SRP: 단일 책임 원칙
- OCP: 개방/폐쇄 원칙
- LSP: 리스코프 치환 원칙
- ISP: 인터페이스 분리 원칙
- DIP: 의존성 역전 원칙

### 코드 리뷰
- 각 Phase 완료 후 코드 리뷰
- 린트 에러 0개
- 테스트 통과율 100%
