# Technical Specification: Weather Forecaster 앱

## 1. 기술 스택 개요

### 1.1 아키텍처
- **아키텍처 패턴**: Single Page Application (SPA)
- **렌더링 방식**: Client-Side Rendering (CSR) with Static Site Generation (SSG) 옵션
- **상태 관리**: React Context API + Custom Hooks
- **데이터 페칭**: SWR (stale-while-revalidate)

### 1.2 핵심 기술
| 카테고리 | 기술 | 버전 | 목적 |
|---------|------|------|------|
| 프레임워크 | Next.js | 14.x | React 프레임워크, SSG/SSR 지원 |
| UI 라이브러리 | React | 18.x | 컴포넌트 기반 UI |
| 언어 | TypeScript | 5.x | 타입 안정성 |
| 스타일링 | Tailwind CSS | 3.x | 유틸리티 우선 CSS |
| 아이콘 | Material Symbols | Latest | 구글 머티리얼 아이콘 |
| 폰트 | Google Fonts (Inter) | Latest | 타이포그래피 |

## 2. 프론트엔드 상세

### 2.1 프레임워크: Next.js 14

#### 선택 이유
- **App Router**: 최신 React Server Components 지원
- **이미지 최적화**: next/image로 자동 최적화
- **폰트 최적화**: next/font로 자동 폰트 최적화
- **API Routes**: 서버리스 API 엔드포인트 제공
- **Static Export**: 정적 사이트로 배포 가능

#### 프로젝트 구조
```
weather-forecaster-demo/
├── app/
│   ├── layout.tsx           # 루트 레이아웃
│   ├── page.tsx             # 메인 페이지 (날씨)
│   ├── cities/
│   │   └── page.tsx         # 도시 목록 페이지
│   ├── radar/
│   │   └── page.tsx         # 레이더 페이지
│   ├── profile/
│   │   └── page.tsx         # 프로필 페이지
│   └── api/
│       └── weather/
│           └── route.ts     # 날씨 API 프록시
├── components/
│   ├── ui/                  # 재사용 가능한 UI 컴포넌트
│   │   ├── Card.tsx
│   │   ├── Button.tsx
│   │   └── Icon.tsx
│   ├── weather/             # 날씨 관련 컴포넌트
│   │   ├── CurrentWeather.tsx
│   │   ├── HourlyForecast.tsx
│   │   ├── DailyForecast.tsx
│   │   └── WeatherDetails.tsx
│   ├── layout/              # 레이아웃 컴포넌트
│   │   ├── Header.tsx
│   │   ├── TabBar.tsx
│   │   └── Container.tsx
│   └── map/
│       └── PrecipitationMap.tsx
├── lib/
│   ├── api/                 # API 클라이언트
│   │   ├── weather.ts
│   │   └── geocoding.ts
│   ├── hooks/               # 커스텀 훅
│   │   ├── useWeather.ts
│   │   ├── useLocation.ts
│   │   └── useGeolocation.ts
│   ├── utils/               # 유틸리티 함수
│   │   ├── formatters.ts
│   │   ├── weatherIcons.ts
│   │   └── temperature.ts
│   └── types/               # TypeScript 타입 정의
│       └── weather.ts
├── public/
│   └── images/
├── styles/
│   └── globals.css          # Tailwind 설정
└── tailwind.config.ts       # Tailwind 구성
```

### 2.2 스타일링: Tailwind CSS

#### 커스텀 설정
```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: 'class',
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: '#2b8cee',
        'background-light': '#f6f7f8',
        'background-dark': '#101922',
        'card-dark': '#1c252e',
      },
      fontFamily: {
        display: ['var(--font-inter)', 'sans-serif'],
      },
      borderRadius: {
        DEFAULT: '0.25rem',
        lg: '0.5rem',
        xl: '0.75rem',
        '2xl': '1rem',
        '3xl': '1.5rem',
        full: '9999px',
      },
      backdropBlur: {
        md: '12px',
        xl: '24px',
      },
    },
  },
  plugins: [],
}
```

