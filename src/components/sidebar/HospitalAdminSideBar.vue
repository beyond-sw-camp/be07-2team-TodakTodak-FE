<template>
    <v-navigation-drawer
    v-model="drawer"
    :rail="rail"
    permanent
    style="background-color: #CEE4FF;"
    @click="rail = false"
    >
    <v-list-item nav>
        <template v-slot:prepend>
            <v-avatar size="40" style="background-color: white;">
                <v-img :src="profileImgUrl" alt="Profile image"></v-img>
            </v-avatar>
        </template>

        <v-list-item-title class="custom-avatar">{{ name }}</v-list-item-title>

        <template v-slot:append>
            <v-btn icon="mdi-chevron-left" variant="text" @click.stop="rail = !rail"></v-btn>
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
        :prepend-avatar="reservation"
        title="예약목록"
        value="reservation"
        @click="toReservation"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="doctor"
        title="의사등록"
        value="doctor"
        @click="toRegisteDoctor"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="hospital"
        title="병원수정"
        value="hospital"
        @click="toModifyHospital"
        ></v-list-item>

        <v-list-item
        :prepend-avatar="clock"
        title="영업시간등록/수정"
        value="cs"
        @click="toRegisterModify"
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
            doctor: 'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/doctor-3d-image.png',
            clock:'https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/operating-hours.png',
        }
    },
    created(){
        this.name = localStorage.getItem("name");
        this.profileImgUrl = localStorage.getItem("profileImgUrl");
    },
    methods:{
        toMyPage() {
            this.$router.push('/member/mypage');
        },
        toReservation() {
            this.$router.push('/member/doctor/reservation');
        },
        toRegisteDoctor(){
            this.$router.push('/hospital/doctor/register');
        },
        toModifyHospital(){
            this.$router.push('/hospital/admin/detail');
        },
        toRegisterModify(){
            this.$router.push('/hospital/admin/time');
        }
    }
}
</script>
<style scoped>
  
</style>
