# 8장 JSX에서 TSX로

## 8.2 타입스크립트로 리액트 컴포넌트 만들기

### 1. JSX로 구현된 `Select` 컴포넌트

아래는 JSX로 구현된 `Select` 컴포넌트이다. 이 컴포넌트는 각 option의 키-값 쌍을 객체로 받고 있으며, 선택된 값이 변경될 때 호출되는 `onChange` 이벤트 핸들러를 받도록 구현되어 있다.

```ts
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

컴포넌트를 사용하는 개발자가 각 속성에 어떤 타입의 값을 전달해야 할지 명확히 알려줄 수 있도록 추가적인 설명이 필요하다.

### 2. JSDocs로 일부 타입 지정하기

JSDocs를 활용하면 컴포넌트에 대한 설명과 각 속성이 어떤 역할을 하는지 간단하게 알려줄 수 있다.

```ts
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
* @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
* @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
* @returns {JSX.Element}
*/
const Select = //...
```

### 3. `props` 인터페이스 적용하기

JSDocs를 활용하면 각 속성의 대략적인 타입과 어떤 역할을 하는지 파악할 수 있지만, `options`가 어떤 형식의 객체를 나타내는지나 `onChange`의 매개변수 및 반환 값에 대한 구체적인 정보를 알기 쉽지 않아서 잘못된 타입이 전달될 수 있는 위험이 존재한다.

이러한 문제를 해결하기 타입스크립트를 사용하여 좀 더 정교하고 구체적인 타입을 지정할 수 있다.

```ts
type Option = Record<string, string>; // {[key: string]: string}

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  //...
};
```

### 4. 리액트 이벤트

리액트는 가상 DOM을 다루면서 이벤트도 별도로 관리한다. `onclick, onchange`와 같이 DOM 엘리먼트에 등록되어 처리하는 이벤트 리스터와 달리, 리액트 컴포넌트(노드)에 등록되는 이벤트 리스너는 `onClick, onChange`처럼 카멜 케이스로 표기한다. 따라서 리액트 이벤트는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지는 않는다.

예를 들어 리액트 이벤트 핸들러는 이벤트 버블링 단계에서 호출된다. 이벤트 캡처 단계에서 이벤트 핸들러를 등록하기 위해서는 `onClickCapture, onChangeCapture`와 같이 일반 이벤트 리스너 이름 뒤에 `Capture`를 붙여야 한다.

또한 리액트는 브라우저 이벤트를 합성한 합성 이벤트(`SyntheticEvent`)를 제공한다.

```ts
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event
) => void | null;
type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;

const eventHandler1: GlobalEventHandlers["onchange"] = (e) => {
  e.target; // 일반 Event는 target이 없음
};

const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음
};
```

`React.ChangeEventHandler<HTMLSelectElement>` 타입은 `React.EventHandler<ChangeEvent<HTMLSelectElement>>`와 동일한 타입이다. `onChange`는 HTML의 select 엘리먼트에서 발생하는 `change` 이벤트에 대한 핸들러로 선언되었다. 이제 우리는 `ChangeEvent<HTMLSelectElement>` 타입의 이벤트를 매개변수로 받아 해당 이벤트를 처리하는 핸들러를 작성할 수 있게 되었다.

```ts
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return <select onChange={handleChange}>{/* ... */}</select>;
};
```

### 5. 훅에 타입 추가하기

`useState` 같은 함수 역시 타입 매개변수를 지정해줌으로써 반환되는 `state` 타입을 지정해줄 수 있다. 만약 제네릭 타입을 명시하지 않으면 타입스크립트 컴파일러는 초깃값의 타입을 기반으로 `state` 타입을 추론한다.

```ts
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```

만약 타입 매개변수가 없다면 `fruit`의 타입이 `undefined`로만 추론되면서 `onChange`의 타입과 일치하지 않아 오류가 발생한다.

```ts
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();

return (
  <Select
    // Error - SetStateAction<undefined>와 맞지 않음
    // (changeFruit에는 undefined만 매개변수로 넘길 수 있음)
    onChange={changeFruit}
    options={fruits}
    selectedOption={fruit}
  />
);
```

```ts
const [fruit, changeFruit] = useState("apple");

// error가 아님
const func = () => {
  changeFruit("orange");
};
```

```ts
type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>("apple");