#### 커스텀 CSS 클래스
```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .no-scrollbar::-webkit-scrollbar {
    display: none;
  }
  
  .no-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
  }
  
  .temp-gradient {
    background: linear-gradient(180deg, #FFFFFF 0%, #A3C9F5 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
}
```

### 2.3 상태 관리

#### Context API 구조
```typescript
// lib/contexts/WeatherContext.tsx
interface WeatherContextType {
  currentWeather: CurrentWeather | null;
  hourlyForecast: HourlyForecast[];
  dailyForecast: DailyForecast[];
  weatherDetails: WeatherDetails | null;
  location: Location | null;
  isLoading: boolean;
  error: Error | null;
  refreshWeather: () => Promise<void>;
  setLocation: (location: Location) => void;
}
```

#### 커스텀 훅
```typescript
// lib/hooks/useWeather.ts
export function useWeather(location?: Location) {
  const { data, error, isLoading, mutate } = useSWR(
    location ? `/api/weather?lat=${location.lat}&lon=${location.lon}` : null,
    fetcher,
    {
      refreshInterval: 600000, // 10분마다 갱신
      revalidateOnFocus: true,
      dedupingInterval: 60000,
    }
  );
  
  return {
    weather: data,
    isLoading,
    error,
    refresh: mutate,
  };
}
```

### 2.4 데이터 페칭: SWR

#### 선택 이유
- **자동 재검증**: 포커스, 네트워크 복구 시 자동 갱신
- **캐싱**: 중복 요청 제거
- **낙관적 업데이트**: 빠른 UI 반응
- **오류 재시도**: 자동 재시도 메커니즘

#### 설정
```typescript
// app/layout.tsx
import { SWRConfig } from 'swr';

const swrConfig = {
  refreshInterval: 600000, // 10분
  revalidateOnFocus: true,
  revalidateOnReconnect: true,
  dedupingInterval: 60000, // 1분
  errorRetryCount: 3,
  errorRetryInterval: 5000,
  fetcher: (url: string) => fetch(url).then(res => res.json()),
};
```

## 3. 백엔드 / API

### 3.1 날씨 API: OpenWeatherMap

#### API 엔드포인트
| 엔드포인트 | 목적 | 호출 빈도 |
|-----------|------|----------|
| Current Weather | 현재 날씨 | 10분마다 |
| One Call API 3.0 | 시간별/일별 예보, 상세 정보 | 10분마다 |
| Geocoding API | 도시 검색 | 필요 시 |
| Air Pollution API | 대기질 (Phase 3) | 30분마다 |

#### API 키 관리
```typescript
// .env.local
NEXT_PUBLIC_OPENWEATHER_API_KEY=your_api_key_here
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_key_here
```

#### API 프록시 (Next.js API Routes)
```typescript
// app/api/weather/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const lat = searchParams.get('lat');
  const lon = searchParams.get('lon');
  
  const response = await fetch(
    `https://api.openweathermap.org/data/3.0/onecall?lat=${lat}&lon=${lon}&appid=${process.env.OPENWEATHER_API_KEY}&units=metric&lang=kr`
  );
  
  const data = await response.json();
  return NextResponse.json(data);
}
```

### 3.2 위치 서비스

#### Geolocation API (브라우저)
```typescript
// lib/hooks/useGeolocation.ts
export function useGeolocation() {
  const [location, setLocation] = useState<GeolocationCoordinates | null>(null);
  const [error, setError] = useState<GeolocationPositionError | null>(null);
  
  useEffect(() => {
    if (!navigator.geolocation) {
      setError(new Error('Geolocation not supported'));
      return;
    }
    
    navigator.geolocation.getCurrentPosition(
      (position) => setLocation(position.coords),
      (error) => setError(error),
      { enableHighAccuracy: true, timeout: 10000, maximumAge: 300000 }
    );
  }, []);
  
  return { location, error };
}
```

#### Geocoding (도시 → 좌표)
```typescript
// lib/api/geocoding.ts
export async function searchCity(cityName: string) {
  const response = await fetch(
    `https://api.openweathermap.org/geo/1.0/direct?q=${cityName}&limit=5&appid=${API_KEY}`
  );
  return response.json();
}
```

### 3.3 지도 API: Google Maps

#### 사용 목적
- 강수량 지도 표시
- 레이더 지도 (Phase 3)

#### 통합 방법
```typescript
// components/map/PrecipitationMap.tsx
import { GoogleMap, LoadScript, GroundOverlay } from '@react-google-maps/api';

