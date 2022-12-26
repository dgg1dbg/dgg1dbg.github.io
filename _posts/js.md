---
published: false
---
# Javascript 공부

### 1. 기본 문법
1. Hello world!
    ```html
    <script>
        alert('Hello world');
    </script>
    <script src="/path/to/script.js"></script>
    ```
2. 주석
    ```js
    //주석1
    /*
    주석2
    */
    ```
3. 엄격모드
    ```js
    "use strict";
    //모던한 방식(ES5)로 코드 실행
    ```
    
4. 기본 상호작용
    - alert
        ```js
        alert("hello");
        ```
    - prompt
        ```js
        result = prompt("type something", "default");
        ```
        입력을 취소한 경우, null이 반환됨.
    - confirm
        ```js
        let result = confirm("true or false");
        alert(result);
        ```
        
### 2. 변수와 변수의 활용
1. 변수와 상수

    - 변수
        ```js
        let message = 'Hello'
        let a;
        a = 'a';
        ```
        let과 var는 거의 동일한 키워드이다.  
        변수를 두 번 이상 선언하면 에러가 발생한다.  
        use strict가 없으면 let 없이도 선언이 가능하나, 추천하지 않는다.  
    - 상수
        ```js
        const message = "hello";
        ```
2. 자료형

    1. number type
        정수, 부동소수점, Infinity, -Infinity, NaN  
        0으로 나눌 경우, zero division error가 아닌 Infinity를 출력한다.  
        NaN은 계산 중 에러가 발생했다는 것을 의미함.  
    2. bigint
        2^53^-1 이상 -(2^53^-1) 이하인 숫자를 사용할때. 
        ```js
        const bigInt = 1234567890123456789012345678901234567890n;
       ```
    3. string
        큰 따옴표나 작은 따옴표는 차이를 두지 않음.  
        역 따옴표는 파이썬의 fstring 처럼 활용할 수 있다.
        ```js
        let name = "John";
        alert(`Hello, ${name}!`); // Hello, John!
        ```
    4. boolean
        true, false
    5. null
        값을 의도적으로 할당하지 않음
    6. undefined
        값을 할당하지 않은 상태
        ```js
        let age = 100;
        age = undefined;
        alert(age); // "undefined"
        ```
    typeof x 또는 typeof(x)로 변수의 자료형을 확인할 수 있다.

3. 형변환
    
    - 문자형으로 변환
        변수가 출력될 때, 또는 String()함수가 사용되었을때 일어난다.
    - 숫자형으로 변환
        변수가 수식 속에서 활용되었을 때, 또는 Number()함수가 사용되었을 때 일어난다.  
        숫자 이외의 글자가 들어있는 문자열의 경우, NaN이 반환된다.  
    - 불린형으로 변환
        논리연상이 수행될 때, 도는 Boolean()함수가 사용될 때 일어난다.  
        0, 빈 문자열, null, undefined, NaN과 같은 비어있는 값들이 false가 된다. 

4. 연산자
    - +이항 연산자의 경우 피연사중 하나가 문자열이면, 다른 하나도 문자열로 변환됨.
    - +단항연산자의 경우, 피연산자를 숫자형으로 변환하는 역활을 함.
    - =연산자는 값을 대입하는 것 뿐만 아니라 반환하는 역활도 함.
        ```js
        let a, b, c;
        a = b = c = 2 + 2;
        ```
    - ,연산자는 여러 표현식을 한 줄에 평가할 수 있도록 함. 마지막 부분만 반환이 된다.  
        ```js
        let a = (1 + 2, 3 + 4);
        alert( a ); // 7 (3 + 4의 결과)
        ```
    - ==(equality operator)와 ===(strict equality operator)의 차이를 알아야 한다. ===는 비교 시 형변환을 하지 않는다.
    - \=\=\=를 제외한 다른 비교 연산자들은 형변환을 기본적으로 수행한다. 다만 \=\=의 경우 null 또는 undefined가 오면, 형변환을 수행하지 않는다. 그 외의 경우 null은 0, undefined는 NaN으로 변환된다. 한편, null==undefined는 true이다... ㅋㅋ
    