// 에러 발생
const func = () => {
  changeFruit("orange");
};
```

### 6. 제네릭 컴포넌트 만들기

```ts
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```

`selectedOption`은 `options`에 존재하지 않는 값을 받아도 아무런 오류가 발생하지 않는다. `Option`의 타입에서 키가 `string`이기만 하면 `prop`으로 넘겨줄 수 있기 때문이다. 하지만 `changeFruit`의 매개변수 `Fruit`은 `prop`으로 전달돼야 하는 `onChange`의 매개변수 타입인 `string`보다 좁기 때문에 타입 에러가 발생한다.

함수 컴포넌트 역시 함수이므로 제네릭을 사용한 컴포넌트를 만들어 이러한 문제를 해결할 수 있다.

```ts
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>) => {
  // Select component implementation
};
```

`Select` 컴포넌트에 전달되는 `props`의 타입 기반으로 타입이 추론되어 `<Select<추론된_타입>>` 형태의 컴포넌트가 생성된다. 이제 `FruitSelect`에서 잘못된 `selectedOption`을 전달하면 타입 에러가 발생한다.

```ts
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  // ...
  // <Select<Fruit> ... />으로 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해줍니다
  // Type Error - Type "orange" is not assignable to type "apple" | "banana" | "blueberry" | undefined

  return (
    <Select options={fruits} onChange={changeFruit} selectedOption="orange" />
  );
};
```

### 7. `HTMLAttributes, ReactProps` 적용하기

`className, id`와 같은 리액트 컴포넌트의 기본 `props`를 추가하려면 `SelectProps`에 직접 `className?: string; id?: string;`을 넣어도 되지만 아래처럼 리액트에서 제공하는 타입을 사용하면 더 정확한 타입을 설정할 수 있다.

```ts
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

`ComponentPropsWithoutRef`는 리액트 컴포넌트의 `prop` 타입을 반환해주는 타입이다.

`Type['key']`를 활용하면 객체 형식의 타입 내부 속성값을 가져올 수 있다. `ReactProps`에서 여러 개의 타입을 가져와야 한다면 `Pick` 키워드를 활용하여 아래처럼 사용할 수도 있다.

`Pick<Type, 'key1' | 'key2' ...>`는 객체 형식의 타입에서 `key1, key2 ...`의 속성만 추출하여 새로운 객체 형식의 타입을 반환한다.

```ts
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
  // ...
}
```

### 8. `styled-components`를 활용한 스타일 정의

```ts
const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;
type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];
```

```ts
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;
```

`Select`를 사용하는 부모 컴포넌트에서 원하는 스타일을 적용하기 위해 `Select` 컴포넌트의 `props`에 `SelectStyleProps` 타입을 상속한다.

`Partial<Type>`을 사용하면 객체 형식의 타입 내 모든 속성이 옵셔널로 설정된다.

```ts
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = "default",
  color = "black",
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

### 9. 공변성과 반공병성

객체의 메서드 타입을 정의하는 방법은 2가지가 있다. 두 방법은 얼핏 비슷해 보이지만 미묘한 차이를 가지고 있다.

```ts
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}

const Component = () => {
  const changeToPineApple = (selectedApple: "apple") => {
    console.log("this is pine" + selectedApple);
  };

  return (
    <Select
      // Error
      // onChangeA={changeToPineApple}
      // OK
      onChangeB={changeToPineApple}
    />
  );
};
```

`onChangeA`일 때는 타입 에러가 발생하지만, `onChangeB`일 때는 타입 에러가 발생하지 않는다.

## 8.3 정리

### 최종적인 `Select` 컴포넌트

```ts
import React, { useState } from "react";
import styled from "styled-components";

const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;

type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];

interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;

type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>>
  extends Partial<SelectStyleProps> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  className,
  id,
  options,
  onChange,
  selectedOption,
  fontSize = "default",
  color = "black",
}: SelectProps<OptionType>) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <StyledSelect
      id={id}
      className={className}
      fontSize={fontSize}
      color={color}
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </StyledSelect>
  );
};

const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

type Fruit = keyof typeof fruits;

const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select
      className="fruitSelectBox"
      options={fruits}
      onChange={changeFruit}
      selectedOption={fruit}
      fontSize="large"
    />
  );
};

export default FruitSelect;
```
