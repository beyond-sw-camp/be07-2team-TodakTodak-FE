<template>
    <v-container style="width: 1200px;">
        <v-container style="width: 700px;">
            <v-spacer :style="{ height: '50px' }"></v-spacer>
            <v-row class="header-row">
                <h2>{{ this.hospitalName }}</h2>
                <v-chip class="schedule-chip ml-3" variant="flat" size="x-large" label>스케쥴 예약</v-chip>
            </v-row>
            <v-row class="header-row">
                <v-col class="big-font">
                    <div class="no-wrap">
                        진료대상 선택
                    </div>
                    <div class="custom-text2">
                        자녀 추가
                    </div>
                </v-col>
            </v-row>
            <v-row>
                <div class="small-font inter-normal">실제 진료받을 자녀를 선택해주세요</div>
            </v-row>
            <v-row>
                <!-- childOptions 배열에 있는 아이들을 반복 렌더링 -->
                <v-col v-for="(child, index) in childOptions" :key="index">
                    <div class="child button-cursor" @click="addChild(child)"
                        :class="{ 'selected-child': this.child == child }">
                        <v-row justify="center">
                            <v-col class="text-center" cols="3">
                                <img :src="child.imageUrl" alt="child image"
                                    style="width: 50px; height: 50px; border-radius:30px;">
                            </v-col>
                            <v-col class="text-center" cols="5" style="margin-top: 11px;">
                                <v-row class="inter-bold child-name">{{ child.name }}</v-row>
                                <v-row class="inter-normal child-ssn">{{ child.ssn }}</v-row>
                            </v-col>
                            <v-col cols="4">
                                <div class="mini-button inline" v-if="this.child == child" style="margin-top: 5px;">선택됨
                                </div>
                            </v-col>
                        </v-row>
                    </div>
                </v-col>
            </v-row>
            <v-row class="header-row">
                <v-col class="big-font">
                    의사 선택
                </v-col>
            </v-row>
            <v-row justify="center">
                <div v-for="(doctor, index) in doctorList" :key="index" class="doctor button-cursor"
                    @click="addDoctor(doctor)" :class="{ 'selected-doctor': this.doctor == doctor }">
                    <v-row>
                        <v-col cols="2">
                            <v-row class="ml-4">
                                <img :src=doctor.profileImgUrl alt="doctor image"
                                    style="width: 65px; height: 65px; border-radius: 30px;">
                            </v-row>
                        </v-col>
                        <v-col cols="2">
                            <v-row class="inter-bold inline">{{ doctor.name }} 원장</v-row>
                        </v-col>
                        <v-col>
                        </v-col>
                        <v-col cols="2">
                            <div class="mini-button" v-if="this.doctor == doctor">선택됨</div>
                        </v-col>
                    </v-row>
                </div>
            </v-row>
            <v-row v-if="doctor" class="header-row">
                <v-col class="big-font">
                    날짜 선택
                </v-col>
            </v-row>
            <v-row v-if="doctor" justify="start" class="ml-2">
                <p style="color: red">{{ this.error }}</p>
                <v-date-picker v-model="date" :allowed-dates="allowedDates" @input="updateDate" :width="650"
                    style="border-radius: 10px;" color="#C2D7FF">
                </v-date-picker>
            </v-row>
            <v-row v-if="date" class="header-row">
                <v-col class="big-font">
                    시간 선택
                </v-col>
            </v-row>
            <v-row v-if="date">
                <v-col v-for="time in doctorTimeSlots" :key="time" class="time-slot" :class="getTimeClass(time)"
                    @click="selectTime(time)" style="flex-basis: 20%; max-width: 20%;">
                    {{ time }}
                </v-col>
            </v-row>
            <v-row class="header-row">
                <v-col class="big-font">
                    진료 항목
                </v-col>
            </v-row>
            <v-row class="mt-1 ml-1">
                <v-col cols="2">
                    <v-row class="inter-bold before-selected inline button-cursor"
                        :class="{ 'selected-medi': mediItem == '일반진료' }" @click="addmediItem('일반진료')"> 일반진료 </v-row>
                </v-col>
                <v-col cols="2">
                    <v-row class="inter-bold before-selected inline button-cursor"
                        :class="{ 'selected-medi': mediItem == '예방접종' }" @click="addmediItem('예방접종')">예방접종 </v-row>
                </v-col>
            </v-row>
            <v-row class="header-row">
                <v-col class="big-font">
                    증상 선택
                </v-col>
            </v-row>
            <v-row>
                <div class="mt-n5 ml-5">
                    <v-chip-group v-if="symptoms.length > 0">
                        <v-chip v-for="(symptom, index) in symptoms" :key="index" class="mr-2" size="large"
                            variant="flat" style="color:#00499E; background-color:#ECF2FD; font-weight:bold;">
                            {{ symptom }}
                        </v-chip>
                    </v-chip-group>
                </div>
            </v-row>
            <v-row class="ml-1">
                <div class="mini-button button-cursor inter-bold mt-3" @click="symptomsModal = true">증상선택하기</div>
            </v-row>
            <v-row>
                <div class="inter-bold dark-blue subtitle mt-6 ml-5">의사선생님께 하고싶은 말</div>
            </v-row>
            <v-row><textarea class="text ml-4" v-model="comment"></textarea></v-row>
            <v-row class="mt-6 ml-1">
                <div class="button  inter-bold" @click="reserved">스케줄예약 신청</div>
            </v-row>

            <v-dialog v-model="symptomsModal" max-width="700px">
                <v-card>
                    <v-card-title class="subtitle dark-blue inter-bold text-center">
                        증상 선택
                    </v-card-title>
                    <v-card-text>
                        <v-container>
                            <!-- 호흡기 -->
                            <v-row justify="center" class="mb-2"> <!-- 위 아래 여백 줄임 -->
                                <v-col cols="11">
                                    <div class="inter-bold subtitle-2 dark-blue">호흡기</div>
                                </v-col>
                                <v-col cols="auto" v-for="(symptom, index) in symptomsOptions.respiratory"
                                    :key="'respiratory-' + index">
                                    <div class="inter-bold symptom-box"
                                        :class="symptoms.includes(symptom) ? 'selected-symptom' : 'symptom'"
                                        @click="addSymptom(symptom)">
                                        {{ symptom }}
                                    </div>
                                </v-col>
                            </v-row>

                            <!-- 소화기 -->
                            <v-row justify="center" class="mb-2"> <!-- 위 아래 여백 줄임 -->
                                <v-col cols="11">
                                    <div class="inter-bold subtitle-2 dark-blue">소화기</div>
                                </v-col>
                                <v-col cols="auto" v-for="(symptom, index) in symptomsOptions.digestive"
                                    :key="'digestive-' + index">
                                    <div class="inter-bold symptom-box"
                                        :class="symptoms.includes(symptom) ? 'selected-symptom' : 'symptom'"
                                        @click="addSymptom(symptom)">
                                        {{ symptom }}
                                    </div>
                                </v-col>
                            </v-row>

                            <!-- 피부 및 알레르기 -->
                            <v-row justify="center" class="mb-2"> <!-- 위 아래 여백 줄임 -->
                                <v-col cols="11">
                                    <div class="inter-bold subtitle-2 dark-blue">피부 및 알레르기</div>
                                </v-col>
                                <v-col cols="auto" v-for="(symptom, index) in symptomsOptions.skinAllergy"
                                    :key="'skinAllergy-' + index">
                                    <div class="inter-bold symptom-box"
                                        :class="symptoms.includes(symptom) ? 'selected-symptom' : 'symptom'"
                                        @click="addSymptom(symptom)">
                                        {{ symptom }}
                                    </div>
                                </v-col>
                            </v-row>

                            <!-- 전신 / 기타 -->
                            <v-row justify="center" class="mb-2"> <!-- 위 아래 여백 줄임 -->
                                <v-col cols="11">
                                    <div class="inter-bold subtitle-2 dark-blue">전신 / 기타</div>
                                </v-col>
                                <v-col cols="auto" v-for="(symptom, index) in symptomsOptions.general"
                                    :key="'general-' + index">
                                    <div class="inter-bold symptom-box"
                                        :class="symptoms.includes(symptom) ? 'selected-symptom' : 'symptom'"
                                        @click="addSymptom(symptom)">
                                        {{ symptom }}
                                    </div>
                                </v-col>
                            </v-row>
                        </v-container>
                    </v-card-text>
                    <v-card-actions>
                        <div class="mini-button" @click="symptomsModal = false" style="margin: auto;">증상 선택 완료</div>
                    </v-card-actions>
                </v-card>
            </v-dialog>
            <v-dialog v-model="reservedModal" max-width="500px">
                <v-card>
                    <v-card-title class="submodal mt-6 inter-bold text-center">
                        예약접수 완료하시겠습니까?
                    </v-card-title>
                    <v-card-text>
                        <v-container class="submodalsubject inter-bold text-center" style="margin-top: -30px;">
                            {{ formatDate(date) }} {{ selectedTime }}
                        </v-container>
                        <v-container class="submodaltext">
                            <v-row justify="center">
                                <v-col class="inter-bold subtitle-3">
                                    병원 정보
                                </v-col>
                            </v-row>
                            <v-row style="margin-top: -10px">
                                <v-col style="margin-top: -10px" class="inter-bord">
                                    {{ this.hospitalName }} <br>
                                    {{ doctor.name }} 원장
                                </v-col>
                            </v-row>
                            <v-row justify="center">
                                <v-col class="inter-bold subtitle-3">
                                    환자 정보
                                </v-col>
                            </v-row>
                            <v-row style="margin-top: -10px">
                                <v-col style="margin-top: -10px" class="inter-bord">
                                    {{ child.name }} <br>
                                    {{ child.ssn }}
                                </v-col>
                            </v-row>
                            <v-row justify="center">
                                <v-col class="inter-bold subtitle-3">
                                    진료 항목
                                </v-col>
                            </v-row>
                            <v-row style="margin-top: -10px">
                                <v-col style="margin-top: -10px" class="inter-bord">
                                    {{ mediItem }}
                                </v-col>
                            </v-row>
                            <v-row justify="center">
                                <v-col class="inter-bold subtitle-3">
                                    진료 정보
                                </v-col>
                            </v-row>
                            <v-row style="margin-top: -10px">
                                <v-col style="margin-top: -10px">
                                    {{ symptoms.join("•") }}
                                </v-col>
                            </v-row>
                            <v-row justify="center">
                                <v-col class="inter-bold subtitle-3">
                                    원장님께 하고 싶은 말
                                </v-col>
                            </v-row>
                            <v-row style="margin-top: -10px">
                                <v-col style="margin-top: -10px">
                                    {{ comment }}
                                </v-col>
                            </v-row>
                        </v-container>
                        <v-container style="text-align: center;">
                            <v-row justify="center" align="center">
                                <v-col cols="4" class="modal-selected" @click="reservedModal = false">
                                    취소
                                </v-col>
                                <v-col cols="4" class="modal-reserved" @click="reservedApply">
                                    예약접수 신청
                                </v-col>
                            </v-row>
                        </v-container>
                    </v-card-text>
                </v-card>
            </v-dialog>
            <v-dialog v-model="successReserveModal" max-width="600px">
                <v-card>
                    <v-card-title class="submodal mt-6 inter-bold text-center">
                        예약접수가 성공적으로 완료되었습니다!
                    </v-card-title>
                    <v-container style="text-align: center;" class="mt-3">
                        <v-row justify="center" align="center">
                            <v-col cols="4" class="modal-success-home" @click="goRoute('home')">
                                홈으로 가기
                            </v-col>
                            <v-col cols="4" class="modal-success-detail" @click="goRoute('detail')">
                                예약상세내역 확인
                            </v-col>
                        </v-row>
                        <br><br>
                    </v-container>
                </v-card>
            </v-dialog>
        </v-container>
    </v-container>
