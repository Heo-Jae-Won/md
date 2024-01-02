## <span style="color:#802548">_1. ref_</span>
- ref로 아래와 같이 generic을 사용할 수 있다. 
- ref<Board>로 선언하면 generic에 필요한 모든 데이터의 초기값을 지정해야 한다. 
```js
const boardInfo = ref<board>({
    boardTitle: '',
    boardContent: '',
    boardWriter: loginUser,
});

const formData = new FormData();
formData.append('boardTitle', boardInfo.boardTitle);
formData.append('boardContent', boardInfo.boardContent);
formData.append('boardWriter', boardInfo.boardWriter);
```
- 하지만 데이터들이 반드시 어떤 type, interface의 일부라는 확신이 있다면 단언하여 사용할 수 있다. 
- 나는 타입 단언이 편리한 거 같다.
```js
const boardInfo = ref({} as Board);
formData.append('boardTitle', boardInfo.value.boardTitle);
formData.append('boardContent', boardInfo.value.boardContent);
formData.append('boardWriter', loginUser.value);
```
- 그럼 axios를 통해 data를 반응성 변수인 ref에 뿌려주고 DOM이 형성된 이후 데이터를 척척 맞춰준다. 
- DOM이 형성된 이후에 쓰는 게 onMounted(()=>{})다. 
- 그래서 onMounted(()=>{})에는 주로 axios에서 데이터를 받아와 fetch하는 함수들이 많이 쓰인다. React로 따지면 useEffect()와 비슷하다. 
```js
 const fetchBoardInfo = async () => {
      await getBoardInfo(boardNo).then(res => {
        boardInfo.value = res.data;
      });
    };

 onMounted(() => {
      fetchBoardInfo();
    });
```
- 위에서 만약 reactive로 선언했다면 재할당이 불가능해서 const 대신 let으로 써야한다. 
```js
let boardInfo = reactive<Board>({
    boardTitle: '',
    boardContent: '',
    boardWriter: loginUser.value,
});
```
- 하지만 그렇게 reactive를 써도 axios의 데이터를 받아오지 못해 초기값이 그대로 적용된다. 따라서 reactive로 쓰지 말자.
- 참고로 ref변수를 쓰면 templat에서는 자동으로 value를 가져오게 되기 때문에(unwrapped 됨) template에서는 boardInfo.value.boardTitle이 아닌 boardInfo.boardTitle로 써주면 된다.
```html
 <VCol cols="12" md="4"> 제목: <VTextField :rules="[(v) => rules.max50(v)]" v-model="boardInfo.boardTitle" /> </VCol>
    <VCol cols="12" md="4">
        내용:
    <VTextField :rules="[(v) => rules.max300(v)]" v-model="boardInfo.boardContent" class="h-50" />
</VCol>
```
## <span style="color:#802548">_2. reactive_</span>
- 반면에 사용자 입력값을 받을 때는 reactive를 쓰는 게 더 편리하다. 
- 이걸 ref변수에 담아 쓰면 써야할 게 훨씬 많아진다. 우선 reactive와 ref를 모두 썼다.
```js
 const form = reactive<User>({
      userId: '',
      userPassword: '',
      userName: '',
      userEmail: '',
      userBirth: '',
      userGender: '',
      userAddress: '',
    });
```
```js
 const form = ref<User>({
      userId: '',
      userPassword: '',
      userName: '',
      userEmail: '',
      userBirth: '',
      userGender: '',
      userAddress: '',
    });
```
ref는 아래에서 form.을 더 추가해줘야 한다.
```html
비밀번호: <v-text-field v-model="userPassword" type="password" />
비밀번호:  <v-text-field v-model="form.userPassword" type="password" />
```

