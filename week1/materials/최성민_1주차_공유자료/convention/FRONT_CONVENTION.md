# Frontend Convention

이 문서는 `/front` 현재 구현을 기준으로 정리한 실사용 규칙입니다. 추상적인 권장사항보다, 지금 코드가 실제로 어떻게 나뉘어 있고 어떤 파일이 어떤 책임을 갖는지를 기준으로 작성합니다.

## 1. 현재 스택과 전제

- 런타임: React 19 + TypeScript + Vite
- 스타일링: Tailwind CSS v4 + `src/App.css` 토큰 정의 + 일부 전용 CSS 파일
- 라우팅: `react-router-dom`
- 서버 상태: `@tanstack/react-query`
- 전역 UI 상태: Zustand
- 공용 UI primitive: shadcn/ui + Radix UI + Lucide
- 경로 별칭: `@/*` 중심, 추가로 `@components/*`, `@config/*`, `@hooks/*`, `@pages/*`, `@routes/*`가 열려 있음
- 앱은 프론트/백 독립 배포를 전제로 두며, 기본 API 엔드포인트는 `VITE_API_URL || /api`입니다.

## 2. 구조

이 구조는 크게 7개 층으로 나뉩니다.

1. 부트스트랩: `src/main.tsx`, `src/App.tsx`
2. 앱 메타/정책: `src/config`, `src/routes`
3. 데이터 계층: `src/apis`, `src/queries`
4. 화면 단위: `src/pages`
5. 조합 컴포넌트: `src/components/layout`, `menu`, `card`, `grid`, `modal`, `loading`, `toast`
6. primitive/infra: `src/components/ui`, `src/lib`, `src/hooks`
7. 전역 상태/타입: `src/store`, `src/types`

## 3. 파일별 책임

### `src/main.tsx`

- React 루트 생성과 전역 Provider 조합의 시작점입니다.
- 현재 순서는 `ThemeProvider -> QueryClientProvider -> App`입니다.
- 새로운 전역 Provider는 여기서 추가합니다.
- 이 보일러플레이트는 개발 중 중복 effect 호출을 줄이기 위해 기본값으로 `StrictMode`를 두지 않습니다.
- 라우트, 레이아웃, 페이지 선택 로직은 두지 않습니다.

### `src/App.tsx`

- 브라우저 라우터, 중첩 라우트, 글로벌 UI 마운트 위치를 담당합니다.
- `Layout`은 여기서 한 번만 감쌉니다.
- `appRoutes`와 `fallbackRoute`를 펼쳐 `<Route>`로 연결합니다.
- `GlobalModal`, `GlobalLoading`, `GlobalToast`처럼 화면 전체에 걸쳐 떠야 하는 UI는 여기서 마운트합니다.
- 개별 페이지의 데이터 fetching, 헤더 텍스트, 메뉴 구성 로직은 넣지 않습니다.

### `src/config/site.ts`

- 사이트 이름, 짧은 이름, 설명, 현재 레이아웃 프리셋을 보관합니다.
- `layoutPresets`는 실제 `LayoutProps` 타입 일부를 재사용해 프리셋의 shape를 강제합니다.
- 레이아웃 기본값은 코드 여러 곳에 분산하지 말고 여기서 바꿉니다.
- 프로젝트 브랜딩을 교체할 때 가장 먼저 보는 파일입니다.

### `src/routes/route-config.tsx`

- 현재 앱의 라우트 소스 오브 트루스입니다.
- 페이지 컴포넌트는 `lazy(() => import(...))`로 선언하고 route element에 연결합니다.
- 각 항목은 `path`, `title`, `description`, `element`를 기본으로 가지며, 필요하면 `icon`, `showInNavigation`을 둡니다.
- SEO 메타가 필요하면 `seoTitle`, `seoDescription`, `seoKeywords`, `seoImage`, `noIndex`를 route definition에 함께 둡니다.
- `navigationItems`는 `appRoutes`에서 파생됩니다. 메뉴용 설정을 별도로 복제하지 않습니다.
- fallback route도 여기서 함께 관리합니다.

### `src/pages`

