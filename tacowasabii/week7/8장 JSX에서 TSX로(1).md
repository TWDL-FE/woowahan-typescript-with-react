# 8장 JSX에서 TSX로

## 9.1 리액트 컴포넌트 타입

### 1. 클래스 컴포넌트 타입

```ts
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
  /* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

`P`와 `S`는 각각 `props`와 상태를 의미한다. `Welcome` 컴포넌트의 `props`타입을 지정해보면 아래와 같다. 상태가 있는 컴포넌트일 때는 제네릭의 두 번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정할 수 있다.

```ts
interface WelcomeProps {
  name: string;
}

class Welcome extends React.Component<WelcomeProps> {
  /* ... 생략 */
}
```

### 2. 함수 컴포넌트 타입

```ts
// 함수 선언을 사용한 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식을 사용한 방식 - React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - React.VFC 사용
// Void Function Component, children prop을 포함하지 않음
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - JSX.Element를 반환 타입으로 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {};

type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  // props에 children을 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  // children 없음
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

리액트 v18로 넘어오면서 `React.VFC`가 삭제되고 `React.FC`에서 `children`이 사라졌다. 그래서 앞으로는 `React.VFC` 대신 `React.FC` 또는 `props` 타입 / 반환 타입을 직접 지정하는 형태로 타이핑 해줘야 한다.

```ts
import React from "react";

interface WelcomeProps {
  name: string;
  children?: React.ReactNode; // children을 명시적으로 정의
}

const Welcome: React.FC<WelcomeProps> = ({ name, children }) => {
  return (
    <div>
      <h1>Welcome, {name}!</h1>
      {children}
    </div>
  );
};

export default Welcome;
```

### 3. `Children props` 타입 지정

```ts
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

가장 보편적인 `children` 타입은 `ReactNode | undefined`가 된다. `ReactNode`는 `ReactElement` 외에도 `boolean, number` 등 여러 타입을 포함하고 있는 타입으로, 더 구체적인 타이핑하는 용도에는 적합하지 않다.

```ts
// example 1
type WelcomeProps = {
  children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분";
};

// example 2
type WelcomeProps = { children: string };

// example 3
type WelcomeProps = { children: ReactElement };
```

### 4. `render` 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs JSX.Element vs React.ReactNode

함수 컴포넌트의 반환 타입인 `ReactElement`는 아래와 같이 정의된다.

```ts
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

`React.createElement`를 호출하는 형태의 구문으로 변환하면 `React.createElement`의 반환 타입은 `ReactElement`이다. 리액트는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 가상 DOM의 엘리먼트는 `ReactElement` 형태로 저장된다. 즉, `ReactElement` 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷이다.

```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

`JSX.Element` 타입은 리액트의 `ReactElement`를 확장하고 있는 타입이며, 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성을 제공한다. 이러한 특성으로 인해 컴포넌트 타입을 재정의하거나 변경하는 것이 용이해진다.

`React.Node`는 다음과 같이 타입이 정의되어 있다.

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

- 포함관계
  ReactNode > ReactElement > JSX.Element

### 5. `ReactElement, ReactNode, JSX.Element` 활용하기

#### ReactElement

JSX는 리액트 엘리먼트를 생성하기 위한 문법이며 트랜스파일러는 JSX 문법을 `createElement` 메서드 호출문으로 변환하여 아래와 같이 리액트 엘리먼트를 생성한다.

```ts
const element = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);

// 주의: 다음 구조는 단순화되었다
const element = {
  type: "h1",
  props: {
    className: "greeting",
    children: "Hello, world!",
  },
};

declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

리액트는 이런 식으로 만들어진 리액트 엘리먼트 객체를 읽어서 DOM을 구성한다. 리액트에는 여러 개의 `createElement` 오버라이딩 메서드가 존재하는데, 이 메서드들이 반환하는 타입은 `ReactElement` 타입을 기반으로 한다.

정리하면 `ReactElement` 타입은 JSX의 `createElement` 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입이라고 볼 수 있다.

#### ReactNode

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
```

`ReactChild` 타입은 `ReactElement | string | number`로 정의되어 `ReactElement`보다는 좀 더 넓은 범위를 갖고 있다.

```ts
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

`ReactNode`는 리액트의 `render` 함수가 반환할 수 있는 모든 형태를 담고 있다고 볼 수 있다.

#### JSX.Element

```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

`JSX.Element`는 `ReactElement`의 제네릭으로 `props`와 타입 필드에 대해 `any` 타입을 가지도록 확장하고 있다. 즉, `JSX.Element`는 `ReactElement`의 특정 타입으로 `props`와 타입 필드를 `any`로 가지는 타입이라는 것을 알 수 있다.

### 6. 사용 예시

#### ReactNode

JSX 형태의 문법을 어떤 타입이든 `children prop`으로 지정할 수 있게 하고 싶다면 `ReactNode` 타입으로 `children` 을 선언하면 된다.

```ts
type PropsWithChildren<P = unknown> = P & {
  children?: ReactNode | undefined;
};

interface MyProps {
  // ...
}

type MyComponentProps = PropsWithChildren<MyProps>;
```

이런 식으로 `ReactNode`는 `prop`으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고 싶을 때 유용하게 사용된다.

#### JSX.Element

`JSX.Element`는 앞서 언급한대로 `props`와 타입 필드가 `any` 타입인 리액트 엘리먼트를 나타낸다. 이러한 특성 때문에 리액트 엘리먼트를 `prop`으로 전달받아 `render props` 패턴으로 컴포넌트를 구현할 때 유용하게 활용할 수 있다.

```ts
interface Props {
  icon: JSX.Element;
}

