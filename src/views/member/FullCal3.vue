<template>
  <v-container class="milcho-entire-container">
  <div>
    <!-- 자녀 리스트 -->
    <div>
      <v-row justify="center" align="center">
        <v-col
        v-for="child in childList"
        :key="child.id"
        cols="12"
        md="2"
        lg="1"
        class="d-flex flex-column align-items-center"
      >
        <v-spacer :style="{ height: '30px' }"></v-spacer>
        <v-avatar size="70">
          <v-img @click="handleChildClick(child)" :src="child.imageUrl" contain :style="selectedChildId === child.id ? '' : 'filter: brightness(0.4);'"/>
        </v-avatar>
        <v-text>{{ child.name }}</v-text>
      </v-col>
      </v-row>
    </div>
      <!-- 이벤트 및 예약 상세 정보 표시 영역 -->
      <!-- 일정 추가 버튼 -->
    <v-chip @click="startNewEvent" class="mb-3 milcho-add-bbtn" color="#C2D7FF" variant="flat" size="large"
     label>
      <span style="font-weight: bold; color: #00499E;">✍️일정추가</span>
    </v-chip>


    <v-container class="milcho-center-container">
      <!-- 캘린더 영역 -->
      <v-container class="milcho-calendar-container">
        <FullCalendar :options="calendarOptions" class="milcho-custom-calendar" />
      </v-container>


      <!-- <v-chip @click="startNewEvent" class="mb-3 ml-10" color="#C2D7FF" variant="flat">✏️</v-chip> -->
      <div class="ml-5"></div>
      <div class="ml-5"></div>

      <!-- <p class="ml-10"></p> -->
      <div class="milcho-details ml-5">
      <!-- 예약 상세 정보 -->
      <div v-if="selectedReservation && isReservationEvent">
        <v-avatar class="ma-2" style="min-height:200px; min-width:300px; border-radius: 10px; object-fit:cover;">
          <img :src="selectedReservation.hospitalImgUrl" style="width: 100%; height: 100%; object-fit: cover;" />
        </v-avatar>
        <div class="milcho-reservation-row">
          <strong>병원명</strong>
          <span>{{ selectedReservation.hospitalName }}</span>
        </div>
        <div class="milcho-reservation-row">
          <strong>의사명</strong>
          <span>{{ selectedReservation.doctorName }}</span>
        </div>
        <div class="milcho-reservation-row">
          <strong>환자명</strong>
          <span>{{ selectedChildName }}</span>
        </div>
        <div class="milcho-reservation-row">
          <strong>예약자</strong>
          <span>{{ selectedReservation.memberName }}</span>
        </div>
        <div class="milcho-reservation-row">
          <strong>진료타입</strong>
          <span>{{ selectedReservation.medicalItem }}</span>
        </div>
        <div class="milcho-reservation-row">
          <strong>예약 날짜</strong>
          <span>{{ selectedReservation.reservationDate }} {{ selectedReservation.reservationTime }}</span>
        </div>
      </div>

      <!-- 사용자 이벤트 상세 정보 또는 생성 양식 -->
      <div v-else-if="selectedEvent && !isReservationEvent || isEditing ">
        <!-- 이벤트 디테일 보기 -->
        <div v-if="!isEditing">
          <div class="milcho-reservation-row">
            <strong>제목</strong>
            <span>{{ formData.title }}</span>
          </div>
          <div class="milcho-reservation-row">
            <strong>내용</strong>
            <span class="milcho-custom-span">{{ formData.content }}</span>
          </div>
          <div class="milcho-reservation-row">
            <strong>타입</strong>
            <span>{{ translateType(formData.type) }}</span>
          </div>
          <div class="milcho-reservation-row">
            <strong>시작일</strong>
            <span>{{ formData.startDateText }}</span>
          </div>
          
          <div class="milcho-reservation-row">
            <strong>종료일</strong>
            <span>{{ formData.endDateText }}</span>
          </div>
          <v-btn class="milcho-btn-edit" @click="isEditing = true">수정하기</v-btn>
        </div>

        <!-- 이벤트 수정 및 생성 폼 -->
        <v-form ref="form" v-if="isEditing">
          <v-text-field v-model="formData.title" variant="underlined" label="제목" required />
          <v-textarea
            v-model="formData.content"
            label="내용"
            rows="5"
            outlined  
            class="milcho-custom-textarea"
          />
          <v-select
            v-model="formData.type"
            :items="eventTypes"
            item-title="name"
            item-value="type"
            label="타입"
            required
          />
          <v-menu
            v-model="menuStart"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-x="true"
            min-width="auto"
          >
            <template v-slot:activator="{ attrs }">
              <v-text-field v-model="formData.startDateText" label="시작일" readonly v-bind="attrs" @click.stop="menuStart = true" />
            </template>
            <v-date-picker v-model="formData.startDate" @input="updateStartDate" show-current="true" scrollable class="milcho-custom-picker">
              <template v-slot:actions>
                <div class="milcho-center-align">
                <v-btn text color="primary" @click="confirmStartDate">확인</v-btn>
                </div>
              </template>
            </v-date-picker>
          </v-menu>

          <v-menu
            v-model="menuEnd"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-x="true"
            min-width="auto"
          >
            <template v-slot:activator="{ attrs }">
              <v-text-field v-model="formData.endDateText" label="종료일" readonly v-bind="attrs" @click.stop="menuEnd = true" />
            </template>
            <v-date-picker v-model="formData.endDate" @input="updateEndDate" show-current="true" scrollable 
            class="milcho-custom-picker">
              <template v-slot:actions>
                <v-btn text color="primary" @click="confirmEndDate">확인</v-btn>
              </template>
            </v-date-picker>
          </v-menu>

          <div class="button-container">
            <v-btn v-if="isEditing" class="milcho-btn-delete" elevation="0" @click="deleteEvent">삭제</v-btn>
            <v-divider vertical class="milcho-vertical-divider"></v-divider>
            <v-btn class="milcho-btn-save" elevation="0" @click="handleSaveEvent">저장</v-btn>
        </div>
        
        </v-form>
      </div>

      <!-- 아무것도 선택되지 않은 경우 -->
      <div v-else>
        <p>선택된 예약 또는 이벤트가 없습니다.</p>
      </div>
    </div>
    <!-- </div> -->
  </v-container>
  </div>
      
