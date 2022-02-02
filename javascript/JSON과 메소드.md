## JSON과 메소드
객체를 주고 받는 과정에서 객체 정보가 빈번하게 수정됨
객체를 문자열로 확인하는 메소드나 기타 정보를 확인하게 하는 메소드를 만들어도 특정 property가 사라지면 메소드를 수정해야됨
이거 여간 짜증나는게 아니니 JSON 쓰셈

### JSON
- JSON은 값/객체를 나타내는 포맷.
- JSON -> 객체 `parse`메소드
- 객체 -> JSON `stringfy`메소드(serialized)

### serialized 된 값/객체
- 문자열은 다 ""로 감싼다.
- 객체, 배열, 문자, 숫자, boolean, null 지원
- 함수, Symbol, undefined property는 미지원
- nested object 케이스도 알아서 척척척
- 순환참조 하는 객체는 serialize x
    ```javascript
    let ob1 = {...};
    let ob2 = {...};
    ob1.ref = ob2;
    ob2.ref = ob1;
    // Error: Converting circular structure to JSON
    JSON.stringfy(ob1); // ERROR
    ```

### stringfy
`JSON.stringify(value[, replacer, space])`
- value : 인코딩 하려는 값
- replacer : 인코딩하고 싶은 property의 배열 or 매핑함수
- space : 서식 변경 목적(가독성)으로 사용할 들여쓰기용 공백문자 수

**replacer**는 순환참조하는 객체 케이스 등을 처리할 때 사용
```javascript
let room = {
  number: 23
};

let meetup = {
  title: "Conference",
  participants: [{name: "John"}, {name: "Alice"}],
  place: room
};

room.occupiedBy = meetup;
// {"title":"Conference","participants":[{},{}]}
// 배열에 name이 없어 participants가 빈 값을 갖는다
alert( JSON.stringify(meetup, ['title', 'participants']) );

/*
{
  "title":"Conference",
  "participants":[{"name":"John"},{"name":"Alice"}],
  "place":{"number":23}
}
*/
alert( JSON.stringify(meetup, ['title', 'participants', 'place', 'name', 'number']) );
```

replacer에 배열 대신 함수를 넣어서 변경도 가능하다
- replacer 함수는 property (key,value)쌍 전체를 대상으로 호출됨
- 기존 값을 대신할 새 값을 반환(누락시키고 싶으면 undefined 반환)
- nested object든 뭐든 척척척
- replacer 함수의 `this`는 현재 처리하는 property가 위치한 객체

```javascript
alert( JSON.stringify(meetup, function replacer(key, value) {
  alert(`${key}: ${value}`);
  return (key == 'occupiedBy') ? undefined : value;
}));
/* alert 결과
:             [object Object]
title:        Conference
participants: [object Object],[object Object]
0:            [object Object]
name:         John
1:            [object Object]
name:         Alice
place:        [object Object]
number:       23
*/
```
replacer 함수가 처음으로 처리하는 케이스가 이상한데, 이는 최초 호출시 `{"": meetup}` wrapper object부터 만들고 처리하기 시작해서임.

### 커스텀 `toJSON`
- `toString`처럼 `toJSON`메소드로 serialize가능.
- `stringfy`는 `toJSON`을 감지하고 호출
```javascript
let room = {
  number: 23,
  toJSON() {
    return this.number;
  }
};

let meetup = {
  title: "Conference",
  date: new Date(Date.UTC(2017, 0, 1)),
  room
};

alert( JSON.stringify(meetup) );
/*
  {
    "title":"Conference",
    "date":"2017-01-01T00:00:00.000Z",  // Date의 toJSON 호출된 결과
    "room": {"number":23}               // room의 toJSON 호출된 결과
  }
*/
```

### JSON.parse
`JSON.parse(str [, reviver]);`
- reviver: 모든 (key, value)쌍을 대상으로 호출되는 function(key, value)
```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
let meetup = JSON.parse(str);
// date property는 걍 문자열이라 에러 뜬다
// meetup.date.getDate(); 

meetup = JSON.parse(str, function(key, value) => {
  if (key == 'date') return new Date(value);
  return value;
})
// 잘됨. 심지어 nested object에 date property가 있어도 잘됨.
meetup.date.getDate();
```