- 실제 URL 화면 단위를 둡니다.
- 현재 규칙은 `src/pages/<page>/index.tsx`입니다.
- `index.tsx`는 페이지 조합과 feature 시작점 역할을 맡습니다.
- 페이지가 커지면 같은 폴더 안에 `HeroSection.tsx`, `StartStepsSection.tsx` 같은 페이지 전용 섹션 컴포넌트를 분리하고 `index.tsx`는 조합만 담당합니다.
- 페이지와 섹션 컴포넌트는 `queries` 훅을 사용할 수 있지만 `apis`를 직접 import하지 않습니다.
- 공통 shell, 공통 네비게이션, 전역 provider 책임은 가져오지 않습니다.

### `src/apis`

- 도메인별 순수 API 호출 함수 계층입니다.
- 권장 구조는 `src/apis/<domain>/...` 입니다.
- 이 계층만 `axiosInstance` 같은 HTTP 클라이언트를 직접 압니다.
- 페이지, 섹션 컴포넌트, `components/ui`, `store`는 이 계층을 직접 호출하지 않게 둡니다.

### `src/queries`

- React Query 훅과 query key 계층입니다.
- 권장 구조는 `src/queries/<domain>/...` 입니다.
- 내부에서 `apis`를 호출하고, 외부에는 `useQuery`/`useMutation` 훅을 노출합니다.
- 페이지와 섹션 컴포넌트는 가능하면 이 계층만 직접 소비합니다.

### `src/components/layout`

- 앱 전체 shell을 담당합니다.
- `Layout.tsx`는 `mode`, `frame`, `contentWidth`, `showHeader`, `showSidebar`, `showFooter`로 shell 구성을 제어합니다.
- lazy route는 `Layout` 내부 `Outlet` 근처의 `Suspense` 경계에서 처리해 header/sidebar/footer shell을 유지합니다.
- `layout.css`는 Tailwind 유틸리티만으로 다루기 애매한 레이아웃 프레임, sticky header, content width, scroll mode를 담당합니다.
- header/footer 하위 컴포넌트는 shell 내부 조합만 맡습니다.

### `src/components/menu`

- 라우트 메타데이터를 화면 네비게이션으로 바꾸는 계층입니다.
- `Menu.data.ts`는 `navigationItems`를 `menuConfig`로 매핑합니다.
- `Menu.types.ts`는 라우트와 메뉴 사이의 최소 공용 인터페이스입니다.
- `left/`는 좌측 사이드바 표현 계층입니다.

### `src/components/ui`

- shadcn/ui 기반 primitive 영역입니다.
- `button.tsx`, `dialog.tsx`, `sheet.tsx`, `sidebar.tsx`처럼 재사용 단위가 모여 있습니다.
- 이 폴더는 생성기(shadcn)에서 들어온 파일과 그에 준하는 primitive 확장 파일이 섞여 있습니다.
- 제품 도메인 규칙을 넣는 곳이 아닙니다.

### `src/lib`

- 공용 인프라 싱글턴과 유틸리티를 둡니다.
- `axios.ts`: API 클라이언트 기본 설정
- `queryClient.ts`: React Query 전역 기본 정책
- `utils.ts`: `cn()` 같은 범용 헬퍼

### `src/store`

- 서버 상태가 아닌 전역 UI 상태만 둡니다.
- 현재는 `loadingStore`, `modalStore`만 있습니다.
- 전역 modal/loading처럼 여러 화면에서 동일하게 열고 닫아야 하는 UI 상태에 적합합니다.
- API 결과 캐시나 페이지별 form 입력값을 무조건 store로 올리지 않습니다.

### `src/hooks`

- primitive나 infra가 공통으로 쓰는 훅을 둡니다.
- 현재 `use-mobile.ts`는 `sidebar.tsx`가 의존하는 브레이크포인트 유틸입니다.
- 페이지 전용 훅은 먼저 페이지/feature 근처에 두는 것을 우선 검토합니다.

### `src/types`

- ambient declaration 같은 전역 타입 선언만 둡니다.
- 현재는 `global.d.ts`에서 JSON module 선언을 다룹니다.

## 4. 의존 방향

현재 구조의 바람직한 의존 방향은 아래입니다.

