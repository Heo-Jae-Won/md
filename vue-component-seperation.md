## <span style="color:#802548">_1. 처음 조회 화면_</span>
​

 - v-for를 이용해 template code를 간소화할 배열을 만든다. 
 - v-for로 template을 반복해서 쓰는행위를 줄인다. v-for 안에 v-for를 쓸 수도 있다. 
 - 그를 위해 변주되는 요소만 담는 배열을 만든다.
```js
const headcountFilteringItems = [
  { label: '성인 사람 수', value: 'headCountAdult' },
  { label: '어린이 사람 수', value: 'headCountChild' },
  { label: '노인 사람 수', value: 'headCountElder' },
  { label: '중증 사람 수', value: 'headCountSevere' },
];
```
- 조회화면 코드를 반복해서 쓰지 않게 위의 배열을 활용해 v-for문을 활용한다.
```html
      <v-select v-for="(element, i) in headcountFilteringItems" :key="i" :label="element.label" :items="[1, 2, 3]" v-model="headcountInfo[element.value]"></v-select>
```
- 조회 화면은 아래와 같이 구성한다.
```html
    <v-app id="inspire">
      <v-main class="blue-grey lighten-4">
        <v-container class="mt-5" style="max-width: 700px" fill-height>
          <v-card>
            <div class="pa-15">
              <h1 style="text-align: center" class="mb-10">세부조회 화면</h1>
           .
           .
           .
                <v-col>
                  <v-sheet class="pa-2 ma-2">
                    <v-select v-for="(element, i) in headcountFilteringItems" :key="i" :label="element.label" :items="[1, 2, 3]" v-model="headcountInfo[element.value]"></v-select>
                  </v-sheet>
                </v-col>
.
.
.
      </v-main>

      <schedule-result :scheduleList="scheduleList" :headcountInfo="headcountInfo" />
    </v-app>
```
- 조회하기를 눌렀을 때 처음 일정 정보를 얻어온다. 
```js
async function fetchSchedule() {
      const response = await getScheduleList(scheduleInfo.value);
      scheduleList.value = response.data;
    }
```
- 조회하기를 누르면 일정 정보 조건 및 인원 수를 child component에 물려준다. 물려준 내용(props)는 2번에서 다룬다.
```js
function handleFetchSchedule() {
      changeScheduleInfo(scheduleInfo.value);
      changeHeadcount(headcountInfo.value);
      fetchSchedule();
}
```
아래와 같은 화면이다.


## <span style="color:#802548">_2. 세부 조회 화면_</span>
- 얻어 온 일정 정보는 위의 component가 아니라 child component에서 render된다. 
- 따라서 render된 값을 props로 내려준다. 
```html
  <schedule-result :scheduleList="scheduleList" :headcountInfo="headcountInfo" />
```
- 아래는 일정의 결과를 가져오는 child component다. 
```html
<template>
  <v-table>
    <tr>
      <td>출발역</td>
.
.
.
    </tr>
    <tr v-for="schedule in scheduleList" :key="schedule.scheduleNo">
      <td>{{ schedule.departStation }}</td>
      <td>{{ schedule.arriveStation }}</td>
      <td>{{ schedule.scheduleDepartDatetime }}</td>
      <td>{{ schedule.scheduleArriveDatetime }}</td>
      <td>{{ schedule.scheduleTrainNo }}</td>
      <td v-if="!isMoreThanAvailableSeat(schedule.seat, 'EXECUTIVE')">
        <executive-seat :schedule="schedule" />
      </td>
      <td v-else>매진</td>
      <td v-if="!isMoreThanAvailableSeat(schedule.seat, 'STANDARD')">
        <standard-seat :schedule="schedule" />
      </td>
      <td v-else>매진</td>
    </tr>
  </v-table>
</template>
```
- 위에서 가져온 scheduleList를 활용해서 예매가능 여부를 판단한다. 
- v-if의 경우, props를 받아왔을 때 true 혹은 false 값이 변경되면 다시 render된다. 
- 그를 위해 props는 v-model로 값을 보내줘야 한다. 
- 직접 array를 만들어서 push하는 것으로는 반응성이 제대로 작동하지 못한다. 따라서 아래와 같이 v-model로 설계하자.
```html
 <v-select v-for="(element, i) in headcountFilteringItems" :key="i" :label="element.label" :items="[1, 2, 3]" v-model="headcountInfo[element.value]"></v-select>
 ```