- ...toRefs()는 reactive 변수를 ref 변수로 변환해주는 함수다. 
- ref로 변환하지 않고 사용할 수는 없다. reactive는 기본형으로 쓸 수 없기 때문이다. 
- toRefs()는 reactive 변수를 인자로 받아 그 속에 있는 key-value 구조를 해석해 ref로 만들어 준다고 이해하면 편할 것 같다. 

​

- torefs()로 해서 모두 ref 변수가 된 거고 특정한 key만 ref로 변환하려면 toRef([reactive 변수명],[ref로 만들고자 하는 key])로 쓴다. 
- 예를 들면 ...toRefs(form, 'userId'); 같은 식이다.
```js
 return {
      ...toRefs(form), //...toRef(form, 'userId'),
    };

//
 return {
      form
    };
```
- 위와같이 return하면 아래와 같이 만들어지는 식이다. 
```js
const userId = ref('');
const userPassword = ref('');
const userName = ref('');
const userEmail = ref('');
const userBirth = ref('');
const userGender = ref('');
```
- ref로 form을 만들면 script에서도 .value를 더 추가해줘야 한다. 좀 귀찮다. 
- 그럴 땐 reactive로 만드는 게 더 편한것 같다. 
```js
const formData = new FormData();
formData.append("userPassword", form.userPassword);
//
const formData = new FormData();
formData.append("userPassword", form.value.userPassword);
```
- 아쉽게도 저기서 form을 destructuring하는 순간 반응성이 사라진다. 따라서 아래와 같이 사용할 수 없다. 
```js
const {userPassword} = form;
```
- 일일이 변수명 앞에 form.을 붙여줘야 한다. form.userPassword 같은 식으로 말이다.


## <span style="color:#802548">_3. computed_</span>
- computed는 복잡한 로직을 template에 때려박으면 매우 이쁘지 않기 때문에 쓴다. 
- 아래는 loginUser와 게시글 작성자가 동일한 지 비교하여 버튼을 무력화시키는 로직이다.
```html
<VBtn :disabled="boardInfo.boardWriter !== loginUser" type="button" block class="mt-2" @click="onDeleteSubmit">삭제하기</VBtn>
```
- 이를 computed를 활용한다면 아래와 같이 HTML 영역을 깔끔하게 쓸 수 있다.
```js
  const isWriterEquals = computed(() => boardInfo.value.boardWriter === loginUser.value);
```
  ```html
<VBtn :disabled="!isWriterEquals" type="button" block class="mt-2" @click="onDeleteSubmit">삭제하기</VBtn>
```
- 하지만 computed는 반응성 변수를 활용했을 때만 의도한 목적을 달성할 수 있다. 
- 즉 author를 reactive나 ref로 선언을 해야 update된 변수에 따라 computed가 반응한다는 것이다. 
- 아래도 보면 boardInfo와 loginuser 모두 ref변수로 선언된 것을 볼 수 있다. 
- ref변수의 값을 script 단에서 가져오려면 .value에서 key를 찾아야 한다.
```js
const isWriterEquals = computed(() => boardInfo.value.boardWriter === loginUser.value);
```