```text
main/App
  -> config/routes
  -> apis/queries
  -> layout/menu/pages
  -> lib/store/components
  -> ui/hooks/types
```

상세 규칙:

- `main.tsx`는 provider와 `App`만 조립합니다.
- `App.tsx`는 라우트와 글로벌 오버레이만 조립합니다.
- `apis`는 네트워크 호출만 담당합니다.
- `queries`는 `apis`를 감싸는 서버 상태 훅 계층입니다.
- `routes`는 `pages`를 참조할 수 있지만, 반대로 페이지가 라우트 배열을 수정하려고 의존을 거꾸로 만들지 않습니다.
- `menu`는 `routes`에서 파생됩니다.
- `layout`은 `menuConfig`와 shell props를 소비합니다.
- `pages`는 `components`, `queries`, `lib`, `config`를 읽을 수 있지만, shell 제어 책임이나 직접 API 호출 책임까지 가져가지 않습니다.
- `components/ui`는 가능한 한 가장 아래층에 두고, domain-specific 문맥을 넣지 않습니다.

## 5. 라우팅과 메뉴 규칙

현재 코드에서 가장 중요한 구조 규칙은 “라우트에서 메뉴를 파생한다”는 점입니다.

### 필수 규칙

- 새 페이지를 추가할 때는 먼저 `src/pages/<name>/index.tsx`를 만듭니다.
- 그다음 `src/routes/route-config.tsx`에 route definition을 추가합니다.
- route page import는 direct import 대신 `lazy(() => import(...))`를 기본으로 둡니다.
- 헤더/사이드바에 노출하려면 `showInNavigation: true`를 켭니다.
- 메뉴에 들어갈 라벨/설명/아이콘은 route definition에 둡니다.
- SEO 정보가 필요하면 route definition에 같이 둡니다.
- 메뉴 전용 별도 배열을 만들지 않습니다.

### 이유

- 지금 구조는 route metadata와 navigation metadata가 하나의 source로 합쳐져 있습니다.
- SEO metadata도 같은 route object에 같이 두면 페이지 식별 정보가 한 군데에 모입니다.
- 이 흐름을 깨면 route title, 메뉴 제목, 설명, 아이콘이 쉽게 어긋납니다.
- `Menu.data.ts`는 route 구조를 menu 구조로 변환하는 얇은 adapter여야 합니다.

### 금지

- `MenuLeft`, `HeaderNav` 안에서 직접 하드코딩한 메뉴 배열 만들기
- 페이지 추가 후 `route-config.tsx`를 빼먹고 링크만 수동으로 다는 방식
- 메뉴용 title과 페이지 title을 다른 파일에서 중복 정의하는 방식

## 6. 레이아웃 규칙

레이아웃은 `siteConfig.layoutPreset -> activeLayoutPreset -> Layout props` 순서로 전달됩니다.

### 현재 프리셋

- `web`: 사이드바 + 헤더 + 푸터
- `mobile`: 좁은 frame + sidebar 없음 + footer 없음 + mobile scroll mode
- `landing`: sidebar 없음 + content 중심

### 규칙

- 시작 레이아웃은 `src/config/site.ts`에서 선택합니다.
- 새 화면을 만든다고 해서 페이지 내부에서 헤더/사이드바를 직접 복제하지 않습니다.
- shell 동작 변경은 우선 `LayoutProps`와 `layoutPresets` 조합으로 해결합니다.
- 반복되는 layout width 규칙은 page에서 클래스 하드코딩하지 말고 `contentWidth` 또는 `layout.css`로 흡수합니다.
- `Layout.tsx`의 `data-scroll-mode`, `data-frame`, `data-content-width`는 shell 동작의 기준점입니다. 페이지에서 이 계약을 우회하지 않습니다.

### CSS 배치 원칙

- 전역 토큰과 base reset: `src/App.css`
- shell 구조 CSS: `src/components/layout/layout.css`
- 일반 컴포넌트 표현은 가급적 Tailwind 유틸리티 클래스 안에서 해결
- CSS 파일은 layout/frame/scroll처럼 구조적 이유가 분명할 때만 추가

## 7. 컴포넌트 분류 규칙

### `components/ui`

