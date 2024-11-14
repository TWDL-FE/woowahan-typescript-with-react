Inline Style

```html
<!-- inline-style -->
<p style={{ color: 'white', backgroundColor: 'red' }}/>

<!-- Dom Node -->
<p style="”color:" white; background-color: red;” />
```

CSS-in-JS

```tsx
// css-in-js
const text = styled.div`
	color: white;
	background-color: black;
`

const Example = () => <Text/>

// Dom Node
<style>
.hash136s21 {
	color: white;
	background-color: black;
}
</style>

<div class='hash136s21"/>
```

CSS-in-JS 특징

- 컴포넌트 별로 개별의 스코프를 가져서 이름 중복, 선택자 중복을 고려하지 않아도 된다.
- 자동으로 벤더 프리픽스가 붙는다.
- CSS를 컴포넌트와 동일하게 관리, 분리 할 수 있다.
- 자바스크립트 함수를 사용해서 CSS 작성이 가능하다.
- 컴포넌트의 props와 동일한 속성을 전달할경우 Pick, Omit 등의 유틸리티 타입 사용 가능

```html
<style data-emotion="css" data-s="">
  .css-1ikpj0u-SelectHeader {
    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: justify;
    -webkit-justify-content: space-between;
    justify-content: space-between;
    -webkit-align-items: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    padding: 10px;
    border-radius: 4px;
    cursor: pointer;
    background: #010101;
    gap: 4px;
    color: #ffffff;
  }
</style>
```