export function PrecipitationMap({ center }: { center: LatLng }) {
  return (
    <LoadScript googleMapsApiKey={process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY!}>
      <GoogleMap
        mapContainerStyle={{ width: '100%', height: '128px' }}
        center={center}
        zoom={10}
        options={{
          styles: darkMapStyles, // 다크 테마
          disableDefaultUI: true,
        }}
      >
        <GroundOverlay
          url={`https://tile.openweathermap.org/map/precipitation_new/{z}/{x}/{y}.png?appid=${API_KEY}`}
          bounds={bounds}
        />
      </GoogleMap>
    </LoadScript>
  );
}
```

## 4. 타입 시스템 (TypeScript)

### 4.1 핵심 타입 정의

```typescript
// lib/types/weather.ts

export interface Location {
  lat: number;
  lon: number;
  name: string;
  country: string;
}

export interface CurrentWeather {
  location: string;
  temperature: number;
  feelsLike: number;
  condition: string;
  icon: string;
  high: number;
  low: number;
  timestamp: Date;
}

export interface HourlyForecast {
  time: string;
  timestamp: number;
  temperature: number;
  condition: string;
  icon: string;
  precipitationChance?: number;
}

export interface DailyForecast {
  day: string;
  date: Date;
  condition: string;
  icon: string;
  precipitationChance?: number;
  high: number;
  low: number;
  sunrise: number;
  sunset: number;
}

export interface WeatherDetails {
  uvIndex: {
    value: number;
    level: 'Low' | 'Moderate' | 'High' | 'Very High' | 'Extreme';
    advice: string;
  };
  humidity: {
    percentage: number;
    dewPoint: number;
    description: string;
  };
  wind: {
    speed: number;
    direction: string;
    degrees: number;
    gust?: number;
  };
  visibility: {
    distance: number;
    description: string;
    forecast: string;
  };
  pressure?: number;
  cloudCover?: number;
}

export interface WeatherResponse {
  current: CurrentWeather;
  hourly: HourlyForecast[];
  daily: DailyForecast[];
  details: WeatherDetails;
}
```

### 4.2 유틸리티 타입

```typescript
// lib/types/utils.ts

export type WeatherCondition = 
  | 'Clear'
  | 'Clouds'
  | 'Rain'
  | 'Drizzle'
  | 'Thunderstorm'
  | 'Snow'
  | 'Mist'
  | 'Fog';

export type TemperatureUnit = 'celsius' | 'fahrenheit';

export type WindDirection = 'N' | 'NE' | 'E' | 'SE' | 'S' | 'SW' | 'W' | 'NW';

export interface ApiError {
  code: string;
  message: string;
  statusCode: number;
}
```

## 5. 성능 최적화

### 5.1 이미지 최적화
```typescript
// next.config.js
module.exports = {
  images: {
    domains: ['openweathermap.org', 'maps.googleapis.com'],
    formats: ['image/avif', 'image/webp'],
  },
};
```

### 5.2 코드 스플리팅
```typescript
// 동적 임포트로 번들 크기 최적화
import dynamic from 'next/dynamic';

const PrecipitationMap = dynamic(
  () => import('@/components/map/PrecipitationMap'),
  { 
    loading: () => <MapSkeleton />,
    ssr: false, // 클라이언트에서만 로드
  }
);
```

### 5.3 캐싱 전략
```typescript
// 로컬 스토리지 캐싱
export function cacheWeatherData(location: string, data: WeatherResponse) {
  const cacheKey = `weather_${location}`;
  const cacheData = {
    data,
    timestamp: Date.now(),
  };
  localStorage.setItem(cacheKey, JSON.stringify(cacheData));
}

export function getCachedWeatherData(location: string, maxAge = 600000) {
  const cacheKey = `weather_${location}`;
  const cached = localStorage.getItem(cacheKey);
  
  if (!cached) return null;
  
  const { data, timestamp } = JSON.parse(cached);
  if (Date.now() - timestamp > maxAge) return null;
  
  return data;
}
```

### 5.4 메모이제이션
```typescript
import { memo, useMemo } from 'react';