- 재사용 가능한 primitive
- shadcn generator 결과물 또는 그에 가까운 기초 컴포넌트
- 도메인 이름 금지
- `Button`, `Dialog`, `Sidebar*`처럼 범용 API를 유지

### `components/layout`

- 앱 shell에 속한 조합 컴포넌트
- `Header`, `Footer`, `Layout`, `HeaderNav`, `HeaderLogo`
- route/menu/siteConfig를 조합해 shell을 만드는 계층

### `components/menu`

- navigation 전용 조합 컴포넌트
- route metadata를 시각 메뉴 구조로 바꾸는 역할
- active path 계산, collapse 처리, 메뉴 tooltip 처리

### `components/card`, `components/grid`

- 특정 도메인에 묶이지 않는 디자인 블록
- 페이지 여러 곳에서 반복 사용 가능한 중립 컴포넌트
- `SurfaceCard`처럼 이름만 봐도 역할이 드러나야 합니다.

### `components/modal`, `components/loading`, `components/toast`

- 전역 오버레이 진입점
- 상태/primitive를 조합해 실제 전역 UI를 마운트하는 thin wrapper
- App 레벨에서 한 번만 렌더링합니다.

## 8. 코드 스타일 규칙

### 들여쓰기와 포맷

- 앱 소유 코드(`src/pages`, `src/config`, `src/routes`, `src/store`, `src/lib`, `src/components/layout|menu|card|grid|modal|loading|toast`)는 현재 4-space + 세미콜론 스타일이 기본입니다.
- `src/components/ui`와 `src/hooks/use-mobile.ts`는 shadcn 생성 코드 스타일이 섞여 있어 2-space, 세미콜론 생략, double quote가 일부 존재합니다.
- 생성기에서 들어온 파일을 대규모로 재포맷하지 않습니다.
- 직접 작성하는 앱 코드에 생성기 스타일을 확산시키지 않습니다.

즉, 현재 저장소의 실전 규칙은 아래처럼 이해합니다.

- 앱 코드: 기존 4-space 스타일 유지
- 생성 코드: 생성기 스타일 유지 가능
- 둘을 섞은 파일에서는 먼저 그 파일의 기존 스타일을 따름

### import 규칙

- 외부 패키지 import를 먼저 둡니다.
- 내부 import는 alias import를 우선합니다.
- 같은 폴더 또는 매우 가까운 하위 폴더만 `./`, `../`를 허용합니다.
- boot 파일(`main.tsx`, `App.tsx`)처럼 루트 근처 조립 파일은 상대 경로를 써도 됩니다.
- 타입 전용 import는 `import type`을 사용합니다.

현재 코드 기준 정리:

- cross-domain import: `@/config/site`, `@/components/ui/button`
- local composition import: `./HeaderLogo`, `./Footer`, `../menu/left`

### export 규칙

- 화면/조합 컴포넌트 1개가 주인공인 파일은 `default export`를 사용합니다.
- 공유 상수, store hook, config object, provider helper, utility function은 named export를 사용합니다.
- `index.tsx`는 얇은 re-export 용도로만 사용합니다.

실제 패턴:

- default export: `HomePage`, `Layout`, `Header`, `Footer`, `SurfaceCard`, `Grid`
- named export: `queryClient`, `axiosInstance`, `appRoutes`, `navigationItems`, `menuConfig`, `useModalStore`

### 타입 선언 규칙

- 컴포넌트 props는 파일 상단에 `interface`로 두는 패턴이 기본입니다.
- 재사용되는 union이나 configuration type은 `type` alias를 사용합니다.
- `site.ts`처럼 기존 타입 일부를 재사용할 때는 `Pick`, `Required`, `satisfies`를 적극적으로 사용합니다.
- route/menu/store처럼 shared contract는 파일 안에 명시적으로 선언하고 암묵적 객체 shape에 기대지 않습니다.

### React 컴포넌트 규칙

- 함수형 컴포넌트를 사용합니다.
- `React.FC`는 사용하지 않습니다.
- children이 필요하면 `ReactNode`를 명시합니다.
- 조건부 렌더링은 early return 또는 JSX 분기 모두 허용하지만, 상태 분기가 복잡해지면 wrapper 컴포넌트로 분리합니다.
- 페이지는 조합 중심, primitive는 API 중심으로 작성합니다.