- 실제로 Vue 공식 사이트의 예시를 보면 Date.now()를 사용하면 new Date()는 반응성 변수가 아니므로 새로고침을 해도 시간이 변화하지 않는다고 한다. 
- computed는 이전의 값을 캐싱해놓고서 반응성 변수의 변화를 감지하여 변화하는 방식으로 작동하기 때문이다. 
```js
const now = computed(() => Date.now())
```
- computed가 주로 복잡한 로직을 계산하는 데 쓰인다고 하지만 늘 그런 것은 아니다. 아래 같이 간단하게도 쓸 수 있다.
```js
const getLoginUser = computed(() => {
    return loginUser.value
});
//짧게하면   
const getLoginUser = computed(() => loginUser.value);
```
## <span style="color:#802548">_4. props_</span>
- 아래와 같이 parent component에서 child component props를 보낼 수 있다. 
- :변수명=""는 v-bind directive이며, @close=""는 v-on directive다.
```html
<executive-seat :dialog="dialogExecutive" @close="dialogExecutive = false" />
```
- 그런데 parent component에서 props를 보내기 위해선 child component에서 우선 props의 type을 규정해야 한다. 
- custom interface는 아래와 같이 Object as PropType generics같은 형식으로 쓴다.
```js
props: {
    schedule: {
      type: Object as PropType<ScheduleResponse>,
      required: true,
    },
    dialog: {
      type: Boolean,
      default: false,
    },
    seatList: {
      type: Object as PropType<Array<Seat>>,
    },
  },
```
- child component가 받은 props는 template에서 그대로 쓸 수 있다. 
- 아래와 같이 받은 props를 그대로 다시 child component에게 물려주는 모습이다.
```html
  <Payment :seatCodeList="seatCodeList" :schedule="schedule" />
```
- 반면에 script에서 쓰려면 props로 써서 꺼내와야 한다. 
- props로 꺼내온 것들은 그냥 조합하면 반응성을 잃기 때문에 computed에 감싸서 새로운 값을 만든다.
```js
setup(props) {
const isOpen = computed(() => props.dialog);
const exeSeatList = computed(() => props.seatList?.filter((seat: Seat) => seat.seatClass.includes('EXECUTIVE')));
.
}
```
## <span style="color:#802548">_5. emit_</span>
- setter도 있는 computed의 경우 아래와 같이 쓸 수 있다. 
- 그리고 이 setter는 props.dialog의 값을 바꾸지 않는다. 
- 일단 props의 값을 바꾸는 것은 나쁜 선택이기 때문에 대개 emit으로 부모에 event를 보내어 처리하게 된다. 
- 마찬가지로 나도 그런 방식을 택했다.
```js
const isOpen = computed({
      // getter
      get() {
        return props.dialog;
      },
      //setter
      set() {
        proxy?.$emit('close');
      },
    });
```
- 그럼 아래와 같이 close event를 부모가 받아서 실행시킨다. 실행시킨 event는 v-dialog를 닫는 것이다.
```html
<executive-seat :dialog="dialogExecutive" @close="dialogExecutive = false" />
```



## <span style="color:#802548">_6. v-for_</span>
- v-for는 배열 안의 요소를 순회하며 배열의 갯수만큼 html을 반복하여 만든다. 
- tr에 v-for가 걸렸으므로 tr에서 /tr까지 scheduleList라는 배열의 length 만큼 html이 만들어진다. 
- 6개라면 6개의 tr인 셈이다.
```html
 <tr v-for="schedule in scheduleList" :key="schedule.scheduleNo">
        <td>{{ schedule.departStation }}</td>
        <td>{{ schedule.arriveStation }}</td>
        <td>{{ schedule.scheduleDepartDatetime }}</td>
        <td>{{ schedule.scheduleArriveDatetime }}</td>
        <td>{{ schedule.scheduleTrainNo }}</td>
        <td v-if="!isMoreThanAvailableSeat(schedule.seat as Seat[], 'EXECUTIVE')">
          <v-btn @click="selectExecutiveSeat(schedule)">좌석선택</v-btn>
        </td>
        <td v-else>매진</td>
        <td v-if="!isMoreThanAvailableSeat(schedule.seat as Seat[], 'STANDARD')">
          <v-btn @click="selectStandardSeat(schedule)"> 좌석선택</v-btn>
        </td>
        <td v-else>매진</td>
      </tr>
```

