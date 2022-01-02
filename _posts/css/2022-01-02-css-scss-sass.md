<details>
<summary><h2>CSS? SCSS? SASS?</h2></summary>  

SCSS나 SASS는 CSS를 편리한 사용을 지원하고 확장판 기능을 제공한다.  
아래는 각각의 코드 스타일이다.

<details>
    <summary><H3>CSS 코드 스타일</H3></summary>

```css
.list {
    width: 100px;
    float: left;
}
  li {
      color: red;
  }
  li:last-child {
      margin-right: -10px;
  }
```

</details>

<details>
    <summary>SCSS 코드 스타일</summary>

```scss
.list {
width: 100px;
float: left;
li {
color: red;
&:last-child {
margin-right: -10px;
}
}
}
```

</details>

<details>
<summary>SASS 코드 스타일</summary>

```sass
.list
width: 100px
float: left
li
color: red
&:last-child
margin-right: -10px
```

</details>
</details>

## 그렇다면 왜 사용하는가?
CSS의 태생적인 한계를 극복하기 위해서 만들어졌다고 한다. 기본적으로 코드 분량이 많아질 수록 고도화 될 수f록 코드라인의 수는 증가한다. 개인적인 생각으로는 bootstrap만 이용하더라도 어지럽다...  
이러한 문제를 해결하기 위해 다음과 같은 항목들을 지원한다.  
<ul>
    <li>Variable</li>
    <li>If and For</li>
    <li>Import</li>
    <li>Nesting</li>
    <li>Mixin</li>
    <li>Extend/Inheritance</li>
</ul>