### className 조합 규칙

- 조건부 클래스 병합은 `cn()` 또는 `clsx`를 사용합니다.
- 이미 `cn()`이 있는 영역에서는 `cn()`을 우선합니다.
- shadcn primitive 계열은 `cn()`이 사실상 기본입니다.
- 단순 style 계산이 필요한 grid 같은 경우에만 `style={{ ... }}`를 허용합니다.

### 아이콘 규칙

- `lucide-react`를 기본 아이콘 소스로 사용합니다.
- 라우트/메뉴 아이콘은 컴포넌트 인스턴스가 아니라 아이콘 타입 자체를 metadata에 저장합니다.
- 렌더링 계층에서 `<item.icon />` 형태로 출력합니다.

## 9. 스타일링 규칙

### 토큰

- 브랜드/테마 토큰은 `src/App.css`의 `:root`, `.dark`, `@theme inline`에서 관리합니다.
- 컴포넌트에서 임의 HEX 색상을 계속 늘리지 않습니다.
- 가능한 한 `bg-background`, `text-muted-foreground`, `border-border`, `bg-primary` 같은 semantic token 클래스를 사용합니다.

### 유틸리티 우선

- 대부분의 표현은 Tailwind 클래스에서 해결합니다.
- radius, surface, spacing, responsive layout도 utility class를 우선합니다.
- 단, `layout.css`처럼 data attribute 기반 shell 동작은 CSS 파일이 더 명확하므로 예외입니다.

### inline style 허용 범위

- `Grid`처럼 계산식 기반 레이아웃
- CSS custom property 전달
- third-party 컴포넌트 style contract

그 외에는 inline style보다 token/class 기반 접근을 우선합니다.

## 10. 상태와 데이터 규칙

### 서버 상태

- 서버 데이터는 React Query를 기본값으로 둡니다.
- query 기본 정책은 `src/lib/queryClient.ts`에 모읍니다.
- query/mutation 재시도 정책을 컴포넌트마다 제각각 복제하지 않습니다.
- UI 계층은 가능하면 `src/queries/<domain>`의 훅만 직접 사용합니다.

### API 클라이언트

- 공통 base URL, timeout, headers, interceptor는 `src/lib/axios.ts`에 둡니다.
- 화면에서 직접 `axios.create()`를 반복하지 않습니다.
- 새 API helper는 `src/apis/<domain>` 아래에 두고 `axiosInstance`를 감싼 함수로 작성합니다.
- 페이지/섹션/일반 컴포넌트에서 `axiosInstance`, `fetch`, `apis` 직접 호출을 섞지 않습니다.

### Query 계층

- `src/queries/<domain>`에는 `useQuery`, `useMutation`, query key를 둡니다.
- 이 계층이 `src/apis/<domain>`를 호출합니다.
- 페이지와 섹션 컴포넌트는 가능하면 `queries` 훅만 import합니다.

### 전역 UI 상태

- modal/loading처럼 앱 어디서나 열어야 하는 UI만 Zustand store로 올립니다.
- store 이름은 `useXxxStore` 패턴을 따릅니다.
- store 파일은 상태 shape와 action을 한 파일에 둡니다.
- store는 UI state를 저장하고, 렌더링은 전역 컴포넌트(`GlobalModal`, `GlobalLoading`)가 맡습니다.

### 현재 store에서 읽히는 규칙

- `loadingStore`: boolean + 단순 action
- `modalStore`: title/content/close delay 정리까지 포함
- 상태 초기화와 cleanup은 store 내부에서 해결하고, 호출부가 cleanup 타이밍을 매번 신경 쓰지 않게 합니다.

## 11. 생성 코드와 앱 코드의 경계

이 프로젝트는 shadcn 기반 파일과 앱 코드가 같이 있기 때문에, 이 경계를 명확히 지켜야 합니다.

### `components/ui` 수정 시

- 먼저 primitive 수준 변경인지 확인합니다.
- 도메인 텍스트, 특정 서비스 정책, 라우트 의존성은 넣지 않습니다.
- 여러 화면에 영향을 주는 토큰/variant/API 변경인지 먼저 판단합니다.
- 생성기 업데이트 가능성을 고려해 불필요한 재작성은 피합니다.