export const HourlyForecastCard = memo(({ forecast }: { forecast: HourlyForecast }) => {
  const icon = useMemo(() => getWeatherIcon(forecast.condition), [forecast.condition]);
  
  return (
    <div className="flex flex-col items-center gap-3">
      <span className="text-sm">{forecast.time}</span>
      {icon}
      <span className="text-lg font-medium">{forecast.temperature}°</span>
    </div>
  );
});
```

## 6. 테스팅 전략

### 6.1 테스팅 도구
| 도구 | 목적 | 버전 |
|------|------|------|
| Jest | 단위 테스트 (코어 로직만) | 29.x |
| MSW | API 모킹 | 2.x |

**주의**: UI 컴포넌트는 자동화 테스트를 작성하지 않습니다. 수동 테스트로만 검증합니다.

### 6.2 테스트 구조
```
__tests__/
└── unit/
    ├── utils/
    │   ├── formatters.test.ts
    │   ├── temperature.test.ts
    │   ├── wind.test.ts
    │   ├── weatherIcons.test.ts
    │   └── validators.test.ts
    ├── api/
    │   ├── openweathermap.test.ts
    │   └── geocoding.test.ts
    ├── transformers/
    │   └── weatherTransformer.test.ts
    ├── cache/
    │   ├── LocalStorageCache.test.ts
    │   └── MemoryCache.test.ts
    └── hooks/
        ├── useWeather.test.ts
        ├── useGeolocation.test.ts
        └── useLocation.test.ts
```

### 6.3 테스트 예시
```typescript
// __tests__/unit/utils/formatters.test.ts
import { formatTemperature, getWindDirection } from '@/lib/utils/formatters';

describe('formatTemperature', () => {
  it('should format celsius temperature', () => {
    expect(formatTemperature(24, 'celsius')).toBe('24°');
  });
  
  it('should convert to fahrenheit', () => {
    expect(formatTemperature(24, 'fahrenheit')).toBe('75°');
  });
});

describe('getWindDirection', () => {
  it('should return correct direction for degrees', () => {
    expect(getWindDirection(0)).toBe('N');
    expect(getWindDirection(45)).toBe('NE');
    expect(getWindDirection(315)).toBe('NW');
  });
});
```

## 7. 빌드 및 배포

### 7.1 Next.js 정적 내보내기 설정
```typescript
// next.config.js
module.exports = {
  output: 'export',
  basePath: '/weather-forecaster-demo', // GitHub Pages 저장소 이름
  images: {
    unoptimized: true, // GitHub Pages는 이미지 최적화 미지원
  },
  trailingSlash: true,
};
```

### 7.2 빌드 스크립트
```json
// package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest",
    "test:e2e": "playwright test"
  }
}
```

### 7.3 배포 플랫폼: GitHub Pages
| 특징 | 설명 |
|------|------|
| 호스팅 | GitHub Pages (무료) |
| 빌드 | GitHub Actions |
| URL | `https://<username>.github.io/weather-forecaster-demo` |
| 장점 | 무료, Git 통합, 자동 배포 |

### 7.4 환경 변수 관리
```bash
# .env.local (개발)
NEXT_PUBLIC_OPENWEATHER_API_KEY=dev_key
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=dev_key
```

**GitHub Secrets 설정** (Settings → Secrets and variables → Actions):
- `NEXT_PUBLIC_OPENWEATHER_API_KEY`: OpenWeatherMap API 키
- `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`: Google Maps API 키

### 7.5 GitHub Actions CI/CD 파이프라인
```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm run test

      - name: Build Next.js app
        env:
          NEXT_PUBLIC_OPENWEATHER_API_KEY: ${{ secrets.NEXT_PUBLIC_OPENWEATHER_API_KEY }}
          NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: ${{ secrets.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY }}
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./out

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 7.6 GitHub Pages 설정
1. 저장소 Settings → Pages로 이동
2. Source: **GitHub Actions** 선택
3. 첫 배포 후 `https://<username>.github.io/weather-forecaster-demo`에서 확인

