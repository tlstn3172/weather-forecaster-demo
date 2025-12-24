# TDD (Test-Driven Development) 규칙

## 원칙

이 프로젝트의 **UI를 제외한 모든 코어 로직**은 TDD(Test-Driven Development) 방식으로 구현합니다.

## TDD 사이클

모든 코어 로직 개발 시 다음 Red-Green-Refactor 사이클을 따릅니다:

1. **Red**: 실패하는 테스트를 먼저 작성
2. **Green**: 테스트를 통과하는 최소한의 코드 작성
3. **Refactor**: 코드 품질 개선 (테스트는 여전히 통과해야 함)

## 적용 범위

### TDD 필수 적용 대상

- **유틸리티 함수** (`lib/utils/`)
  - 온도 변환 함수
  - 날씨 아이콘 매핑 함수
  - 날짜/시간 포맷팅 함수
  - 풍향 계산 함수
  - 데이터 검증 함수

- **API 클라이언트** (`lib/api/`)
  - 날씨 API 호출 로직
  - Geocoding API 호출 로직
  - 응답 데이터 변환 로직
  - 에러 핸들링 로직

- **커스텀 훅** (`lib/hooks/`)
  - `useWeather`
  - `useLocation`
  - `useGeolocation`
  - 기타 비즈니스 로직을 포함한 훅

- **비즈니스 로직**
  - 날씨 데이터 처리
  - 캐싱 로직
  - 상태 관리 로직

### TDD 제외 대상

- **UI 컴포넌트** (`components/`)
  - React 컴포넌트는 **자동화 테스트를 작성하지 않음**
  - UI는 수동 테스트로만 검증
  - TDD 사이클 적용 안 함

- **페이지** (`app/`)
  - Next.js 페이지 컴포넌트
  - 수동 테스트로만 검증

- **스타일링**
  - CSS, Tailwind 클래스
  - 시각적 검증만 수행

## 테스트 작성 가이드

### 1. 테스트 파일 위치
```
lib/utils/formatters.ts
  → __tests__/unit/utils/formatters.test.ts

lib/api/weather.ts
  → __tests__/unit/api/weather.test.ts

lib/hooks/useWeather.ts
  → __tests__/unit/hooks/useWeather.test.ts
```

### 2. 테스트 구조
```typescript
describe('함수명 또는 모듈명', () => {
  describe('특정 기능', () => {
    it('should 기대하는 동작', () => {
      // Arrange: 테스트 준비
      const input = ...;
      
      // Act: 함수 실행
      const result = functionToTest(input);
      
      // Assert: 결과 검증
      expect(result).toBe(expected);
    });
  });
});
```

### 3. 테스트 커버리지 목표
- **코어 로직**: 최소 90% 커버리지
- **유틸리티 함수**: 100% 커버리지
- **API 클라이언트**: 최소 85% 커버리지

## 개발 워크플로우

### 새로운 기능 추가 시

1. **테스트 먼저 작성**
   ```bash
   # 테스트 파일 생성
   touch __tests__/unit/utils/newFunction.test.ts
   ```

2. **실패하는 테스트 작성**
   ```typescript
   it('should convert celsius to fahrenheit', () => {
     expect(convertToFahrenheit(0)).toBe(32);
   });
   ```

3. **테스트 실행 (실패 확인)**
   ```bash
   npm run test -- newFunction.test.ts
   ```

4. **최소한의 구현**
   ```typescript
   export function convertToFahrenheit(celsius: number): number {
     return celsius * 9/5 + 32;
   }
   ```

5. **테스트 실행 (통과 확인)**
   ```bash
   npm run test -- newFunction.test.ts
   ```

6. **리팩토링**
   - 코드 품질 개선
   - 타입 안정성 강화
   - 성능 최적화

7. **최종 테스트 실행**
   ```bash
   npm run test
   ```

### 버그 수정 시

1. **버그를 재현하는 테스트 작성**
   ```typescript
   it('should handle negative temperatures correctly', () => {
     expect(convertToFahrenheit(-40)).toBe(-40);
   });
   ```

2. **테스트 실행 (실패 확인)**

3. **버그 수정**