</v-container>
  <MyPageSideBar/>
</template>

<script>
import FullCalendar from '@fullcalendar/vue3';
import dayGridPlugin from '@fullcalendar/daygrid';
import timeGridPlugin from '@fullcalendar/timegrid'; // 시간 그리드 플러그인 추가
import interactionPlugin from '@fullcalendar/interaction';
import axios from 'axios';
import MyPageSideBar from "@/components/sidebar/MyPageSideBar.vue";

export default {
  components: {
    FullCalendar,
    MyPageSideBar
  },
  data() {
    return {
      calendarOptions: {
        plugins: [dayGridPlugin, interactionPlugin, timeGridPlugin],
        initialView: 'dayGridMonth',
        headerToolbar: {
        left: 'prev,next today',
        center: 'title',
        right: 'dayGridMonth,timeGridWeek,timeGridDay' // 월별, 주별, 일별 보기 버튼 추가
      },
      views: {
        dayGridMonth: { // 월별 뷰 설정
          buttonText: '월별'
        },
        timeGridWeek: { // 주별 뷰 설정
          buttonText: '주별'
        },
        timeGridDay: { // 일별 뷰 설정
          buttonText: '일별'
        }
      },
        events: [],
        editable: true,
        dateClick: this.handleDateClick,
        eventClick: this.handleEventClick,
        displayEventTime: true,
        eventDrop: this.handleEventDrop, // 이벤트 드래그
        navLinks: true,
        dayCellClassNames: (arg) => {
          const day = arg.date.getDay();
          if (day === 0) {
          return 'fc-sunday';
          } else if (day === 6) {
          return 'fc-saturday';
          } else {
          return 'fc-weekday';
          }
      },
      dayHeaderClassNames: (arg) => {
          const day = arg.date.getDay();
          if (day === 0) {
          return 'fc-sunday-header';
          } else if (day === 6) {
          return 'fc-saturday-header';
          } else {
          return 'fc-weekday-header';
          }
      },
      datesSet: this.handleDatesSet,
      locale: 'ko', // 한국어 로케일 설정
      },
      isModalOpen: false,
      isEditing: false,
      eventTypes: [
          { name: '진행 중', type: 'IN_PROGRESS' },
          { name: '완료됨', type: 'COMPLETED' }
      ],
      formData: {
        id: null,
        title: '',
        content: '',
        startDate: new Date(),
        endDate: new Date(),
        startDateText: '',
        endDateText: '',
        type: '',
      },
      holidays: [], // 공휴일 초기화
      menuStart: false,
      menuEnd: false,
      childList: [],
      reservationList: [],
      userEvents: [],
      selectedReservation: null,
      selectedEvent: null,
      isReservationEvent: false,
      selectedChildName: '',
      selectedChildId: null,
      selectedChild: null,
      colorPalette: ['#FF9800', '#4CAF50', '#FFC107', '#2196F3', '#9C27B0', '#E91E63', '#00BCD4', '#3F51B5', '#8BC34A', '#FF5722'],
      colorIndex: 0,
      memberColors: {},
    };
  },
methods: {
  translateType(englishItem) {
    const itemMap = {
      'IN_PROGRESS': '진행 중',
      'COMPLETED': '완료',
      // 추가적인 타입들을 여기에 넣어주세요
    };
    return itemMap[englishItem] || englishItem;  // 매핑되지 않은 값은 그대로 출력
  },
  handleDatesSet(info) {
  const startDate = info.start.toISOString(); // 시작일 ISO 포맷
  const endDate = info.end.toISOString();     // 종료일 ISO 포맷

  this.fetchHolidays(startDate, endDate);
  if (this.selectedChild) {
      this.fetchReservationList(this.selectedChild, startDate, endDate);
  }
  },


  fetchHolidays(startDate, endDate) {
      axios
          .get(`${process.env.VUE_APP_API_BASE_URL}/member-service/api/google-calendar-holidays`, {
          params: {
              start: startDate,
              end: endDate,
          },
          })
      .then((response) => {
          // 공휴일 데이터를 저장
          this.holidays = response.data.items.map(korea => ({
              title: korea.summary,
              start: korea.start.date || korea.start.dateTime,
              end: korea.end.date || korea.end.dateTime,
              allDay: true,
              backgroundColor: 'red',
              editable: false,
              extendedProps: {
                  isHoliday: true,
              }
          }));

          // 공휴일을 가져온 후 이벤트 결합
          this.combineEvents();
      })
      .catch((error) => {
          console.error('Error fetching holidays:', error);
      });
  },



  fetchChildList() {
  axios
      .get(`${process.env.VUE_APP_API_BASE_URL}/member-service/child/`)
      .then((response) => {
          this.childList = response.data.result;
      })
      .catch((error) => {
          console.error('Error fetching child list:', error);
      });
  },
      
  fetchReservationList(child) {
      console.log(child)
      this.selectedChildName = child.name;
      this.colorIndex = 0;
      this.memberColors = {};

      axios
          .get(`${process.env.VUE_APP_API_BASE_URL}/reservation-service/reservation/list/child/${child.id}`)
          .then((response) => {
              const reservationEvents = response.data.map((reservation) => {
                  let memberName = reservation.memberName;

                  if (!this.memberColors[memberName]) {
                      this.memberColors[memberName] = this.colorPalette[this.colorIndex];
                      this.colorIndex = (this.colorIndex + 1) % this.colorPalette.length;
                  }

                  return {
                      title: `${reservation.hospitalName} - ${reservation.medicalItem}`,
                      start: new Date(`${reservation.reservationDate}T${reservation.reservationTime}`),
                      end: new Date(`${reservation.reservationDate}T${reservation.reservationTime}`),
                      backgroundColor: this.memberColors[memberName],
                      allDay: false,
                      editable: false,
                      extendedProps: {
                          ...reservation,
                          isReservationEvent: true,
                      },
                  };
              });
              this.reservationList = reservationEvents;

              // 예약 리스트를 가져온 후 이벤트 결합
              this.combineEvents(); 
          })
          .catch((error) => {
              console.error('Error fetching reservation list:', error);
          });
  },
    fetchUserEvents() {
      axios
        .get(`${process.env.VUE_APP_API_BASE_URL}/member-service/event/list`)
        .then((response) => {
          const userEvents = response.data.content.map((event) => ({
            id: event.id,
            title: event.title,
            start: new Date(event.startTime),
            end: new Date(event.endTime),
            backgroundColor: event.type === 'COMPLETED' ? '#D3D3D3' : '#0075FF',
            border: 'none',
            extendedProps: {
              content: event.content,
              type: event.type,
              isReservationEvent: false,
            },
          }));
          this.userEvents = userEvents;
          this.combineEvents();
        })
        .catch((error) => {
          console.error('Error fetching user events:', error);
        });
    },


  combineEvents() {
  // 공휴일, 사용자 이벤트, 예약 리스트를 모두 결합
  this.calendarOptions.events = [...this.holidays, ...this.userEvents, ...this.reservationList];
  
  this.$nextTick(() => {
      if (this.$refs.fullCalendar) {
          this.$refs.fullCalendar.getApi().refetchEvents();
      }
  });
  },


  handleChildClick(child) {
  // 자녀 선택 후 selectedChild에 저장
      this.selectedChild = child;
      this.selectedChildId = child.id; // 클릭한 자녀의 ID 저장
      this.fetchReservationList(child);
  },


    handleEventClick(info) {
      const isReservationEvent = info.event.extendedProps.isReservationEvent;
      const isHoliday = info.event.extendedProps.isHoliday;

      // 공휴일일 경우 아무 동작도 하지 않음
      if (isHoliday) {
      return;  // 클릭 이벤트를 무시
      }

      if (isReservationEvent) {
        this.selectedReservation = info.event.extendedProps;
        this.isReservationEvent = true;
      } else {
        this.isEditing = false; // 처음엔 수정모드가 아님
        this.selectedEvent = info.event.extendedProps;
        this.formData.id = info.event.id;
        this.formData.title = info.event.title;
        this.formData.content = info.event.extendedProps.content;
        this.formData.startDate = info.event.start;
        this.formData.endDate = info.event.end;
        this.formData.startDateText = this.formatDate(info.event.start);
        this.formData.endDateText = this.formatDate(info.event.end);
        this.formData.type = info.event.extendedProps.type || '';
        this.isReservationEvent = false;
      }
    },

    handleDateClick(info) {
      this.isEditing = false;
      this.formData.startDate = info.date;
      this.formData.endDate = info.date;
      this.formData.startDateText = this.formatDate(info.date);
      this.formData.endDateText = this.formatDate(info.date);
      this.selectedEvent = null;
      this.isReservationEvent = false;
    },

    formatDate(date) {
      const d = new Date(date);
      let month = '' + (d.getMonth() + 1);
      let day = '' + d.getDate();
      const year = d.getFullYear();

      if (month.length < 2) month = '0' + month;
      if (day.length < 2) day = '0' + day;

      return [year, month, day].join('-');
    },

    confirmStartDate() {
      this.updateStartDate(this.formData.startDate);
      this.menuStart = false;
    },

    confirmEndDate() {
      this.updateEndDate(this.formData.endDate);
      this.menuEnd = false;
    },

    updateStartDate(val) {
      this.formData.startDate = val;
      this.formData.startDateText = this.formatDate(val);
    },

    updateEndDate(val) {
      this.formData.endDate = val;
      this.formData.endDateText = this.formatDate(val);
    },

    handleSaveEvent() {
      const url = this.isEditing && this.formData.id 
        ? `${process.env.VUE_APP_API_BASE_URL}/member-service/event/update/${this.formData.id}`
        : `${process.env.VUE_APP_API_BASE_URL}/member-service/event/create`;

      const payload = {
        title: this.formData.title,
        content: this.formData.content,
        startTime: `${this.formData.startDateText}T00:00:00`,
        endTime: `${this.formData.endDateText}T23:59:59`,
        type: this.formData.type,
      };

      axios({
        method: 'post',
        url: url,
        data: payload,
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`,
        },
      })
        .then(() => {
          this.fetchUserEvents(); // 이벤트 저장 후 사용자 이벤트 새로고침
          this.isEditing = false; // 수정 모드를 해제하고 다시 디테일을 보여줌
        })
        .catch((error) => {
          console.error('Error saving event:', error);
        });
    },

    deleteEvent() {
      const id = this.formData.id;

      axios
        .get(`${process.env.VUE_APP_API_BASE_URL}/member-service/event/delete/${id}`)
        .then(() => {
          this.fetchUserEvents(); // 삭제 후 사용자 이벤트 새로고침
        })
        .catch((error) => {
          console.error('Error deleting event:', error);
        });
    },

    startNewEvent() {
      this.isEditing = true;
      this.formData = {
        id: null,
        title: '',
        content: '',
        startDate: new Date(),
        endDate: new Date(),
        startDateText: this.formatDate(new Date()),
        endDateText: this.formatDate(new Date()),
        type: '',
      };
      this.selectedEvent = null;
      this.isReservationEvent = false;
    },

  handleEventDrop(info) {
      const eventId = info.event.id;  // 이벤트 ID
      const newStartDate = info.event.startStr.split('+')[0];  // 새로운 시작 날짜에서 오프셋 제거
      const newEndDate = info.event.endStr ? info.event.endStr.split('+')[0] : newStartDate;  // 종료 날짜
  
  // 서버로 업데이트 요청
  axios({
    method: 'post',
    url: `${process.env.VUE_APP_API_BASE_URL}/member-service/event/update/${eventId}`,
    data: {
      startTime: newStartDate,  // 새로운 시작 날짜
      endTime: newEndDate,  // 새로운 종료 날짜
    },
    headers: {
      Authorization: `Bearer ${localStorage.getItem('token')}`,
    },
  })
  .then(() => {
    this.fetchUserEvents();  // 이벤트를 다시 불러옴
  })
  .catch((error) => {
    console.error('Error updating event:', error);
    // 이벤트 드래그가 실패하면 원래 위치로 되돌림
    info.revert();
  });
},
  },

  mounted() {
      this.fetchUserEvents(); 
      this.fetchChildList(); 
  },
};
</script>

<style>
.milcho-entire-container {
   position: relative; /* 부모 요소 고정 */
  max-width: 1500px !important;
  margin: 0 auto !important;
  width: 100% !important;
  display: flex;
  justify-content: center; /* 수평 중앙 정렬 */
  flex-direction: column; /* 자녀 선택 리스트가 위에 나오도록 변경 */
  align-items: center; /* 중앙 정렬 */
}

.milcho-center-container {
  display: flex;
  justify-content: space-between; /* 캘린더와 디테일 사이 공간 유지 */
  align-items: flex-start;
  width: 100%;
  max-width: 1400px;
  margin-top: 20px;
  gap: 40px; /* 캘린더와 디테일 사이에 공간 추가 */
  position: relative; /* 자식 요소가 absolute로 배치될 수 있도록 설정 */
}

.milcho-v-row {
  width: 100%;
  max-width: 1200px;
  margin-bottom: 20px;
  display: flex;
  justify-content: center;
  flex-wrap: wrap; /* 자녀 리스트가 여러 줄로 보여지도록 함 */
  height: auto;
  overflow-y: auto;
}

/* Calendar 관련 스타일 */
.milcho-calendar-container {
  flex: 3;
  max-width: 1000px; /* 달력의 최대 너비를 더 넓게 조정 */
  height: 900px;
  min-width: 600px; /* 최소 너비도 더 넓게 설정 */
  min-height: 800px;
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.milcho-custom-calendar {
  width: 800px; /* 달력의 너비를 넓게 조정 */
  height: 900px;
  min-width: 750px;
  min-height: 800px;
  margin-left: 250px;
}

.milcho-custom-calendar .fc-h-event {
  border: none !important; /* 테두리 완전히 제거 */
}

/* Detail 관련 스타일 */
.milcho-details {
  flex: 1;
  max-width: 350px;
  min-width: 350px;
  width: 400px;
  height: 800px;
  border: 1px solid #ddd;
  background-color: #f9f9f9;
  border-radius: 10px;
  margin-top: 100px;
  margin-right: 30px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.milcho-reservation-row {
  display: flex;
  margin-bottom: 10px;
  margin-left: 20px;
}

.milcho-reservation-row strong {
  width: 100px;
}

.button-container {
  display: flex;
  align-items: center; /* 수직 가운데 정렬 */
}

.milcho-btn-save {
background-color: #C2D7FF !important;
color: #00499e;
width: 50px;
height: 44px;
margin: 10px 20px 40px 20px;
border-radius: 20px;
font-family: 'Inter';
font-weight: 700;
font-size: 20px;
display: flex;
justify-content: center;
align-items: center;
}

.milcho-btn-delete {
background-color: #FFAFAF !important;
color: #650101;
width: 50px;
height: 44px;
margin: 10px 20px 40px 60px;
border-radius: 20px;
font-family: 'Inter';
font-weight: 700;
font-size: 20px;
display: flex;
justify-content: center;
align-items: center;
}

.milcho-vertical-divider {
width: 1px !important;
height: 44px !important;
background-color: black !important;
margin: 0 10px !important;
}

.milcho-btn-edit {
  background-color: #c2d7ff !important;
  margin-left: 135px;
  margin-top: 10px;
}

.milcho-btn-add {
  position: absolute; /* 고정된 위치로 변경 */
  top: 30px; /* 위치 조정 */
  right: 30px; /* 오른쪽으로 이동 */
  background-color: #ECF2FD !important;
  width: 50px;
  height: 50px;
  min-width: 50px !important;
  min-height: 50px !important;
  border-radius: 50% !important;
  display: flex;
  justify-content: center;
  align-items: center;
}

.milcho-custom-picker {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-left: 700px;
  margin-top: 200px;
}

.milcho-center-align {
  display: flex;
  justify-content: center;
  width: 100%;
}

.milcho-custom-span {
  display: block;
  width: 100%; /* 부모 요소의 너비에 맞춰줍니다 */
  word-wrap: break-word; /* 긴 단어가 있을 경우 자동 줄바꿈 */
  white-space: normal; /* 텍스트가 자동으로 줄바꿈되도록 설정 */
  overflow: hidden; /* 박스를 넘는 텍스트를 숨깁니다 */
  margin-left: 25px; /* 기본 여백으로 설정 */
  padding: 10px; /* 텍스트와 박스 간의 여백을 추가 */
  max-height: 150px; /* content 영역에 최대 높이를 설정하여 너무 길지 않게 제한 */
  overflow-y: auto; /* 너무 긴 내용일 경우 스크롤이 생기게 설정 */
}

/* FullCalendar Styles */
.fc-weekday .fc-daygrid-day-number {
  color: black;
}

.fc-saturday .fc-daygrid-day-number {
  color: blue;
}

.fc-sunday .fc-daygrid-day-number {
  color: red;
}

.fc-weekday-header .fc-col-header-cell-cushion {
  color: black !important;
}

.fc-saturday-header .fc-col-header-cell-cushion {
  color: blue !important;
}

.fc-sunday-header .fc-col-header-cell-cushion {
  color: red !important;
}
.milcho-add-bbtn{
  position: absolute; /* 절대 위치 설정 */
  top: 7%; /* 부모 높이가 1000px일 때 */
  left: 71.3%; /* 부모 너비가 1920px일 때 */
  z-index: 10; /* 다른 요소 위에 배치 */
}

/* 작은 화면에 맞춰 조정 */
@media (max-width: 768px) {
  .milcho-center-container {
    flex-direction: column;
    align-items: center;
  }
  
  .milcho-calendar-container {
    max-width: 100%;
    min-width: 100%;
  }
  
  .milcho-details {
    max-width: 100%;
    margin-top: 20px;
    min-height: auto;
  }
  
  .milcho-btn-add {
    right: 20px; /* 오른쪽 고정 */
    top: 10px; /* 상단 고정 */
  }
}

</style>
