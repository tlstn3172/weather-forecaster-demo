# SOLID 원칙

## 원칙

이 프로젝트의 모든 구현은 **SOLID 원칙**을 따릅니다.

## SOLID 5가지 원칙

### 1. SRP (Single Responsibility Principle) - 단일 책임 원칙

**정의**: 하나의 클래스/함수/모듈은 하나의 책임만 가져야 합니다.

#### ✅ 좋은 예
```typescript
// lib/utils/temperature.ts
export function celsiusToFahrenheit(celsius: number): number {
  return celsius * 9/5 + 32;
}

export function fahrenheitToCelsius(fahrenheit: number): number {
  return (fahrenheit - 32) * 5/9;
}

// lib/utils/formatters.ts
export function formatTemperature(temp: number, unit: TemperatureUnit): string {
  return `${Math.round(temp)}°`;
}
```

#### ❌ 나쁜 예
```typescript
// 하나의 함수가 너무 많은 책임을 가짐
export function processWeatherData(data: any) {
  // 1. API 응답 파싱
  // 2. 데이터 검증
  // 3. 온도 변환
  // 4. 포맷팅
  // 5. 캐싱
  // 너무 많은 책임!
}
```

### 2. OCP (Open/Closed Principle) - 개방/폐쇄 원칙

**정의**: 확장에는 열려있고, 수정에는 닫혀있어야 합니다.

#### ✅ 좋은 예
```typescript
// lib/utils/weatherIcons.ts
interface WeatherIconMapper {
  getIcon(condition: string): string;
}

class MaterialIconMapper implements WeatherIconMapper {
  getIcon(condition: string): string {
    const iconMap: Record<string, string> = {
      'Clear': 'wb_sunny',
      'Clouds': 'cloud',
      'Rain': 'rainy',
    };
    return iconMap[condition] || 'help';
  }
}

// 새로운 아이콘 시스템 추가 시 기존 코드 수정 없이 확장
class EmojiIconMapper implements WeatherIconMapper {
  getIcon(condition: string): string {
    const iconMap: Record<string, string> = {
      'Clear': '☀️',
      'Clouds': '☁️',
      'Rain': '🌧️',
    };
    return iconMap[condition] || '❓';
  }
}
```

#### ❌ 나쁜 예
```typescript
// 새로운 아이콘 타입 추가 시 함수 수정 필요
export function getWeatherIcon(condition: string, type: 'material' | 'emoji' | 'fontawesome') {
  if (type === 'material') {
    // ...
  } else if (type === 'emoji') {
    // ...
  } else if (type === 'fontawesome') {
    // 새로운 타입 추가 시마다 수정 필요
  }
}
```

### 3. LSP (Liskov Substitution Principle) - 리스코프 치환 원칙

**정의**: 하위 타입은 상위 타입을 대체할 수 있어야 합니다.

#### ✅ 좋은 예
```typescript
// lib/api/weatherProvider.ts
interface WeatherProvider {
  getCurrentWeather(lat: number, lon: number): Promise<WeatherData>;
  getForecast(lat: number, lon: number): Promise<ForecastData>;
}

class OpenWeatherMapProvider implements WeatherProvider {
  async getCurrentWeather(lat: number, lon: number): Promise<WeatherData> {
    // OpenWeatherMap API 호출
  }
  
  async getForecast(lat: number, lon: number): Promise<ForecastData> {
    // OpenWeatherMap API 호출
  }
}

class WeatherAPIProvider implements WeatherProvider {
  async getCurrentWeather(lat: number, lon: number): Promise<WeatherData> {
    // WeatherAPI.com 호출
  }
  
  async getForecast(lat: number, lon: number): Promise<ForecastData> {
    // WeatherAPI.com 호출
  }
}

// 어떤 구현체를 사용하든 동일하게 동작
function useWeatherData(provider: WeatherProvider) {
  const weather = await provider.getCurrentWeather(37.5, 127.0);
  // provider의 구체적인 타입과 무관하게 동작
}
```

### 4. ISP (Interface Segregation Principle) - 인터페이스 분리 원칙

**정의**: 클라이언트는 사용하지 않는 인터페이스에 의존하지 않아야 합니다.

