<template>
    <v-navigation-drawer
    v-model="drawer"
    :rail="rail"
    permanent
    style="background-color: #FCFCFC;"
    @click="rail = false"
    >
    <v-list-item
        :prepend-avatar="profileImgUrl"
        :title="name"
        nav
    >
        <template v-slot:append>
        <v-btn
            icon="mdi-chevron-left"
            variant="text"
            @click.stop="rail = !rail"
        ></v-btn>
        </template>
    </v-list-item>

    <v-divider></v-divider>

    <v-list density="compact" nav>
        <v-list-item
        :prepend-avatar="myPage"
        title="마이페이지"
        value="myPage"
        @click="toMyPage"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="child"
        title="자녀관리"
        value="childConfig"
        @click="toMyChild"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="hospital"
        title="예약내역"
        value="reservation"
        @click="toMyReservation"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="review"
        title="리뷰내역"
        value="review"
        @click="toMyReview"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="calendar"
        title="우리아이 캘린더"
        value="calendar"
        @click="toCalendar"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="doctorQnA"
        title="My Q&A"
        value="doctorQnA"
        @click="toMyQnA"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="cs"
        title="고객센터"
        value="cs"
        @click="toChatList"
        ></v-list-item>

    </v-list>
    </v-navigation-drawer>
    <v-main style="height: 250px"></v-main>
</template>
<script>

export default{
    data(){
        return{
            drawer: true,
            rail: true,
            name: '',
            profileImgUrl: '',
            calendar: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/calendar-icon.png',
            child: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/child-icon.png',
            cs: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/cs_center_image.png',
            doctorQnA: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/doctor-qna-icon.png',
            myPage: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/mypage-icon.png',
            reservation: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/reservation-icon.png',
            review: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/review-icon.png',
            hospital: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/hospital-icon.png',
            doctors: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/214ff1d2-bd49-4766-9925-d6dea16d1139_doctor.png'
        }
    },
    created(){
        this.name = localStorage.getItem("name");
        this.profileImgUrl = localStorage.getItem("profileImgUrl");
        const role = localStorage.getItem("role");
        this.isDoctorAdmin = role === 'HospitalAdmin' || role === 'TodakAdmin';
    },
    methods:{
        toMyPage() {
            this.$router.push('/member/mypage');
        },
        toMyReservation() {
            this.$router.push('/member/mypage/reservation');
        },
        toMyChild() {
            this.$router.push('/member/child');
        },
        toCalendar() {
            this.$router.push('/member/mychild-cal');
        },
        toMyQnA() {
            // 아직 페이지 없어서 마이페이지로 라우팅
            this.$router.push('/member/mypage/myPost');
        },
        toMyReview() {
            this.$router.push('/member/myReviewList');
        },
        toChatList() {
            // this.$router.push('/chat');
            // ChatListComponent를 새로운 창에서 열기
            const chatWindow = window.open(
            '/chat/my-chat/list',  // ChatListComponent가 렌더링될 URL
            '_blank',  // 새로운 창을 열기 위한 옵션
            'width=500,height=800'  // 창의 크기를 지정
            );

            if (!chatWindow) {
            alert('팝업이 차단되었습니다. 팝업 차단 설정을 해제해주세요.');
            }
        },

    }
}
</script>
<style scoped>
  
</style>