4. **테스트 실행 (통과 확인)**

## 테스트 도구

### 단위 테스트
- **프레임워크**: Jest
- **실행 명령어**: `npm run test`
- **Watch 모드**: `npm run test:watch`

### API 모킹
- **도구**: MSW (Mock Service Worker)
- **용도**: API 호출 모킹

### 테스트 예시

```typescript
// __tests__/unit/utils/formatters.test.ts
import { formatTemperature, getWindDirection } from '@/lib/utils/formatters';

describe('formatTemperature', () => {
  it('should format positive celsius temperature', () => {
    expect(formatTemperature(24, 'celsius')).toBe('24°');
  });

  it('should format negative celsius temperature', () => {
    expect(formatTemperature(-5, 'celsius')).toBe('-5°');
  });

  it('should convert celsius to fahrenheit', () => {
    expect(formatTemperature(0, 'fahrenheit')).toBe('32°');
    expect(formatTemperature(100, 'fahrenheit')).toBe('212°');
  });

  it('should round to nearest integer', () => {
    expect(formatTemperature(24.7, 'celsius')).toBe('25°');
  });
});

describe('getWindDirection', () => {
  it('should return N for 0 degrees', () => {
    expect(getWindDirection(0)).toBe('N');
  });

  it('should return NE for 45 degrees', () => {
    expect(getWindDirection(45)).toBe('NE');
  });

  it('should return E for 90 degrees', () => {
    expect(getWindDirection(90)).toBe('E');
  });

  it('should handle 360 degrees as N', () => {
    expect(getWindDirection(360)).toBe('N');
  });

  it('should round to nearest direction', () => {
    expect(getWindDirection(23)).toBe('N');
    expect(getWindDirection(68)).toBe('NE');
  });
});
```

## 체크리스트

새로운 코어 로직 구현 전 확인사항:

- [ ] 테스트 파일이 먼저 생성되었는가?
- [ ] 실패하는 테스트를 먼저 작성했는가?
- [ ] 테스트가 실제로 실패하는 것을 확인했는가?
- [ ] 최소한의 코드로 테스트를 통과시켰는가?
- [ ] 리팩토링 후에도 테스트가 통과하는가?
- [ ] 엣지 케이스를 테스트했는가?
- [ ] 에러 케이스를 테스트했는가?

## 금지 사항

❌ **테스트 없이 코어 로직 구현**
```typescript
// 잘못된 예: 테스트 없이 바로 구현
export function calculateHumidity(temp: number, dewPoint: number) {
  // 구현...
}
```

❌ **구현 후 테스트 작성**
```typescript
// 잘못된 예: 구현이 먼저
// 1. 함수 구현
// 2. 테스트 작성 (X)
```

❌ **테스트를 통과시키기 위해 테스트 수정**
```typescript
// 잘못된 예: 실패하는 테스트를 수정
it('should return correct value', () => {
  // expect(result).toBe(100); // 원래 테스트
  expect(result).toBe(99); // 통과시키기 위해 수정 (X)
});
```

## 권장 사항

✅ **작은 단위로 테스트 작성**
```typescript
// 좋은 예: 하나의 기능만 테스트
it('should convert 0°C to 32°F', () => {
  expect(convertToFahrenheit(0)).toBe(32);
});

it('should convert 100°C to 212°F', () => {
  expect(convertToFahrenheit(100)).toBe(212);
});
```

✅ **의미 있는 테스트 이름**
```typescript
// 좋은 예
it('should return "N" for wind direction between 337.5° and 22.5°', () => {
  expect(getWindDirection(0)).toBe('N');
  expect(getWindDirection(350)).toBe('N');
  expect(getWindDirection(10)).toBe('N');
});
```

✅ **엣지 케이스 테스트**
```typescript
// 좋은 예: 경계값 테스트
describe('temperature validation', () => {
  it('should handle absolute zero', () => {
    expect(isValidTemperature(-273.15)).toBe(true);
  });

  it('should reject temperatures below absolute zero', () => {
    expect(isValidTemperature(-300)).toBe(false);
  });
});
```

## 참고 자료

- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Testing Library](https://testing-library.com/docs/)
- [TDD Best Practices](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