#### ✅ 좋은 예
```typescript
// lib/types/weather.ts
interface CurrentWeatherData {
  temperature: number;
  condition: string;
  humidity: number;
}

interface ForecastData {
  hourly: HourlyForecast[];
  daily: DailyForecast[];
}

interface WeatherDetails {
  uvIndex: number;
  windSpeed: number;
  visibility: number;
}

// 필요한 인터페이스만 사용
class CurrentWeatherDisplay {
  constructor(private data: CurrentWeatherData) {}
  // CurrentWeatherData만 필요
}

class ForecastDisplay {
  constructor(private data: ForecastData) {}
  // ForecastData만 필요
}
```

#### ❌ 나쁜 예
```typescript
// 모든 것을 포함하는 거대한 인터페이스
interface WeatherData {
  temperature: number;
  condition: string;
  humidity: number;
  hourly: HourlyForecast[];
  daily: DailyForecast[];
  uvIndex: number;
  windSpeed: number;
  visibility: number;
  // ... 더 많은 필드
}

// 현재 날씨만 표시하는데 모든 데이터가 필요
class CurrentWeatherDisplay {
  constructor(private data: WeatherData) {}
  // 대부분의 필드를 사용하지 않음
}
```

### 5. DIP (Dependency Inversion Principle) - 의존성 역전 원칙

**정의**: 고수준 모듈은 저수준 모듈에 의존하지 않아야 하며, 둘 다 추상화에 의존해야 합니다.

#### ✅ 좋은 예
```typescript
// lib/services/weatherService.ts
interface WeatherRepository {
  fetchWeather(lat: number, lon: number): Promise<WeatherData>;
}

interface CacheStorage {
  get(key: string): WeatherData | null;
  set(key: string, data: WeatherData): void;
}

// 고수준 모듈: 추상화에 의존
class WeatherService {
  constructor(
    private repository: WeatherRepository,
    private cache: CacheStorage
  ) {}
  
  async getWeather(lat: number, lon: number): Promise<WeatherData> {
    const cacheKey = `${lat},${lon}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached) return cached;
    
    const weather = await this.repository.fetchWeather(lat, lon);
    this.cache.set(cacheKey, weather);
    
    return weather;
  }
}

// 저수준 모듈: 추상화 구현
class OpenWeatherMapRepository implements WeatherRepository {
  async fetchWeather(lat: number, lon: number): Promise<WeatherData> {
    // 구체적인 API 호출
  }
}

class LocalStorageCache implements CacheStorage {
  get(key: string): WeatherData | null {
    // localStorage 사용
  }
  
  set(key: string, data: WeatherData): void {
    // localStorage 사용
  }
}
```

#### ❌ 나쁜 예
```typescript
// 고수준 모듈이 저수준 모듈에 직접 의존
class WeatherService {
  private api = new OpenWeatherMapAPI(); // 구체적인 구현에 의존
  private storage = new LocalStorage(); // 구체적인 구현에 의존
  
  async getWeather(lat: number, lon: number): Promise<WeatherData> {
    // API나 Storage를 교체하려면 WeatherService 수정 필요
  }
}
```

## 프로젝트 적용 가이드

### 파일 구조에서의 SOLID

```
lib/
├── interfaces/          # 추상화 (DIP)
│   ├── WeatherProvider.ts
│   ├── CacheStorage.ts
│   └── LocationService.ts
├── services/           # 고수준 비즈니스 로직 (SRP, DIP)
│   ├── WeatherService.ts
│   └── LocationService.ts
├── repositories/       # 데이터 접근 계층 (SRP, DIP)
│   ├── OpenWeatherMapRepository.ts
│   └── GeocodingRepository.ts
├── utils/             # 단일 책임 유틸리티 (SRP)
│   ├── temperature.ts
│   ├── formatters.ts
│   └── validators.ts
└── types/             # 타입 정의 (ISP)
    ├── weather.ts
    ├── location.ts
    └── forecast.ts
