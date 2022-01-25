
# How to write CSS

우리팀은 앞으로의 프로젝트에 있어서 CSS 작성 방법을 토론했고, CSS의 발전 과정과 각 단계의 장,단점을 살펴본 뒤 CSS iN JS 방식인 `Styled Component`를 선택하였습니다. CSS가 왜 발전하게 되었는지, 그 문제점이 무엇이었는지에 대해 중점을 뒀습니다.

## CSS

CSS의 발전 순서는 다음과 같습니다.

1. CSS
2. CSS in CSS

- CSS Pre-processor(SASS)
- CSS Module

3. CSS in JS(Styled-Component)

각 단계마다 전 단계의 문제점을 개선하기 위해 다음 단계의 CSS가 출시되었습니다.

### CSS의 단점

다음은 페이스북 개발자인 Christopher Chedeau aka Vjeux 가 소개한 CSS의 문제점입니다.

#### Global namespace

> 모든 스타일이 global에 선언되어 중복되지 않는 class 이름을 적용해야 하는 문제

css의 가장 대표적인 문제점으로 어디에 선언을 하던지, 어디에 import를 하던지 항상 `global namespace`를 가집니다.

#### Dead Code Elimination

> 코드를 수정하고 삭제하는 과정에서 불필요한 CSS를 제거하기 어려운 문제

CSS안에서 전혀 사용하지 않는 CSS가 있더라도 빌드 과정에서는 해당 CSS 라인은 여전히 남아있습니다. 또, 전임자(타인)이 작업했던 코드를 받게되면 어느 CSS를 사용하고 있고 어느 CSS가 불필요한지 확인하기가 어렵습니다.

#### Minification

> 클래스 이름의 최소화 문제

CSS의 의도는 최소한의 코드로 반복적인 서식을 피하기 위함이었으나 복잡한 디자인과 레이아웃이 만들어지게 되면서 많은 양의 코드를 작성하게 되었습니다.

#### Non-deterministic Resolution

> CSS 로드 순서에 따라 스타일 우선 순위가 달라지는 문제

CSS의 우선순위 문제는 상당히 복잡합니다. CSS 셀렉터마다 고유한 Level과 포인트가 존재하고 그 총 합을 통해 우선순위가 결정되며 같은 우선순위라면 CSS가 나중에 적용되는 순서대로 덮어쓰도록 되어 있습니다.

| 종류         | 점수                         |
| ------------ | ---------------------------- |
| !important   | 무한대                       |
| 인라인 선언  | 1000점                       |
| id선택자     | 100점                        |
| 클래스선택자 | 10점                         |
| 태그 선택자  | 1점                          |
| 전체 선택자  | 0점                          |
| 상속         | 항상 우선하지 않음. 점수없음 |

#### Isolation

> 전역적인 속성으로 인해 CSS 에서는 파일이나 구성 요소 간에 격리를 하는 것은 사실상 불가능에 가깝다.

다만 인라인 스타일을 적용한다면 해결은 할 수 있습니다.

#### Sharing Constants

> JS 코드와 상태(상수)값을 공유할 수 없는 문제

과거에는 문제가 되었으나 `CSS Variable`이 해결책으로 등장하였고 CSS 기본기능으로 탑재 되었습니다.(IE11 제외)

#### Dependencies

> CSS의 종속성 문제

아무리 특정 요소가 특정 CSS에 의존하고 있다고 해도 스타일은 전역적이기 때문에 이를 명확하게 말하는 것은 힘들며, 모든 파일에서 모든 스타일 요소를 적용할 수가 있으므로 제어를 잃기가 쉽습니다.

## CSS in CSS

이로 인해 CSS 전처리기가 등장합니다. 반복 작업 등 기존 CSS의 불리한 점을 해결하기 위해 개발되었습니다.

### CSS Preprocessor(SASS)

Syntax에 맞게 코드를 작성하고 컴파일러를 통해서 css 파일로 결과물이 나오는 구조입니다. 반복적인 부분을 스크립트나 변수로 처리할 수 있고, 대표적인 css 전처리기로는 `SASS`가 있습니다.

#### 특징

> 재사용성: 공통 요소 또는 반복적인 항목을 변수 또는 함수로 대체

```sass
$primary-color: #eee

.class-A
    background: $primary-color
    <!-- sass는 들여쓰기로 구분하고,세미콜론을 쓰지않는다. -->

.class-B
    background: $primary-color
```

> 시간적 비용 감소: 임의 함수 및 Built-in 함수(여기서는 darken)로 인해 개발 시간적 비용 절약

```sass
$color: #555
$buttonColor: darken($color, 10%)

button
  background: $color
  box-shadow: 0 0 5px $buttonColor
```