</template>

<script>
import axios from 'axios';
import { useRoute } from 'vue-router';

export default {
    data() {
        return {
            medicalType: "Scheduled",
            hostpitalName: "",
            hospitalId: '',
            child: null,
            doctor: null,
            childOptions: [],
            doctorList: [],
            symptoms: [],
            symptomsOptions: {
                respiratory: ['콧물', '기침', '인후통', '호흡곤란', '가래'],
                digestive: ['복통', '구토', '설사', '식욕부진', '변비'],
                skinAllergy: ['발진', '충혈/눈곱', '피부창백', '두드러기', '잇몸 부음'],
                general: ['무기력', '두통', '오한', '기절', '수면문제']
            },
            mediItem: null,
            symptomsModal: false,
            comment: null,
            date: null,
            reservationType: null,
            doctorTimeSlots: [],
            reservedTimes: [], // 이미 예약된 시간
            selectedTime: null, // 선택된 시간
            reservedModal: false,
            successReserveModal: false,
            reason: null,
            error: '',
        }
    },
    methods: {
        addChild(child) {
            this.child = child;
            console.log(child);
        },
        addDoctor(doctor) {
            this.doctor = doctor;
            this.fetchDoctorTime();
            console.log(doctor);
        },
        addmediItem(mediItem) {
            this.mediItem = mediItem;
            console.log(mediItem);
        },
        addSymptom(symptom) {
            const index = this.symptoms.indexOf(symptom);
            if (index === -1) {
                // 증상이 배열에 없으면 추가
                this.symptoms.push(symptom);
            } else {
                // 이미 있으면 배열에서 제거
                this.symptoms.splice(index, 1);
            }
        },
        updateDate(date) {
            this.date = date;
            console.log(date);
        },
        allowedDates(val) {
            const selectedDate = new Date(val);
            const currentDate = new Date();

            // 이번 주 일요일 계산
            const weekStart = new Date(currentDate);
            weekStart.setDate(currentDate.getDate() - currentDate.getDay()); // 이번 주 일요일

            // 이번 주 일요일을 포함한 다음 주 일요일까지
            const weekEnd = new Date(weekStart);
            weekEnd.setDate(weekStart.getDate() + 7); // 다음 주 일요일까지

            // 현재 선택된 의사 객체에서 운영시간을 가져옴
            const operatingHours = this.doctor.operatingHours || [];
            const selectedDay = selectedDate.getDay();

            // 운영 시간에 해당하는 요일을 확인.
            const isValidDay = operatingHours.some(hour => {
                const dayOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'].indexOf(hour.dayOfWeek);
                return dayOfWeek === selectedDay && !hour.untact;
            });

            // 예약 범위 & 운영 요일
            return selectedDate >= weekStart && selectedDate <= weekEnd && isValidDay; // 범위 체크
        },
        getTimeClass(time) {
            const currentTime = new Date();
            const timeInMinutes = this.timeToMinutes(time);
            const currentTimeInMinutest = currentTime.getHours() * 60 + currentTime.getMinutes();

            const selectedDate = new Date(this.date);
            const currendDate = new Date(currentTime.getFullYear(), currentTime.getMonth(), currentTime.getDate());

            const isvalidate = selectedDate.toString() == currendDate.toString();

            // 시간 상태에 따라 클래스 반환
            if (this.reservedTimes.includes(time)) {
                return "reserved"; // 예약된 시간
            } else if (timeInMinutes < currentTimeInMinutest && isvalidate) {
                return "reserved";
            } else if (this.selectedTime === time) {
                return "selected"; // 선택된 시간
            } else {
                return "available"; // 예약 가능한 시간
            }
        },
        selectTime(time) {
            // 예약된 시간은 선택 불가
            if (!this.reservedTimes.includes(time)) {
                this.selectedTime = time; // 선택된 시간 업데이트
            }
            console.log(this.selectedTime)
        },
        async reservedApply() {
            try {
                const formattedDate = this.date.toLocaleDateString('en-CA')

                const req = {
                    childId: this.child.id,
                    hospitalId: this.hospitalId,
                    doctorName: this.doctor.name,
                    doctorEmail: this.doctor.doctorEmail,
                    reservationType: this.medicalType,
                    reservationDate: formattedDate,
                    reservationTime: this.selectedTime,
                    untact: "false",
                    medicalItem: this.mediItem,
                    status: "Confirmed",
                    field: this.symptoms.toString(),
                    message: this.comment
                }
                console.log(req)

                const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/reservation-service/reservation/scheduled`,
                    req);

                console.log(response)
                this.reservedModal = false;
                this.successReserveModal = true;
            } catch (e) {
                alert(e.message)
            }
        },
        async fetchDoctorList() {
            try {
                console.log(this.hospitalId);
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/member-service/member/doctorList/${this.hospitalId}`);
                this.doctorList = response.data.result.content;
                console.log(response)
                this.doctorList.forEach(doctor => {
                    doctor.profileImgUrl = doctor.profileImgUrl ? doctor.profileImgUrl :
                        "https://todak-file.s3.ap-northeast-2.amazonaws.com/default-images/doctor-3d-image.png";
                })

                console.log(this.doctorList);
            } catch (e) {
                console.log(e);
            }
        },
        async fetchChildList() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/member-service/child/`);
                this.childOptions = response.data.result;
                console.log(this.childOptions)
            } catch (e) {
                console.log(e);
            }
        },
        async fetchDoctorTime() {
            try {
                this.reservedTimes = [];
                const request = {
                    doctorEmail: this.doctor.doctorEmail,
                    date: this.date.toLocaleDateString('en-CA')
                };
                const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/reservation-service/reservation/get/time`,
                    request);
                this.reservedTimes = response.data.map(time => time.slice(0, 5));
            } catch (e) {
                console.log(e)
            }
        },
        formatDate(date) {
            const option = {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                weekday: 'short'
            }

            const formattedDate = new Date(date).toLocaleDateString('ko-KR', option);
            return formattedDate
        },
        goRoute(route) {
            if (route == 'home') {
                this.$router.push('/')
            }
            else {
                // 여기에 "예약상세내역 페이지 경로"
                this.$router.push('/member/mypage/reservation')
            }
        },
        async operatingTime(openTime, closeTime, dayOfWeek) {
            console.log(openTime, closeTime, dayOfWeek);
            this.doctorTimeSlots = [];

            const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/reservation-service/hospital-operating-hours/getBreakTime/${this.hospitalId}`);
            const todayOperating = response.data;
            console.log(todayOperating);
            console.log(response);
            let breakStart = '';
            let breakEnd = '';
            todayOperating.find(item => {
                if (item.dayOfWeek == dayOfWeek) {
                    breakStart = item.breakStart
                    breakEnd = item.breakEnd
                }
            })

            console.log(breakStart, breakEnd);
            const start = this.timeToMinutes(openTime);
            const end = this.timeToMinutes(closeTime);
            if (breakStart || breakEnd) {
                breakStart = this.timeToMinutes(breakStart);
                breakEnd = this.timeToMinutes(breakEnd);
            }
            for (let i = start; i < end; i += 30) {

                if ((breakStart|| breakEnd) && i < breakStart || i >= breakEnd) {
                    this.doctorTimeSlots.push(this.minutesToTime(i))
                }else if(!(breakStart || breakEnd)){
                    this.doctorTimeSlots.push(this.minutesToTime(i))
                }
            }

            console.log("나 의사 시간", this.doctorTimeSlots);
        },
        timeToMinutes(time) {
            const [hours, minutes] = time.split(':').map(Number);
            return hours * 60 + minutes;
        },
        minutesToTime(time) {
            const hours = Math.floor(time / 60);
            const mins = time % 60;
            return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}`;
        },
        reserved() {
            try {
                if (!this.child) {
                    throw new Error("자녀를 선택해주세요.")
                }
                else if (!this.doctor) {
                    throw new Error("의사를 선택해주세요.")
                }
                else if (!this.date) {
                    throw new Error("진료 날짜를 선택해주세요.")
                }
                else if (!this.selectedTime) {
                    throw new Error("진료 시간을 선택해주세요.")
                }
                else if (!this.mediItem) {
                    throw new Error("진료항목을 선택해주세요.")
                }
                this.reservedModal = true;
            } catch (e) {
                alert(e.message)
            }
        }
    },
    async created() {
        const route = useRoute();
        this.hospitalId = route.params.hospitalId;
        this.hospitalName = this.$route.query.hospitalName;
        this.fetchDoctorList();
        this.fetchChildList();
    },
    watch: {
        date(newDate) {
            try {
                this.error = '';
                const date = new Date();
                const today = new Date(date.getFullYear(), date.getMonth(), date.getDate());

                if (newDate < today) {
                    this.date = null;
                    throw new Error("이전날은 선택할 수 없습니다.")
                }

                const options = { weekday: 'long' }
                const dayOfWeek = newDate.toLocaleDateString('en-US', options);

                const selectDay = this.doctor.operatingHours.filter(item => item.dayOfWeek === dayOfWeek);
                console.log(selectDay);
                this.operatingTime(selectDay[0].openTime, selectDay[0].closeTime, dayOfWeek);
                this.fetchDoctorTime();
            } catch (e) {
                console.log(e.message)
                this.error = e.message;
            }
        }
    },
    computed: {
    }
}
</script>