```

### 의존성 주입 패턴

```typescript
// lib/hooks/useWeather.ts
export function useWeather(
  repository: WeatherRepository = new OpenWeatherMapRepository(),
  cache: CacheStorage = new LocalStorageCache()
) {
  const service = new WeatherService(repository, cache);
  
  // ...
}
```

## 체크리스트

코드 작성 시 확인사항:

### SRP
- [ ] 이 함수/클래스가 하나의 책임만 가지는가?
- [ ] 변경 이유가 하나뿐인가?
- [ ] 함수 이름이 하는 일을 정확히 설명하는가?

### OCP
- [ ] 새로운 기능 추가 시 기존 코드 수정이 필요한가?
- [ ] 인터페이스나 추상 클래스를 사용하는가?
- [ ] 확장 가능한 구조인가?

### LSP
- [ ] 하위 타입이 상위 타입의 계약을 위반하지 않는가?
- [ ] 하위 타입으로 교체해도 프로그램이 정상 동작하는가?

### ISP
- [ ] 인터페이스가 너무 크지 않은가?
- [ ] 사용하지 않는 메서드를 구현하도록 강제하지 않는가?
- [ ] 인터페이스를 더 작은 단위로 분리할 수 있는가?

### DIP
- [ ] 구체적인 구현이 아닌 추상화에 의존하는가?
- [ ] 의존성을 주입받는 구조인가?
- [ ] 구현체를 쉽게 교체할 수 있는가?

## 실전 예제

### 날씨 데이터 처리

```typescript
// ✅ SOLID 원칙 적용
// lib/interfaces/WeatherDataProcessor.ts
interface WeatherDataProcessor {
  process(rawData: any): WeatherData;
}

// lib/processors/OpenWeatherMapProcessor.ts
export class OpenWeatherMapProcessor implements WeatherDataProcessor {
  process(rawData: any): WeatherData {
    return {
      temperature: rawData.main.temp,
      condition: rawData.weather[0].main,
      humidity: rawData.main.humidity,
    };
  }
}

// lib/processors/WeatherAPIProcessor.ts
export class WeatherAPIProcessor implements WeatherDataProcessor {
  process(rawData: any): WeatherData {
    return {
      temperature: rawData.current.temp_c,
      condition: rawData.current.condition.text,
      humidity: rawData.current.humidity,
    };
  }
}

// lib/services/WeatherService.ts
export class WeatherService {
  constructor(
    private repository: WeatherRepository,
    private processor: WeatherDataProcessor
  ) {}
  
  async getWeather(lat: number, lon: number): Promise<WeatherData> {
    const rawData = await this.repository.fetchRawData(lat, lon);
    return this.processor.process(rawData);
  }
}
```

### 캐싱 전략

```typescript
// ✅ SOLID 원칙 적용
// lib/interfaces/CacheStrategy.ts
interface CacheStrategy {
  get(key: string): any | null;
  set(key: string, value: any, ttl?: number): void;
  clear(): void;
}

// lib/cache/LocalStorageCache.ts
export class LocalStorageCache implements CacheStrategy {
  get(key: string): any | null {
    const item = localStorage.getItem(key);
    if (!item) return null;
    
    const { value, expiry } = JSON.parse(item);
    if (Date.now() > expiry) {
      localStorage.removeItem(key);
      return null;
    }
    
    return value;
  }
  
  set(key: string, value: any, ttl: number = 600000): void {
    const item = {
      value,
      expiry: Date.now() + ttl,
    };
    localStorage.setItem(key, JSON.stringify(item));
  }
  
  clear(): void {
    localStorage.clear();
  }
}

// lib/cache/MemoryCache.ts
export class MemoryCache implements CacheStrategy {
  private cache = new Map<string, { value: any; expiry: number }>();
  
  get(key: string): any | null {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  set(key: string, value: any, ttl: number = 600000): void {
    this.cache.set(key, {
      value,
      expiry: Date.now() + ttl,
    });
  }
  
  clear(): void {
    this.cache.clear();
  }
}
```

## 금지 사항

❌ **God Object (신 객체)**
```typescript
// 모든 것을 하는 거대한 클래스
class WeatherManager {
  fetchWeather() {}
  processWeather() {}
  cacheWeather() {}
  formatWeather() {}
  validateWeather() {}
  displayWeather() {}
  // 너무 많은 책임!
}
```

❌ **구체적인 구현에 의존**
```typescript
class WeatherService {
  private api = new OpenWeatherMapAPI(); // 구체적인 클래스에 의존
  
  async getWeather() {
    return this.api.fetch(); // 교체 불가능
  }
}
```

❌ **거대한 인터페이스**
```typescript
interface WeatherService {
  getCurrentWeather(): Promise<Weather>;
  getHourlyForecast(): Promise<Forecast[]>;
  getDailyForecast(): Promise<Forecast[]>;
  getHistoricalData(): Promise<Historical[]>;
  getAlerts(): Promise<Alert[]>;
  // 너무 많은 메서드!
}
```

## 참고 자료

- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Dependency Injection in TypeScript](https://www.typescriptlang.org/docs/handbook/2/classes.html)