### 3. 기본 코드 구조
1. if, else문
    ```js
    if (year == 2015){
        alert("true");
    } else if (year > 2015){
        alert("too big");
    }else {
        alert("too small");
    }
    ```
2. ? 조건부 연산자
    ```js
    let result = condition ? val1 : val2;
    ```
3. || 연산자
    피연산자가 불린형이 아닌 경우, 첫번째 true값, 모두 false인 경우 마지막 값을 반환한다.
4. && 연산자
    피연산자가 불린형이 아닌 경우, 첫번째 false값, 모두 true인 경우 마지막 값을 반환한다.
5. ?? 연산자
    ||와 비슷하지만, true가 아닌 null 또는 undefined 여부를 확인함.
    ```js
    let height = 0;
    height = height || 100; //height는 100
    height = height ?? 100; //height는 0
    ```
6. while, for문
    ```js
    while (condition){
        //code
    }
    
    do {
        //code
    } while (codition);
    ```
    ```js
    for (let i=0; i<3; i++){
        //code
    }
    //for문 변수 선언, 조건, 스텝을 생략할 수 있다. 
    ```
    continue와 break가 존재한다.  
    레이블을 사용하여 특정 부분으로 continue, break할 수 있다.
    ```js
    labelName: for (...) {
      ...
    }
    ```
7. switch문
    ```js
    switch(x){
        case 1:
            //code
        case 2:
        case 3:
            //code
        default:
            //code
    }
    ```
    *switch문의 조건은 값 뿐만아니라 자료형까지의 일치를 요구한다.*  
    
    

### 4. 함수
1. 함수의 선언
    ```js
    function showMessage(message, option="default") {
        alert(message);
        return true;
    }
    ```
    매개변수에 값이 할당되지 않은 경우, 매개변수는 undefined값을 갖고, return 값이 없을 경우 함수는 undefined를 반환한다.
2. 함수 표현식
    ```js
    let sayHi = function(){
        alert("Hello");
    };
    
    let func = sayHi;
    func();
    ```
    함수를 이러한 변수의 개념으로 바라봄으로써, 함수를 콜백함수로서 활용할 수도 있고, 익명함수를 만들 수도 있다. 
    함수 선언문과 함수 표현식의 차이점은,
    1. 함수 선언문은 해당 구역에 대해 전역적이다.(순서 상관 x)  
    2. 함수 표현식은 함수 변수의 선언 위치에 따라, 전역적으로 활용할 수 있다.  
3. 화살표 함수
    ```js
    let sum = (a, b) => a + b;
    
    /* 위 화살표 함수는 아래 함수의 축약 버전입니다.
    
    let sum = function(a, b) {
      return a + b;
    };
    */
    ```
    매개변수가 하나면 괄호를 생략하고, 매개변수가 없으면, 괄호만 사용하면 된다.  
    ```js
    let age = prompt("나이를 알려주세요.", 18);
    
    let welcome = (age < 18) ?
      () => alert('안녕') :
      () => alert("안녕하세요!");
    
    welcome();
    ```

### 5. 자료구조와 자료형
1. 숫자형
    - num.toString(base)
        base진법으로 숫자를 변환한다.
    - Math.floor(num) / Math.ceil(num) / Math.round(num) / Math.trunc(num)
        바닥함수, 천장함수, 반올림, 소수부 삭제
    - isNaN(num) / isFinite(num)
    - parseInt(str) / parseFloat(str)
        가능한 부분까지 정수와 실수를 반환함
    - Math.random()
    - Math.max(nums) / Math.min(nums)