## 8. 보안

### 8.1 API 키 보호
- 환경 변수로 관리 (`.env.local`)
- Next.js API Routes로 프록시하여 클라이언트 노출 방지
- Rate limiting 적용

### 8.2 HTTPS
- GitHub Pages 자동 SSL 인증서 제공
- 모든 API 호출 HTTPS 강제

### 8.3 CSP (Content Security Policy)
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline' cdn.tailwindcss.com; style-src 'self' 'unsafe-inline' fonts.googleapis.com; font-src fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' api.openweathermap.org maps.googleapis.com;"
  }
];
```

## 9. 접근성 (a11y)

### 9.1 시맨틱 HTML
```tsx
<main role="main">
  <section aria-label="Current Weather">
    <h1 className="sr-only">Current Temperature</h1>
    <div aria-live="polite" aria-atomic="true">
      {temperature}°
    </div>
  </section>
</main>
```

### 9.2 키보드 네비게이션
- 모든 인터랙티브 요소에 `tabIndex` 설정
- 포커스 스타일 명확히 표시
- Skip to content 링크 제공

### 9.3 스크린 리더 지원
```tsx
<button aria-label="Open settings menu">
  <span className="material-symbols-outlined" aria-hidden="true">
    settings
  </span>
</button>
```

## 10. 모니터링 및 분석

### 10.1 성능 모니터링
- **Web Vitals**: Next.js 내장 지원
- **Lighthouse CI**: 자동 성능 측정

### 10.2 에러 트래킹
```typescript
// lib/errorTracking.ts
export function logError(error: Error, context?: Record<string, any>) {
  console.error('Error:', error, context);
  
  // 프로덕션에서는 Sentry 등으로 전송
  if (process.env.NODE_ENV === 'production') {
    // Sentry.captureException(error, { extra: context });
  }
}
```

### 10.3 사용자 분석
- Google Analytics 4
- 커스텀 이벤트 트래킹 (도시 검색, 지도 열기 등)

## 11. 개발 도구

### 11.1 필수 도구
- **Node.js**: 18.x 이상
- **npm/yarn/pnpm**: 패키지 매니저
- **VS Code**: 권장 에디터
- **Git**: 버전 관리

### 11.2 VS Code 확장
- ESLint
- Prettier
- Tailwind CSS IntelliSense
- TypeScript Vue Plugin (Volar)

### 11.3 린팅 및 포맷팅
```json
// .eslintrc.json
{
  "extends": ["next/core-web-vitals", "prettier"],
  "rules": {
    "react/no-unescaped-entities": "off",
    "@next/next/no-page-custom-font": "off"
  }
}

// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "tabWidth": 2,
  "printWidth": 100
}
```

## 12. 의존성 목록

### 12.1 프로덕션 의존성
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "swr": "^2.2.0",
    "@react-google-maps/api": "^2.19.0",
    "date-fns": "^3.0.0"
  }
}
```

### 12.2 개발 의존성
```json
{
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0",
    "eslint": "^8.0.0",
    "eslint-config-next": "^14.0.0",
    "prettier": "^3.0.0",
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0",
    "ts-jest": "^29.0.0",
    "msw": "^2.0.0"
  }
}
```

## 13. 마일스톤 및 일정

### Phase 1: MVP (2주)
- Week 1: 프로젝트 설정, 기본 UI 구현
- Week 2: API 통합, 현재 날씨 + 예보 기능

### Phase 2: 고급 기능 (2주)
- Week 3: 상세 정보, 지도 통합
- Week 4: 도시 검색, 설정 페이지

### Phase 3: 확장 (2주)
- Week 5: 레이더, 알림 기능
- Week 6: 테스팅, 최적화, 배포

## 14. 참고 문서

- [Next.js Documentation](https://nextjs.org/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [OpenWeatherMap API](https://openweathermap.org/api)
- [Google Maps JavaScript API](https://developers.google.com/maps/documentation/javascript)
- [Material Symbols](https://fonts.google.com/icons)
- [SWR Documentation](https://swr.vercel.app/)