- 이제 받아온 props를 가지고 예매가 불가능하면 매진을, 예매가 가능하면 좌석 선택을 표시한다. 
- 위에 있는 조건을 맞추고 조회하기를 일정이 표시되고, 좌석 예매 가능 여부도 같이 판단한다.
```js
function isMoreThanAvailableSeat(data: Array<Seat>, seatClass: string) {
      const executiveClassSeat = data.filter((seat: Seat) => seat.seatClass.includes(seatClass));
      const soldExecutiveClassSeat = executiveClassSeat.reduce((countSoldSeat, element) => (element.seatSold ? ++countSoldSeat : countSoldSeat), 0);
      const isMoreThanAvailableSeat = soldExecutiveClassSeat + sumHeadCount > executiveClassSeat.length;

      return isMoreThanAvailableSeat;
    }
```


## <span style="color:#802548">_3. 좌석 선택 화면_</span>
- 좌석 component는 ScheduleResult의 child component다. 
- 좌석 선택 버튼을 누르게 되면 dialog가 true가 되면서 화면이 튀어나오게 된다.
```html
  <v-btn color="blue lighten-1 text-capitalize" depressed large block dark class="mb-3" @click="dialog = true">
    좌석선택

    <v-dialog v-model="dialog" @click:outside="clearSeatItems">
      <v-card>
        <v-card-text>
          <Schedule :schedule="schedule" />

          <v-row no-gutters>
            <v-col>
              특실 3호차
              <!--v-for에는 ref 같은 vue 변수를 집어넣어야 함. let 변수로 넣었더니 작동 안 함. 또한 v-if와 v-for는 같이 못 씀.-->
              <v-checkbox
                v-for="seat in exeSeatList"
                :key="seat.seatNo"
                block
                class="text-none text-black mb-4"
                color="red-accent-2"
                :label="seat.seatNo"
                :variant="seat.seatSold ? 'flat' : 'outlined'"
                :disabled="seat.seatSold"
                v-model="seatCodeList"
                :value="seat">
              </v-checkbox>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-actions>
          <v-btn-group class="mx-auto">
            <Payment :seatCodeList="seatCodeList" :schedule="schedule" />
            <v-btn color="secondary" @click="clearSeatItemsDialog">닫기</v-btn>
          </v-btn-group>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-btn>
```
- 팔린 좌석의 경우 팔렸다는 표시를 내고 체크되는 경우를 막기 위해 disabled 처리를 했다. 
- 5명의 예매 인원이 있다면, 5개의 좌석을 선택해야만 다음 결제 화면으로 넘어가는 것이 가능하다. 

## <span style="color:#802548">_4. 결제 화면_</span>
- 선택해서 결제화면으로 넘어가면, 선택한 좌석들을 확인할 수 있다. 
- 아래와 같이 선택한 수 만큼의 좌석이 표시된다. 
```html
  <v-btn color="primary" @click="checkSumHeadCount"
    >예매 정보 보기
    <v-dialog v-model="dialog">
      <v-card>
        <v-card-text>
          <v-row no-gutters>
            <v-col>
              요금: 총합<v-text-field v-model="pay.paymentCharge" readonly /> 결제방법: <v-select label="결제방법" :items="['카드', '무통장', '네이버페이']" v-model="paymentType"></v-select> 상품명:
              <v-text-field v-model="paymentItemName" readonly />
            </v-col>
          </v-row>
          기차번호: {{ schedule?.scheduleTrainNo }}
          <v-row no-gutters>
            <v-col v-for="seat in seatCodeList" :key="seat.seatCode">
              좌석등급: <v-text-field class="ma-2 pa-2" v-model="seat.seatClass" readonly /> 좌석호실: <v-text-field class="ma-2 pa-2" v-model="seat.seatCarriage" readonly /> 좌석번호:
              <v-text-field class="ma-2 pa-2" v-model="seat.seatNo" readonly />
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
      <v-btn color="secondary" @click="payment">결제하기</v-btn>
      <v-btn color="secondary" @click="dialog = false">닫기</v-btn>
    </v-dialog>
  </v-btn>
```

 