> 유지 관리: 중첩,상속과 같은 요소로 인해 구조화된 코드로 유지 및 관리가 용이

```sass
// 중첩
.class-A
    width: 100%

    .A-child-1
        color: red

    .A-child-2
        color: blue

// 컴파일 하면 다음과 같다.
.class-A {
  width: 100%
}

.class-A .A-child-1 {
  color: red
}

.class-A .A-child-2 {
  color: blue
}
```

```sass
//상속
.class-A
    color: red
    padding: 10px
.class-B
    @extend .class-A
    border: 1px solid red

// 컴파일 하면 다음과 같다.
.class-A, .class-B{
  color: red;
  padding: 10px;
}

.class-B {
  border: 1px solid red;
}
```

#### 한계

- 전처리기를 위한 도구가 필요하며, 다시 컴파일하는 시간이 느릴 수 있습니다.

### CSS Module

SASS는 클래스네임이 겹치지 않도록 개발자가 관리해야 하는 불편함이 있었습니다. 이로 인해서 CSS Module이 등장합니다.

CSS 모듈은 CSS를 모듈화 하여 사용하는 방식으로, 자동으로 고유한 클래스네임을 만들어서 scope를 지역적(local)으로 제한합니다. 때문에 css 파일이름(nameSpace)만 다르면 중복된 클래스네임을 사용할 수 도 있습니다.

#### 특징

> 클래스네이밍: 각각 CSS 파일 별로 각각의 CSS파일이 별도로 컴파일 되기 때문에 간단하게 네이밍된 클래스 선택자를 사용할 수 있고, 각각 중복된 이름이 가능하다. 실제 사용되는 클래스명은 자동으로 생성되고 유니크함이 보장된다.

```css
/* myStyle.module.css */
.btn {
  color: red;
}

/* 컴파일 하면 다음과 같다. */

.myStyle_btn_j3xk {
  /* 클래스네임이 자동으로 생성된다. */
  color: red;
}
```

```jsx
import style from "./myStyle.module.css";

<button className={style.btn}>버튼</button>;
```

#### 한계

- 한 곳에서 모든 것을 작성하지 않기 때문에 별도로 많은 CSS 파일을 만들어 관리해야 합니다.

## CSS in JS(Styled-Component)

CSS Module은 방대하고 많은 CSS 파일을 관리해야 하는 불편함이 있었습니다. 이로 인해서 CSS in JS가 등장합니다. 하나의 컴포넌트에 CSS를 같이 작성하며 이로인해 컴포넌트에 대한 관리를 쉽게 할수 있게 됩니다.

2014년 CSS의 문제점을 제기한 Vjuex가 처음 소개하였으며, 앞서 말한 문제점을 모든 이슈를 해결할 수 있다고 강조하였습니다.

### 특징

#### props를 활용한 조건부 스타일링

> JS와 CSS사이의 상수와 함수,변수를 하여, 이를 활용한 스타일링이 가능하다.

```jsx
import styled from "styled-components";

const Container = styled.div`
  background: ${({ active }) => {
    return active ? "white" : "black";
  }};
`;

function Exam() {
  <Container active={true} />
}
```

#### 코드 경량화

> 짧은 길이의 유니크한 클래스를 자동으로 생성해준다.

```jsx
import styled from "styled-components";

const Container = styled.div`
color: white;
`

<Container className="zyvqL"> // 실제 생성되는 클래스네임
```

#### 한계

- FOUC(Flash of unsttyled content): 브라우저에 보여줄 것들이 많아짐에 다라, 점차적으로 속도가 느려진다. 특히, 컴포넌트가 렌더링되며 형태가 잡히기 때문에 원형(날것)의 모습이 잠깐 노출되는 현상인 FOUC가 나타난다.
- 인터랙션한 페이지일 경우 스타일이 동적인 변수에 의존하게 되어, 해당 컴포넌트가 리렌더링되면 CSS 파일을 따로 관리하는 방법에 비해 느린 성능을 보여줄 수 있다.
- 별도의 라이브러리를 설치해 하므로 번들의 크기가 커진다.

#### 결론

- 큰 규모의 프로젝트가 아니기 때문에, 위에서 언급한 한계(성능 이슈, 번들 크기)는 해당되지 않는다고 판단하였습니다.
- CSS in JS로 작성한 코드를 CSS파일로 따로 최적화하여 추출해주는 라이브러리([Linaria](https://github.com/callstack/linaria),[Compiled](https://compiledcssinjs.com/))를 사용하면 성능 이슈를 해결할 수 있을 것이라 판단하였습니다.
- 지금은 `styled-component`로 결정하였지만, 필요시 다른 css(`sass`,`css-module`)도 유도리있게 사용하자고 결정하였습니다.