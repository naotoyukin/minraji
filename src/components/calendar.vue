<template>
<div>
  
    
  <main>
  

  <div id = "calendar-parameter">
    <FullCalendar 
      default-view="dayGridMonth"
      @eventClick="eventClick"
      :locale="locale"
      :header="calendarHeader"
      :weekends="calendarWeekends"
      :plugins="calendarPlugins"
      :events="calendarEvents"
      :selectable="calendarSelectable"
      
    />
  </div>

  <div class="p-modal" :class="{'is-open': isModalActive}">
    <p>ポップアップテスト</p>
    <p>{{roomtitle}}</p>
    <p><button v-on:click='openItem'>close</button></p>
  </div>
  <table>
    <td><router-link to="/"><button id="back-home-button">ホーム</button></router-link></td>
    <td><router-link to="/room"><button id="go-room-button">ルーム広場に戻る</button></router-link></td>
    <td><amplify-sign-out></amplify-sign-out></td>

  </table>
  </main>
</div>
</template>

<script>
  import API, {  graphqlOperation } from '@aws-amplify/api';
  // import calendar from "./calendar"
  import { listRooms } from '../graphql/queries';

  import UserStore from '../mobx/UserStore'

  import FullCalendar from '@fullcalendar/vue'
  import dayGridPlugin from '@fullcalendar/daygrid'
  import timeGridPlugin from "@fullcalendar/timegrid";
  import interactionPlugin from "@fullcalendar/interaction";
  import jaLocale from "@fullcalendar/core/locales/ja"; // 日本語化用
  

  export default {
    name: 'calendar',
    components: {
      FullCalendar // make the <FullCalendar> tag available
    },
    data() {
      return {
        userName: "",
        userId: "",
        members: [],
        error: "",
        rooms: [],
        locale: jaLocale, // 日本語化
        // カレンダーヘッダーのデザイン
        calendarHeader: {
          left: "prev,next today",
          center: "title",
          right: "dayGridMonth,Week,Day,listWeek"
        },
        calendarWeekends: true, // 土日を表示するか
        // カレンダーで使用するプラグイン
        calendarPlugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
        // カレンダーに表示するスケジュール一覧
        calendarEvents: [
          // TODO: データベースから取ってきたやつを反映できるようにする。
        ],
        calendarSelectable:true,
        isModalActive: false,
        roomtitle: ""
      }
    },
    methods: {
      async fetch(){
        //部屋データを取得
        const roomlimit = 100  // 表示する部屋の数のmax
        const rooms = await API.graphql(graphqlOperation(listRooms,{limit:roomlimit}))
        this.rooms = rooms.data.listRooms.items
        // console.log("in fetch")
        // console.log(this.rooms)
        this.addCalendarEvents(this.rooms)
      },
      addCalendarEvents(rooms){
        console.log("add eventの中")
        for (const room in rooms){
          console.log(rooms[0].id)
          console.log(rooms[0].tilte)
          console.log(rooms[0].createdAt)
          this.calendarEvents.push({
            title: rooms[room].tilte,
            start: new Date().getTime() //rooms[room].createdAt
            // start: rooms[room].createdAt
          });
        }
      },
      eventClick: function(event) {
        //クリックしたイベントのタイトルが取れるよ
        // alert('Clicked on: ' + item.title);
        this.roomtitle = event.text
        this.openItem()
        console.log(event)
        console.log(event.event.title)
      },
      openItem() {
        this.toggleModal();
      },
      /**
      * active状態を切り替える。
      */
      toggleModal() {
        this.isModalActive = ! this.isModalActive;
      },
    },
    async created(){
      const { username,userid } = await UserStore
      console.log(UserStore)
      this.userName = username
      this.userId = userid 
      this.fetch()
      // console.log("fetch finished!")
      // console.log("add eventの前")
      this.addCalendarEvents(this.rooms)
      // console.log("add eventの後")
    }
  }


</script>

<style lang='scss'>
  @import '~@fullcalendar/core/main.css';
  @import '~@fullcalendar/daygrid/main.css';
  .p-modal {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  overflow-y: auto;
  visibility: hidden;
  opacity: 0;
  z-index: -1;
  &.is-open {
    visibility: visible;
    opacity: 1;
    z-index: 100;
    position:fixed;
    top:0;
    left:0;
    width:100%;
    height:100%;
    background-color:rgba(0,0,0,0.5);
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

#calendar-parameter{
  width:100%;
 
}
</style>