- 참고로 v-for는 아래와 같이 nested v-for도 가능하다. 
- seatItems라는 배열 안에 원소 안에 배열이 들어있다면 v-for로 또 반복시킬 수 있다. 
```js
//아래와 같은 배열이 있다고 해보자. 
  onMounted(() => {
      seatItems.value = [
        { title: '일반실 1호차', seatList: stFirstCarriageSeat },
        { title: '일반실 2호차', seatList: stSecondCarriageSeat },
      ];
      document.addEventListener('keydown', onEscapeKeydown);
    });
//이 배열을 1차로 먼저 돌린다. 그리고 나서 element의 seatList라는 배열을 또 돌린다.

 <v-row no-gutters v-for="element in seatItems" :key="element.title">
          <v-col>
            {{ element.title }}
            <v-checkbox
              v-for="seat in element.seatList"
              :disabled="seat.seatSold"
              :key="seat.seatNo"
              block
              class="text-none text-black mb-4"
              color="red-accent-2"
              :label="seat.seatNo"
              :variant="seat.seatSold ? 'flat' : 'outlined'"
              v-model="seatCodeList"
              :value="seat">
            </v-checkbox>
          </v-col>
        </v-row>
```

## <span style="color:#802548">_7. v-bind_</span>
​- v-bind는 특정한 변수를 binding시키는 것이다. 
위에 보면 :key가 보일 것이다. 바로 :가 v-bind의 약자다.
```html
 <tr v-for="schedule in scheduleList" :key="schedule.scheduleNo">
 ```
- :disabled, :label, :variant, :value도 모두 v-bind로서 변수명을 받아서 표시해준다. 
- 바로 아래와 같은 형식이다. 그럼 그 변수에 맞는 값이 들어가게 된다.


## <span style="color:#802548">_8. v-if_</span>
- v-if는 true이면 html을 rendering하고, v-if가 false면 rendering하지 않거나 v-else의 html을 rendering한다.
```html
<td v-if="!isMoreThanAvailableSeat(schedule.seat as Seat[], 'EXECUTIVE')">
    <v-btn @click="selectExecutiveSeat(schedule)">좌석선택</v-btn>
</td>
<td v-else>매진</td>
<td v-if="!isMoreThanAvailableSeat(schedule.seat as Seat[], 'STANDARD')">
    <v-btn @click="selectStandardSeat(schedule)"> 좌석선택</v-btn>
</td>
<td v-else>매진</td>
```
- 아래와 같은 함수가 있다면, 받아오는 parameter에 따라 true냐 false냐가 달라지게 된다. 
- 계산 값의 결과가 true이면 좌석선택이라는 text가 보이고, false면 v-else로 넘어가 매진이라는 text가 보이게 된다.
```js
function isMoreThanAvailableSeat(data: Array<Seat>, seatClass: string) {
      const executiveClassSeat = data.filter((seat: Seat) => seat.seatClass.includes(seatClass));
      const soldExecutiveClassSeat = executiveClassSeat.reduce((countSoldSeat, element) => (element.seatSold ? ++countSoldSeat : countSoldSeat), 0);
      const isMoreThanAvailableSeat = soldExecutiveClassSeat + sumHeadCount > executiveClassSeat.length;

      return isMoreThanAvailableSeat;
    }
```
- 실제로 값을 바꾸고 조회하기를 누르면 text가 변화하는 모습을 볼 수 있다.

 

## <span style="color:#802548">_9. v-on_</span>
- v-on은 event들을 의미하며 약자로 @를 사용한다. @click, @prevent 등이 대표적이다. 
- event기 때문에 event가 발생하면 발동할 함수를 넣는다.
```html
<v-btn @click="selectExecutiveSeat(schedule)">좌석선택</v-btn>
<v-btn type="button" @click="handleMemberSave"> 회원가입 </v-btn>
<v-btn type="button" @click="$router.back()"> 뒤로가기 </v-btn>
```
## <span style="color:#802548">_10. v-model_</span>
- v-model은 양방향 데이터 바인딩이다. 값을 입력하면 그 즉시 script에도 반영되며, 화면에도 반영된다. 
- 아래는 vuetify component들의 예시지만, vue를 사용한다면 순수 html에서도 v-model은 똑같이 작동한다. 
```html
  <v-text-field type="text" v-model="memberId" />
  <v-checkbox label="3" value="3" v-model="seatCodeList" />
<v-select label="결제방법" :items="['카드', '무통장', '네이버페이']" v-model="paymentType" />
```
## <span style="color:#802548">_10. v-model_</span>
??
​