2. 문자열
    - str.length
    - str.[pos] / str.charAt(pos)
        없을 경우 undefined, ''를 반환
    - str.toUpperCase()
    - str.indexOf(str [, pos]) / str.lastindexOf()
        없을 경우 -1 반환
    - str.includes(str) / str.startsWith(str) / str.endsWith(str)
    - str.slice(start [, end])
        end는 포함하지 않음
        음수를 통해 뒤를 기준으로 인덱싱 할 수 있음
    -str.substring(start [ ,end])
        slice와의 차이는 slice는 start가 end보다 크면 ''를 반환하며, 음수 값을 활용할 수 있다는 점이다.
    -str.substr(start [,length])
3. 배열
    ```js
    let arr = new Array();
    let arr = [];
    let arr = ["a", "b"];
    ```
    배열 요소의 자료형에는 제약이 없다.
    - push(), pop()을 통해 queue를 구현할 수 있다.
    - 배열을 순회할 때에는 for..of와 for..in을 사용할 수 있다.
        ```js
        for (let fruit of fruits) {
          alert( fruit );
        }
        for (let key in arr) {
          alert( arr[key] ); // 사과, 오렌지, 배
        }
        ```
    - 둘의 차이는 for .. in은 리스트 뿐만 아니라 객체의 프로퍼티에 대해서도 사용가능하다는 점이다. 그만큼 느리다.
    ```js
    let matrix = [
      [1, 2, 3],
      [4, 5, 6],
      [7, 8, 9]
    ];
    
    alert( matrix[1][1] ); 
    ```
4. 배열과 메서드
    - arr.splice(index [, deleteCount, elem1, ...,elemN])
        배열의 요소를 index부터 deleteCount 개만큼 삭제하고 반환한다.  
        deleteCount를 0으로 하면, 삭제하지 않고, elm들을 삽입하는 함수로 활용할 수 있다.  
    - arr.slice([start], [end])
        새로운 배열을 반환
    - arr.concat(arg, arg2....)
        arg가 배열인 경우, 평탄화하여 결합한다.  
    - arr.forEach
        ```js
        ["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {
          alert(`${item} is at index ${index} in ${array}`);
        });
        ```
    - arr.indexOf(item, from) / arr.lastIndexOf(from) / arr.includes(item, from)
    - arr.find(fn) / arr.findIndex(fn)
        ```js
        let arr = [1, 2, 3, 4, 5];
        let elm = arr.find(item => item==2);
        alert(elem);
        ```
        findIndex는 요소가 아닌 요소의 위치를 반환(없으면 -1)한다.
    - arr.filter(fn)
        ```js
        let arr = [1, 2, 3, 4, 5];
        let new_arr = arr.filter(item => item % 2 == 0);
        alert(new_arr);
        ```
    - arr.filter(fn)
        ```js
        let result = arr.map(function(item, index, array) {
          // 요소 대신 새로운 값을 반환합니다.
        });
        ```
    - arr.map(fn)
    - arr.sort(fn)
        기본적으로, fn을 명시하지 않는 경우 sort는 배열의 원소를 문자열로 변환하여 비교한다는 점을 주의해야한다.
        ```js
        functon fn(a, b) {
          if (a > b) return 1;
          if (a == b) return 0;
          if (a < b) return -1;
        }
        ```
    - arr.reverse()
    - arr.split(str)
        str을 기준으로 문자열을 쪼개어 배열로 만든다.  
        str을 ''로 정의하면, 글자단위로 문자열을 쪼갤 수 있다. 
    - arr.join(str)
    - arr.reduce(fn) / arr.reduceRight(fn)
        ```js
        let value = arr.reduce(function(accumulator, item, index, array) {
          // ...
        }, [initial]);
        ```
        ```js
        let arr = [1, 2, 3, 4, 5];
        let result = arr.reduce((sum, current) => sum + current, 0);
        alert(result); // 15
        ```
    - Array.isArray(arr)
    
    
    
    
        
        
        
        
        
        
        