<style scoped>
* {
    font-weight: bold;
    user-select: none;
}

.title {
    margin-top: 50px;
    font-weight: bold;
    font-size: 1.7rem;
}

.big-font {
    text-align: left;
    color: #00499E;
    font-weight: bold;
    font-size: 21px;
    letter-spacing: -1px;
}

.small-font {
    text-align: left;
    margin-top: -20px;
    font-size: 13px;
    margin-left: 25px;
    color: #898787;
}

.custom-text {
    background-color: #ECF2FD;
    color: #00499E;
    font-size: 1.2rem;
    border-radius: 10px;
    padding: 4px 20px;
    display: inline-block;
    font-weight: bold;
}

.custom-text2 {
    background-color: #C2D7FF;
    color: #00499E;
    border-radius: 30px;
    font-size: 17px;
    padding: 2px 20px;
    display: inline-block;
    font-weight: bold;
    margin-left: 15px;
}

.custom-text3 {
    background-color: #0066FF;
    color: #FFFFFF;
    border-radius: 30px;
    font-size: 15px;
    padding: 2px 15px;
}

.mini-button {
    background-color: #0066FF;
    color: white;
    border-radius: 10px;
    padding: 10px 10px;
    margin-right: 10px;
    font-size: 14px;
    text-align: center;
    cursor: pointer;
}