## <span style="color:#802548">_5. 티켓 확인_</span>
- 티켓을 확인할 수 있는 내 정보 창이 있다. 내 정보창에 들어가면 side tab으로 보이게 한다.
```html
    <v-navigation-drawer expand-on-hover rail>
      <v-list>
        <v-list-item prepend-avatar="https://randomuser.me/api/portraits/women/85.jpg" :title="loginMember"></v-list-item>
      </v-list>
      <v-divider></v-divider>
      <v-list density="compact" nav>
        <v-list-item prepend-icon="mdi-folder" title="쿠폰함" @click="$router.push('/coupon')"></v-list-item>
        <v-list-item prepend-icon="mdi-account-multiple" title="선물함" @click="$router.push('/gift')"></v-list-item>
        <v-list-item prepend-icon="mdi-star" title="티켓" @click="$router.push('/myTicket')"></v-list-item>
      </v-list>
    </v-navigation-drawer>
```
- 그럼 아래와 같은 화면이 연출된다. 왼쪽에 정렬되는데, 옵션으로 바꿀 수 있을 것으로 보인다. 나중에 찾아보려 한다.


- 그럼 이제 티켓 목록을 출력해야 한다. 
- 티켓 목록은 결제별로 묶어 하나의 티켓처럼 보이게 한다. 그를 위해서는 backend로부터 받은 data를 다시 재구조화해야 한다. reduce를 2번 사용하여 재구조화한다.  
```js
onMounted(async () => {
      const result = (await getTicketList(loginMember.value)).data;
      ticketList.value = result;
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
          SeatInfo: [current['seatResponse']] as Seat[],
        });
        return acc; //모아놓은 값.
      }, [] as customerViewingTicket[]);

      mergedTicketList.value = valueWishedList?.reduce((acc, cur) => {
        const obj = acc.find(obj => obj.paymentNo === cur.paymentNo);

        if (obj) {
          obj.seatInfo = [...obj.SeatInfo, ...cur.SeatInfo];
        } else {
          acc.push(cur);
        }
        return acc;
      }, [] as customerViewingTicket[]);
    });
```
- 위에서처럼 대괄호 표기법으로 가져오려면 index signature가 필요하다.
```js
export interface customerViewingTicket {
  paymentNo: string;
  trainNo: string;
  scheduleInfo: ScheduleResponse;
  seatInfo: Seat[];
  [key: string]: string | number | undefined | ScheduleResponse | Seat[];
}
```
- 처음 8개의 길이를 가진 array는 모델링 이전이다. 그 이후 원소가 4개인 array는 모델링 이후다. 


- 실제 내용은 아래와 같이 구성된다. 
- 이제 data modeling을 끝마쳤으니 template에 표시한다. 우선은 모델링된 ticket을 v-for로 rendering하고, 그 ticket 안의 좌석 배열을 다시 v-for로 rendering한다. 
```html
    <v-row justify="space-around">
      <v-card width="1600" title="티켓 좌석" subtitle="This is a subtitle" text="This is content">
        <v-col v-for="payment in mergedTicketList" :key="payment.paymentNo">
          기차번호: <v-text-field class="ma-2 pa-2" v-model="payment.trainNo" readonly /> 일정정보 :
          <v-sheet elevation="12" max-width="600" rounded="lg" width="100%" class="pa-4 text-center mx-auto">
            출발역: <v-text-field class="ma-2 pa-2" v-model="payment.scheduleInfo.departStation" readonly /> 도착역:
            <v-text-field class="ma-2 pa-2" v-model="payment.scheduleInfo.arriveStation" readonly /> 출발일시:
            <v-text-field class="ma-2 pa-2" v-model="payment.scheduleInfo.scheduleDepartDatetime" readonly /> 도착일시:
            <v-text-field class="ma-2 pa-2" v-model="payment.scheduleInfo.scheduleArriveDatetime" readonly />
          </v-sheet>
          좌석 갯수: {{ payment.SeatInfo.length }}
          <v-sheet elevation="12" max-width="600" rounded="lg" width="100%" class="pa-4 text-center mx-auto" v-for="seat in payment.SeatInfo" :key="seat.seatCode">
            좌석 등급: <v-text-field class="ma-2 pa-2" v-model="seat.seatClass" readonly /> 좌석 호차: <v-text-field class="ma-2 pa-2" v-model="seat.seatCarriage" readonly /> 좌석 코드:
            <v-text-field class="ma-2 pa-2" v-model="seat.seatNo" readonly />
          </v-sheet>
          <v-btn class="mx-auto" @click="handlePaymentUndo(payment.paymentNo, payment.SeatInfo)">결제 환불하기</v-btn>
        </v-col>
      </v-card>
    </v-row>
```
​