## <span style="color:#802548">_11. onMounted_</span>
- onMounted는 dom이 생겨난 이후와 데이터가 입혀지기 전 사이에 쓰이는 lifecycle hook이다. 
- axios로 데이터를 받고, 받은 데이터를 모델링한다. axios로 데이터만 받는 경우도 흔하다. 
- 모델링은 필요할 때만 하는 것이다.
```js
onMounted(async () => {
      const result = (await getTicketList(loginMember.value)).data;
      ticketList.value = result;

      /**
       * @description: 데이터 재구조화
       */
      const valueWishedList = ticketList.value?.reduce((acc, current) => {
        acc.push({
          paymentNo: current['paymentNo'],
          trainNo: current['trainNo'],
          scheduleInfo: {
            arriveStation: current['arriveStationName'] as string,
            departStation: current['departStationName'] as string,
            scheduleDepartDatetime: current['departDateTime'],
            scheduleArriveDatetime: current['arriveDateTime'],
          },
          seatInfo: [current['seatResponse']] as Seat[],
        });

        return acc; //모아놓은 값.
      }, [] as customerViewingTicket[]);

      mergedTicketList.value = valueWishedList?.reduce((acc, cur) => {
        const obj = acc.find(obj => obj.paymentNo === cur.paymentNo);

        if (obj) {
          obj.seatInfo = [...obj.seatInfo, ...cur.seatInfo];
        } else {
          acc.push(cur);
        }

        return acc;
      }, [] as customerViewingTicket[]);
    });
```
- 또는 아래와 같이 onCreated  composition API에서는 setup()- 에서 만든 값을 할당하거나, 전역 listener를 달기도 한다.
```js
 setup(props) {
 const stFirstCarriageSeat = computed(() => props.seatList?.filter((seat: Seat) => seat.seatClass.includes('STANDARD') && seat.seatCarriage.includes('1')));
    const stSecondCarriageSeat = computed(() => props.seatList?.filter((seat: Seat) => seat.seatClass.includes('STANDARD') && seat.seatCarriage.includes('2')));
.
.
onMounted(() => {
      seatItems.value = [
        { title: '일반실 1호차', seatList: stFirstCarriageSeat },
        { title: '일반실 2호차', seatList: stSecondCarriageSeat },
      ];
      document.addEventListener('keydown', onEscapeKeydown);
    });
.
.
}
```

## <span style="color:#802548">_12. onBeforeUpdate_</span> 
​

- onBeforeUpdated()는 re-rendering되기 전에 발동되는 lifecycle hook이다. 
- 처음 rendering은 onMounted()의 영향을 받고 그 다음부터 생기는 re-rendering은 onBeforeUpdated()를 많이 활용하는 것으로 보인다. 

​

- 아래와 같이 특정 버튼을 누르면 값이 변화하여 새로운 rendering을 해야할 때, computed()는 활용할 수 없을 때쓴다.
- 아래를 computed로 하면 조회하기 버튼을 누르기도 전에 숫자만 select에서 바꿔도 바로 바뀐 결과가 나타난다. 
- 하지만 조회하기를 누를 때 바뀐 결과가 반영되어야 한다. 
- 따라서 computed는 활용할 수 없다. 그럴 때 onBeforeUpdate lifecycle hook을 활용한다.
```java
 let sumHeadCount = props.headcountInfo.headCountAdult + props.headcountInfo.headCountChild + (props.headcountInfo.headCountElder ?? 0);

onBeforeUpdate(() => {
      sumHeadCount = props.headcountInfo.headCountAdult + props.headcountInfo.headCountChild + (props.headcountInfo.headCountElder ?? 0);
    });
```