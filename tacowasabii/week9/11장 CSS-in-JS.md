# 11장 CSS-in-JS

## 11.1 CSS-in-JS란

### 1. CSS-in-JS와 인라인 스타일의 차이

CSS-in-JS는 CSS-in-CSS보다 더 강력한 추상화 수준을 제공한다. CSS-in-JS를 활용하면 자바스크립트로 스타일을 선언적이고 유지보수할 수 있는 방식으로 표현할 수 있다.

- 인라인 스타일

```js
const textStyles = {
  color: "white",
  backgroundColor: "black",
};

const SomeComponent = () => {
  return <p style={textStyles}>inline style!</p>;
};
```

위 코드는 브라우저에서 DOM 노드를 다음과 같이 연결한다.

```js
<p style="color: white; background-color: black;">inline style!</p>
```

- CSS-in-JS

```js
import styled from "styled-components";

const Text = styled.div`
  color: white;
  background: black;
`;

// 다음처럼 사용
const Example = () => <Text>Hello CSS-in-JS</Text>;
```

위 코드는 브라우저에서 DOM 노드를 다음과 같이 연결한다.

```js
<style>
  .hash136s21 {
    background-color: black;
    color: white;
  }
</style>

<p class=”hash136s21”>Hello CSS-in-JS</p>
```

인라인 스타일은 DOM 노드에 속성으로 스타일을 추가한 반면에 CSS-in-Js는 DOM 상단에 `<style>` 태그를 추가했다.

- CSS-in-JS의 장점

1. 컴포넌트로 생각할 수 있다.

   CSS-in-JS는 스타일 컴포넌트 단위로 추상화하여 생각할 수 있게 해준다. 따라서 별도의 스타일시트를 유지보수할 필요 없이 각 컴포넌트의 스타일을 관리할 수 있다.

2. 부모와 분리할 수 있다.

   CSS에는 명시적으로 정의하지 않은 경우 부모 요소에서 자동으로 상속되는 속성이 있다. 하지만 CSS-in-JS는 이러한 상속을 받지 않는다. 따라서 각 컴포넌트의 스타일은 부모와 독립되어 독립적으로 동작한다.

3. 스코프를 가진다.

   CSS는 하나의 전역 네임스페이스를 가지기 때문에 선택자 충돌을 피하기 어렵다. CSS-in-JS는 CSS로 컴파일될 때 고유한 이름을 생성하여 스코프를 만들어준다. 따라서 선택자 충돌을 방지할 수 있다.

4. 자동으로 프리픽스가 붙는다.

   CSS-in-JS 라이브러리들은 자동으로 벤더 프리픽스를 추가하여 브라우저 호환성을 향상해준다.

5. 자바스크립트와 CSS 사이에 상수와 함수를 쉽게 공유할 수 있다.

   CSS-in-JS를 활용하면 자바스크립트 변수, 상수, 함수를 스타일 코드 내에서 쉽게 사용할 수 있다. 이를 통해 스타일과 관련된 로직을 함께 관리할 수 있다.

## 11.2 유틸리티 함수를 활용하여 `styled-components`의 중복 타입 선언 피하기

```ts
interface Props {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
  // ...
}

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  // ...
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

interface StyledProps {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
}

const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) => height || "10px"};
  margin: 0;
  background-color: ${({ color }) => colors[color || "gray7"]};
  border: none;

  ${({ isFull }) =>
    isFull &&
    css`
      margin: 0 -15px;
    `}
`;
```

`props`와 똑같은 타입임에도 `StyledProps`를 따로 정의해줘야 하는 점 때문에 코드 중복이 발생한다. 또한 `props`의 `height, color, isFull` 타입이 변경되면 `StyledProps`도 변경되어야 한다.

`props`에서 받은 타입을 `styled-components`로 넘겨서 활용할 때는 `Pick, Omit`같은 유틸리티 타입을 활용할 수 있다.

```ts
const HrComponent = styled.hr<Pick<Props, "height" | "color" | "isFull">>`
  // ...
`;
```