### 앱 코드 수정 시

- `components/ui`를 건드리지 않고 해결 가능한지 먼저 봅니다.
- 특정 화면 요구사항은 wrapper 또는 composition layer에서 해결합니다.
- 예를 들어 메뉴 tooltip 문구, active path 계산, 레이아웃 노출 여부는 `menu`, `layout`, `routes`에서 처리합니다.

## 12. 새 페이지 추가 규칙

권장 순서는 아래입니다.

1. `src/pages/<page>/index.tsx` 생성
2. `src/routes/route-config.tsx`에 route 등록
3. 내비게이션 노출이 필요하면 `showInNavigation: true` 추가
4. 페이지가 shell 기본값과 다르면 `siteConfig.layoutPreset` 또는 `Layout` 계약을 재검토
5. 공통 블록이 보이면 `components`로 추출

페이지 내부 규칙:

- 페이지는 공통 header/footer/sidebar를 직접 만들지 않습니다.
- `index.tsx`는 컨텐츠 조합과 feature entry에 집중합니다.
- 해당 페이지에서만 쓰는 섹션 컴포넌트는 같은 페이지 폴더에 co-location 합니다.
- 페이지 전용 섹션은 공용 `components`로 성급히 올리지 않습니다.
- 페이지와 섹션은 `queries` 훅을 호출할 수 있지만 `apis`를 직접 import하지 않습니다.
- 반복 UI가 생기면 페이지 안에서 계속 복붙하지 말고 `components` 또는 feature 하위로 뽑습니다.

## 13. 새 컴포넌트 추가 기준

아래 질문으로 위치를 결정합니다.

### `src/components/ui`

- 범용 primitive인가?
- 서비스 문맥 없이도 재사용 가능한가?
- variant/size/className API가 중심인가?

### `src/components/layout`

- 앱 shell 일부인가?
- header/footer/frame/sidebar 노출 정책과 연결되는가?

### `src/components/menu`

- navigation 표현/active state/collapse와 직접 연결되는가?

### `src/components/<custom>`

- 두 개 이상의 페이지에서 재사용 가능한 조합 블록인가?
- 도메인 중립 이름을 붙일 수 있는가?

### 페이지 내부 또는 향후 feature 폴더

- 특정 화면에만 강하게 묶인 구성인가?
- 아직 공용화 근거가 약한가?

## 14. 주의할 점

### 현재 코드에서 읽히는 암묵 규칙

- starter 안내 텍스트는 최대한 중립적으로 유지합니다.
- 실제 프로젝트 시작 시 `siteConfig`, 홈 샘플 문구, footer 문구를 먼저 교체합니다.
- 아직 frontend 테스트 러너/스크립트는 잡혀 있지 않습니다. 테스트가 필요한 로직을 넣기 시작하면 테스트 도구부터 명시적으로 도입해야 합니다.
- `route-config -> menuConfig -> Layout/Header/Menu` 흐름은 이 starter의 핵심 연결선입니다.

### 하지 않는 것

- 메뉴 구조를 별도 하드코딩하지 않는다.
- 페이지 안에서 공통 shell을 재구현하지 않는다.
- 전역 UI 상태를 페이지 local state처럼 여기저기 흩뿌리지 않는다.
- `components/ui`에 제품 도메인 로직을 넣지 않는다.
- 미래를 위해 빈 폴더나 빈 모듈을 미리 만들지 않는다.
- 생성 파일 전체를 앱 스타일로 일괄 재포맷하지 않는다.

## 15. 빠른 체크리스트

새 코드를 추가할 때 아래를 확인합니다.

- 이 파일은 어느 계층에 속하는가?
- 메뉴 정보가 route metadata와 중복되고 있지 않은가?
- shell 변경을 page 내부에서 우회하고 있지 않은가?
- 서버 상태와 UI 상태를 올바른 계층에 두고 있는가?
- 공통 primitive를 수정해야 하는지, 조합 계층에서 해결해야 하는지 구분했는가?
- 이 파일의 포맷 스타일은 그 파일이 속한 계층의 기존 스타일과 맞는가?