.no-wrap {
    display: inline-block;
}

.header-row {
    width: 700px;
    margin: 0 auto;
}

.child {
    border: 1px solid #ccc;
    border-radius: 10px;
    width: 320px;
    padding: 12px;
}

.before-selected {
    margin-top: -20px;
    background-color: #E7E7E7;
    padding: 6px 20px;
    border-radius: 15px;
    font-size: 15px;
    letter-spacing: -1px;
}

.child-name {
    font-size: 15px;
}

.child-ssn {
    font-size: 12px;
    color: #898787;
}

.selected-child {
    background-color: #c2d7ff93;
    color: black;
}

.selected-doctor {
    background-color: #C2D7FF !important;
    color: black;
    height: 100px;
}

.selected-medi {
    background-color: #C2D7FF;
    color: #00499E;
}

.doctor {
    border: 1px solid #ccc;
    background-color: #F3F3F3;
    border-radius: 15px;
    width: 670px;
    height: 100px;
    padding: 20px 10px;
    margin-bottom: 20px;
    display: flex;
    align-items: center;
}

.more {
    margin-right: 20px;
}

.inline {
    display: inline-block;
}

.subtitle {
    font-size: 21px;
}

.dark-blue {
    color: #00499E;
}

.selected-symptom {
    background-color: #C2D7FF;
    border-radius: 40px;
    padding: 10px;
    font-size: 17px;
    color: #00499E;
    text-align: center;
}

