# Selection and Range

## Range
- selection의 기본 컨셉은 선택된 범위의 시작/끝 point 값을 갖는 [Range](https://dom.spec.whatwg.org/#ranges)임. 
- 부모 노드에 따라 point의 offset값이 달라짐
    - element면 point의 offset은 자손 element의 순번
        ![element](https://ko.javascript.info/article/selection-range/range-example-p-0-1.svg)
        ```html
        <p id="p">Example: <i>italic</i> and <b>bold</b></p>
        From <input id="start" type="number" value=1> – To <input id="end" type="number" value=4>
        <button id="button">Click to select</button>
        <script>
        button.onclick = () => {
            let range = new Range();

            range.setStart(p, start.value);
            range.setEnd(p, end.value);

            // apply the selection, explained later
            document.getSelection().removeAllRanges();
            document.getSelection().addRange(range);
        };
        </script>
        ```
    - 텍스트노드면 point의 offset은 해당 문자의 위치임.
        ```html
        <p id="p">Example: <i>italic</i> and <b>bold</b></p>

        <script>
        let range = new Range();

        range.setStart(p.firstChild, 2);
        range.setEnd(p.querySelector('b').firstChild, 3);

        alert(range); // ample: italic and bol

        // use this range for selection (explained later)
        window.getSelection().addRange(range);
        </script>
        ```
        ![텍스트](https://ko.javascript.info/article/selection-range/range-example-p-2-b-3-range.svg)
        - 위 예시에서 빠진 `range`의 `collapsed` property는 시작점과 끝점이 같은 경우(그니까 `range`안에 암것도 없는 경우) `true`임
### Range methods
| 메소드 | 설명 |
| --- | --- |
|`setStart(node, offset)` | 노드 내 offset을 시작점으로 설정 |
|`setStartBefore(node)`| 해당 노드 바로 이전을 시작점으로 설정 |
|`setStartAfter(node)` | 해당 노드 바로 다음을 시작점으로 설정 |
| `setEnd(node, offset)` | 노드 내 offset을 끝점으로 설정 |
| `setEndBefore(node)` | 해당 노드 바로 이전을 끝점으로 설정 |
| `setEndAfter(node)` | 해당 노드 바로 다음을 끝점으로 설정 |
| `selectNode(node)` | 노드 전체를 range로 설정 |
| `selectNodeContents(node)` | 노드 내부 컨텐츠를 range로 설정 |
| `collapse(toStart)` | `toStart=true`면 `end=start`, `false`면 `start=end` |
| `cloneRange()` | range 복사 |
| `deleteContents()` | range영역 내의 요소들을 삭제 |
| `extractContents()` | range영역 내의 요소들을 삭제하고 삭제된 요소들을 `DocumentFragment`로 반환 |
| `cloneContents()` | range영역 내의 요소들을 복사해 `DocumentFragment`로 반환 |
| `insertNode(node)` | `node`를 range 시작점에 추가 |
| `surroundContents(node)` | `node`로 range를 감싼다. `<i>abc`같이 부분 태그가 들어있으면 실패함. |

## Selection
- `range`를 0개 이상 보유하는 객체. 
- `window.getSelection()`나 `document.getSelection()`로 확인 가능.
- firefox 제외하고 `selection`이 가질 수 있는 `range`는 1개로 제한됨..
- `range`와 달리 방향을 가지고 있음. selection은 마우스를 좌->우측으로 드래그 할 때랑 우->좌측으로 드래그 할때의 시작점/끝점이 다르게 나옴
- property
    - `anchorNode` : selection이 시작된 노드
    - `anchorOffset` : `anchorNode`에서 selection이 시작된 offset
    - `focusNode` : selection이 끝난 노드
    - `focusOffset` : `focusNode`에서 selection이 끝난 offset
    - `isCollapsed` : selection 내부가 비었거나 selection이 없으면 `true`
    - `rangeCount` : 보유한 `range`수. firefox빼곤 죄다 1이 최대임
- event
    - `elem.onselectstart` : 요소에서 selection이 시작될 때 발생. 기본 동작을 막으면 selection이 시작되지 않음
    - `document.onselectionchange` : selection이 변경될 때마다 발생. `document`에만 설정 가능하다.
    ```html
    <p id="p">Select me: <i>italic</i> and <b>bold</b></p>

    Cloned: <span id="cloned"></span>
    <br>
    As text: <span id="astext"></span>

    <script>
    document.onselectionchange = function() {
        let selection = document.getSelection();

        cloned.innerHTML = astext.innerHTML = "";

        // Clone DOM nodes from ranges (we support multiselect here)
        for (let i = 0; i < selection.rangeCount; i++) {
            cloned.append(selection.getRangeAt(i).cloneContents());
        }

        // Get as text
        astext.innerHTML += selection;
    };
    </script>
    ```
- 메소드
    - `getRangeAt(i)` – i번째 range 반환. firefox만 작동되고 나머지 브라우저는 걍 0임.
    - `addRange(range)` – range 추가. 당연히 firefox만 작동되고, 이미 있는거면 무시됨.
    - `removeRange(range)` – range삭제
    - `removeAllRanges()` – 모든 range삭제
    - `empty()` – `removeAllRanges`와 동일
    - 기타 등등..

### form에서 Selection
`<input>`이나 `<textarea>`에서 property로 지원함
- `input.selectionStart`
- `input.selectionEnd`
- `input.selectionDirection` – selection 방향(`forward`, `backward`, `none`(더블클릭))

이벤트로 `onselect`도 지원해주고 죄다 선택해주는 `select`메소드나 범위 지정해서 선택해주는 `setSelectionRange`같은것도 지원해줌

예제
```html
<input id="input" style="width:200px" value="Select here and click the button">
<button id="button">Wrap selection in stars *...*</button>

<script>
button.onclick = () => {
  if (input.selectionStart == input.selectionEnd) {
    return; // nothing is selected
  }

  let selected = input.value.slice(input.selectionStart, input.selectionEnd);
  input.setRangeText(`*${selected}*`);
};
</script>
```

### Making unselectable
1. css `user-select: none`<br>
해당 요소에서 select못하게는 할 수 있는데 밖에서부터 드래깅해서 들어오면 다른 애들은 다 되는데 지혼자 뻘쭘하게 select 안되게 보임. 복사하려고 해도 select 안된부분은 복사에 포함 안됨
2. `onselectstart`나 `mousedown` 막아버리기
    ```js
    <div>Selectable <div id="elem">Unselectable</div> Selectable</div>

    <script>
    elem.onselectstart = () => false;
    </script>
    ```
    얘는 해당 영역만 select안되고 밖에서 드래깅해서 들어오면 같이 selecte됨.
3. `document.getSelection().empty()` 사용<br>
자주 사용되진 않음. 깜빡이는 현상도 있어서 별로인듯