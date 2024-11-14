@types/react 내장타입을 활용하여 리액트 컴포넌트들의 타입을 작성해보자

함수형 컴포넌트의 반환 타입 `FC`

```tsx
type FC<P = {}> = FunctionComponent<P>;

// React 18에서는 children이 사라져서 직접 지정해주어야한다.
interface FunctionComponent<P = {}> {
  (props: P, context?: any): ReactNode;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

children의 props타입 지정할때 가장 보편적으로 쓰이는 타입은 `ReactNode | undefined` 형태이다.

```tsx
type PropsWithChildren<P = unknown> = P & { children?: ReactNode | undefined };
```

ReactNode << ReactElement << JSX.Element 와 같은 포함관계를 가지며

ReactElement는 React.createElement의 반환 타입으로 리액트에서 가상 DOM을 저장하는 형태가 ReactElement이며 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타낸다.

JSX.Element는 ReactElement를 확장한 형태로 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성을 준다.

```tsx
type ReactNode =
  | ReactElement
  | string
  | number
  | Iterable<ReactNode>
  | ReactPortal
  | boolean
  | null
  | undefined
  | DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES];
```

```tsx
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

```tsx
declare global {
    /**
     * @deprecated Use `React.JSX` instead of the global `JSX` namespace.
     */
    namespace JSX {
        // We don't just alias React.ElementType because React.ElementType
        // historically does more than we need it to.
        // E.g. it also contains .propTypes and so TS also verifies the declared
        // props type does match the declared .propTypes.
        // But if libraries declared their .propTypes but not props type,
        // or they mismatch, you won't be able to use the class component
        // as a JSX.ElementType.
        // We could fix this everywhere but we're ultimately not interested in
        // .propTypes assignability so we might as well drop it entirely here to
        //  reduce the work of the type-checker.
        // TODO: Check impact of making React.ElementType<P = any> = React.JSXElementConstructor<P>
        type ElementType = string | React.JSXElementConstructor<any>;
        interface Element extends React.ReactElement<any, any> { } // 여기서 확인
        interface ElementClass extends React.Component<any> {
            render(): React.ReactNode;
        }
...
```

합성 컴포넌트를 만들때는 아래와 같이 모든 리액트 컴포넌트를 children으로 받을 수 있게한다.

```tsx
interface MyComponentsProps {
  childeren?: React.ReactNode;
}
```

JSX.Element 타입으로 선언하면 해당 props에 JSX 문법만 삽입할 수 있다.

ReactElement로 좁히면 넣어준 props를 추론할 수 있다.

```tsx
interface IocnProps {
	size: number;
}

interface Props {
	icon: React.ReactElement<IconProps>
}

const item = ({icon}:Props) => {
	const iconSize = icon.props.size; // 추론 가능!
	...
}
```

DetailedHTMLProps vs ComponentWithoutRef

HTML 태그의 속성을 활용하기 위한 타입들로 ref를 props로 전달할때 함수 컴포넌트의 경우 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않는다. 함수 컴포넌트에서는 생성된 인스턴트가 없기 때문이다. 따라서 React.forwardRef 메서드를 통해 ref를 전달받을 수 있고 이러한 경우 ComponentWithoutRef 타입을 사용하여 ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전하다.

```tsx
type NaitiveButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type NaitiveButtonProps = React.ComponentPropsWithoutRef<"button">;

type ButtonProps = {
  onClick?: NaitiveButtonProps["onClick"]; // 실제 HTML button 태그의 onclick 이벤트 핸들러 타입과 동일
};
```

```tsx
// ref를 전달할때는 ComponentPropsWithoutRef를 사용하는것이 안전하다
type InputType = React.ComponentPropsWithoutRef<"input">;
const Input = forwardRef<HTMLInputElement, InputType>((props, ref) => {
  return <input ref={ref} {...props} />;
});
```

Select 컴포넌트와 같이 전달받은 options에 대해서만 onSelect, defaultOption 과 같은 props에 대해 작동하게 하고 싶을때 제너릭 컴포넌트를 활용하자

```tsx
interface SlectProps<Option extends Record<string, string>> {
  options: Option;
  selectOption: keyof Option;
  onChange: (selected: keyof Option) => void;
}
```

위와 같이 작성하면 options props에 전달받은 Option 을 기반으로 타입을 추론하여 나머지 props에서도 안전하게 타입 사용이 가능하다.