.symptom {
    background-color: #cccccc;
    border-radius: 40px;
    padding: 10px;
    font-size: 17px;
    color: #676767;
    text-align: center;
}

.button {
    cursor: pointer;
    background-color: #C2D7FF;
    border-radius: 10px;
    padding: 10px 20px;
    margin-right: 10px;
    font-size: 17px;
    color: #00499E;
}

.text {
    border: 1px solid #ccc;
    border-radius: 10px;
    padding: 10px 15px;
    height: 180px;
    width: 700px;
    margin-top: 10px;
}

.symptom-box {
    cursor: pointer;
    margin-bottom: 10px;
    width: 100px;
    text-align: center;
}

.subtitle-2 {
    font-size: 1.2em;
    margin-bottom: -15px;
    margin-top: -20px;
}

.time-slot {
    cursor: pointer;
    margin-bottom: 5px;
    width: 100px;
    text-align: center;
    border-radius: 10px;
    font-weight: bold;
    margin-left: 30px;
    font-size: 19px;
    padding: 10px 5px;
}

.available {
    background-color: #ECF2FD;
    color: #00499E;
    font-weight: bold;
}

.selected {
    background-color: #0066FF;
    color: #fff;
    font-weight: bold;
}

.reserved {
    background-color: #e0e0e0;
    color: #757575;
    cursor: not-allowed;
}