const Item = ({ icon }: Props) => {
  // prop으로 받은 컴포넌트의 props에 접근할 수 있다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};

// icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
const App = () => {
  return <Item icon={<Icon size={14} />} />;
};
```

#### ReactElement

앞서 살펴본 `JSX.Element` 예시를 확장하여 추론 관점에서 더 유용하게 활용할 수 있는 방법은 `JSX.Element` 대신에 `ReactElement`를 사용하는 것이다. 이때 원하는 컴포넌트의 `props`를 `ReactElement`의 제네릭으로 지정해줄 수 있다.

```ts
interface IconProps {
  size: number;
}

interface Props {
  // ReactElement의 props 타입으로 IconProps 타입 지정
  icon: React.ReactElement<IconProps>;
}

const Item = ({ icon }: Props) => {
  // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};
```

### 7. 리액트에서 기본 HTML 요소 타입 활용하기

#### `DetailedHTMLProps`와 `ComponentWithoutRef`

HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 `DetailedHTMLProps`와 `ComponentWithoutRef` 타입을 활용하는 것이다.

먼저 `React.DetailedHTMLProps`를 활용하는 경우에는 아래와 같이 HTML 태그 속성과 호환되는 타입을 선언할 수 있다.

```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

첫 번째 제네릭 인자로 속성 타입(attribute type)을 받는다. 두 번째 제네릭 인자로는 HTML 요소의 타입을 받는다.

그리고 `React.ComponentWithoutRef` 타입은 아래와 같이 활용할 수 있다.

```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

1. `React.ButtonHTMLAttributes<HTMLButtonElement>`만 사용하는 경우:

```ts
interface CustomButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {}

const CustomButton = (props: CustomButtonProps) => {
  return <button {...props}>Click Me</button>;
};

// 사용 예시
<CustomButton
  onClick={() => alert("Clicked!")}
  disabled={false}
  type="submit"
/>;
```

2. `React.DetailedHTMLProps`를 사용하는 경우:

```ts
interface CustomButtonProps
  extends React.DetailedHTMLProps<
    React.ButtonHTMLAttributes<HTMLButtonElement>,
    HTMLButtonElement
  > {}

const CustomButton = React.forwardRef<HTMLButtonElement, CustomButtonProps>(
  (props, ref) => {
    return (
      <button ref={ref} {...props}>
        Click Me
      </button>
    );
  }
);

// 사용 예시
const buttonRef = React.createRef<HTMLButtonElement>();

<CustomButton
  ref={buttonRef}
  onClick={() => alert("Clicked!")}
  disabled={false}
  type="submit"
/>;
```

#### 언제 `ComponentPropsWithoutRef`를 사용하면 좋을까

```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```

여기까지 보면 `HTMLButtonElement`의 속성을 모두 `props`로 받아 `button` 태그에 전달했으므로 문제없어 보인다. 그러나 `ref`를 `props`로 받을 경우 고려해야 할 사항이 있다.

클래스 컴포넌트와 함수 컴포넌트에서 `ref`를 `props`로 받아 전달하는 방식에 차이가 있다.

```ts
// 클래스 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
class WrappedButton extends React.Component {
  constructor() {
    this.buttonRef = React.createRef();
  }

  render() {
    return (
      <div>
        <Button ref={this.buttonRef} />
      </div>
    );
  }
}
```

클래스 컴포넌트로 만들어진 버튼은 컴포넌트 `props`로 전달된 `ref`가 `Button` 컴포넌트의 `button` 태그를 그대로 바라보게 된다.

```ts
// 함수 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  );
};
```

하지만 함수 컴포넌트이 경우 전달받은 `ref`가 `Button` 컴포넌트의 `button` 태그를 바라보지 않는다. 클래스 컴포넌트에서 `ref` 객체는 마운트된 컴포넌트의 인스턴스를 `current` 속성값으로 가지지만, 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 `ref`에 기대한 값이 할당되지 않는 것이다.

이러한 문제를 해결하기 위해 `React.forwardRef` 메서드를 사용한다.
`forwardRef`는 2개의 제네릭 인자를 받을 수 있는데, 첫 번째는 `ref`에 대한 타입 정보이며 두번째는 `props`에 대한 타입 정보이다.

```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

앞의 코드를 보면 `Button` 컴포넌트의 `props`에 대한 타입인 `NativeButtonType`을 정의할 때 `ComponentPropsWithoutRef` 타입을 사용한 것을 알 수 있다. 이렇게 타입을 `React.ComponentPropsWithoutRef<"button">`로 작성하면 `button` 태그에 대한 HTML 속성을 모두 포함하지만, `ref` 속성은 제외된다. 이러한 특징 때문에 `DetailedHTMLProps, HTMLProps, ComponentPropsWithRef`와 같이 `ref` 속성을 포함하는 타입과는 다르다.

함수 컴포넌트의 `props`로 `DetailedHTMLProps`와 같이 `ref`를 포함하는 타입을 사용하게 되면, 실제로는 동작하지 않는 `ref`를 받도록 타입이 지정되어 예기치 않은 에러가 발생할 수 있다. 따라서 HTML 속성을 확장하는 `props`를 설계할 때는 `ComponentPropsWithoutRef` 타입을 사용하여 `ref`가 실제로 `forwardRef`와 함께 사용될 때만 `props`로 전달되도록 타입을 정의하는 것이 안전하다.
