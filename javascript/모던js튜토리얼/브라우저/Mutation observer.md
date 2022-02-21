# Mutation observer
DOM요소를 감시하다 변화가 생기면 callback호출하는 built-in 객체
```js
let observer = new MutationObserver(callback);
observer.observe(node, config);
```
- `config`객체의 option(뭐에 반응할지)
    - `childList` – 직계 자손요소의 변화
    - `subtree` – 모든 후손요소의 변화
    - `attributes` – 속성의 변화
    - `attributeOldValue`: `true`면 수정 전 값과 수정 후 값을 모두 callback인자로 넘겨줌
    - `attributeFilter` – 감시할 속성들의 배열
    - `characterData` – 텍스트 변화(`node.data`)
    - `characterDataOldValue` - `true`면 수정 전 값과 수정 후 값 모두 callback인자로 넘겨줌

mutation observer의 메소드
- `disconnect` : 감시 중단.
- `takeRecords` : `disconnect`메소드 사용했는데 아직 처리 덜된 mutation record들이 있을 수 있음. 걔네들 가져오기만 할 때 씀
    - 가져와진 레코드들은 처리용 큐에서 삭제되고, mutation observer의 callback핸들러도 걔네들 대상으론 호출안됨.

요소가 바뀌어서 호출된 callback의 첫 번째 인자로 [MutationRecord객체](https://dom.spec.whatwg.org/#mutationrecord)의 리스트를, 두 번째 인자로 mutation observer를 넘겨줌

MutationRecord객체의 property
- `type` : 뭐 변화됬냐
    - `attributes`
    - `characterData`
    - `childList`
- `target` : 변화가 일어난 곳
    - 속성이면 요소, 텍스트면 텍스트노드, 추가/삭제된 후손
- `addedNodes/removedNodes` : 추가/삭제된 요소
- `previousSibling/nextSibling` : 추가/삭제된 요소의 형제요소
- `attributeName/attributeNamespace` : 변경된 속성의 이름/namespace(xml용)
- `oldValue` : 수정 전 값

```HTML
<div contentEditable id="elem">Click and <b>edit</b>, please</div>

<script>
let observer = new MutationObserver(mutationRecords => {
  console.log(mutationRecords); // console.log(the changes)
});

// observe everything except attributes
observer.observe(elem, {
  childList: true, // observe direct children
  subtree: true, // and lower descendants too
  characterDataOldValue: true // pass old data to callback
});
</script>
```

대충 언제 쓸지 예제
- 원하지 않는 광고 등장시 삭제해버리기
- 브라우저 코드 에디터 같은거 만들다보면 코드 하이라이트 필요할 때

mutation observer는 노드들 참조할 때 내부적으로 weak reference를 씀. 노드가 DOM에서 삭제되면 unreachable => garbage collection 작동