.button-cursor {
    cursor: pointer;
}

.submodal {
    font-size: 25px;
    font-weight: bold;
}

.submodalsubject {
    font-size: 20px;
    font-weight: bold;
    color: #0075FF;
}

.submodaltext {
    background-color: #E5EEFF;
    width: 400px;
    margin: 0 auto;
}

.subtitle-3 {
    font-size: 1.2em;
    color: #0058FF;
}

.modal-selected {
    margin-top: 5px;
    background-color: #CACACA;
    color: #717171;
    border-radius: 10px;
    padding: 5px;
    margin-right: 5px;
    cursor: pointer;
}

.modal-reserved {
    margin-top: 5px;
    background-color: #0058FF;
    border-radius: 10px;
    padding: 5px;
    margin-left: 5px;
    color: #FFFFFF;
    cursor: pointer;
}

.modal-success-home {
    margin-top: 5px;
    background-color: #E0E0E0;
    color: #5F5F5F;
    border-radius: 5px;
    padding: 5px;
    margin-right: 5px;
    cursor: pointer;
    font-weight: bold;
}

.modal-success-detail {
    margin-top: 5px;
    background-color: #C2D7FF;
    border-radius: 5px;
    padding: 5px;
    margin-left: 5px;
    color: #00499E;
    cursor: pointer;
    font-weight: bold;
}

.hospital {
    font-weight: bold !important;
}

.schedule-chip {
    font-weight: bold;
    font-size: 25px;
    color: #00499E;
    background-color: #ECF2FD;
    border-radius: 10px;
}
</style>