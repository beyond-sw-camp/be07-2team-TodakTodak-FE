<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="6">
        <router-link to="/" class="logo">
            <img src="@/assets/todak-new-logo-removebg.png" alt="TodakTodak Logo" class="logo-image" />
        </router-link>

        <h3 class="text-left mb-5 bold-text">가입 정보를 입력 후
          <br>
          가입 승인 요청을 해주세요.
        </h3>

        <v-form ref="form" v-model="formValid">

          <h5 class="text-left">기본 정보</h5>
          <h6 class="text-left">대표자(원장님)</h6>
          <v-text-field v-model="form.adminName"
            variant="underlined"
            label="대표자(원장님) 이름을 입력해주세요." :rules="[rules.required]"></v-text-field>
          <h6 class="text-left">전화번호</h6>
          <v-text-field 
          v-model="form.adminPhoneNumber"
          variant="underlined"
          type="text" 
          label="대표자(원장님) 전화번호를 입력해주세요.(ex.01012341234)" 
          :rules="[rules.required]"
          @input="form.adminPhoneNumber = form.adminPhoneNumber.replace(/[^0-9]/g, '')"
        >
        </v-text-field>        
        <v-text-field
        v-model="form.adminEmail"
        variant="underlined"
        label="대표자(원장님) 이메일을 입력해주세요." 
        :rules="[rules.required, rules.email]"
      >
        <template #append>
          <v-btn @click="sendVerificationEmail" class="address-btn" small variant="flat">
            {{ verificationSent ? '인증 재발송' : '인증 코드 발송' }}
          </v-btn>
        </template>
      </v-text-field>
    
      <!-- 인증 코드 입력 필드 -->
      <v-text-field
      v-model="verificationCode"
      label="인증번호"
      variant="underlined"
      prepend-icon="mdi-key"
      :rules="[rules.required]"
      v-if="verificationSent"
    >
      <!-- 인증 확인 버튼 -->
      <template #append>
        <v-btn @click="checkVerificationCode" class="address-btn" small variant="flat">
          인증번호 확인
        </v-btn>
      </template>
    </v-text-field>
      <div v-if="isVerified" class="d-flex align-center text-center mb-4">
        <v-icon class="mr-2" color="green" size="small">mdi-check-circle</v-icon>
        <h7 class="bold-text email-verified-text">이메일 인증이 완료되었습니다.</h7>
      </div>

          <h6 class="text-left">비밀번호</h6>
          <v-text-field v-model="form.adminPassword"
            variant="underlined"
            label="비밀번호를 입력해주세요.(비밀번호는 최소 8자 이상입니다.)" :type="'password'" :rules="[rules.required]"></v-text-field>
                      <!-- 비밀번호 확인 필드 -->
          <v-text-field 
          v-model="adminPasswordConfirm"
          variant="underlined"
          label="비밀번호를 다시 입력해주세요.(비밀번호는 최소 8자 이상입니다.)" 
          type="password"
          :rules="[rules.required, confirmPasswordRule]"
        ></v-text-field>


          <h5 class="text-left">병원 정보</h5>

          <h6 class="text-left">병원 이름(ex.연세소아과)</h6>
          <v-text-field v-model="form.hospitalName"
            variant="underlined"
            label="병원명을 입력해주세요.(ex.연세소아과)" :rules="[rules.required]"></v-text-field>

          <h6 class="text-left">주소</h6>
          <v-text-field
            v-model="combinedAddress"
            variant="underlined"
            label="주소를 검색하여 입력해주세요."
            readonly
            :rules="[rules.required]"
          >
            <template #append>
              <v-btn @click="openPostcode" class="address-btn" small variant="flat">주소 검색</v-btn>
            </template>
          </v-text-field>
          <!-- <v-text-field v-model="form.latitude" variant="underlined" label="위도" readonly></v-text-field>
          <v-text-field v-model="form.longitude" variant="underlined" label="경도" readonly></v-text-field> -->

          <h6 class="text-left">병원 번호</h6>
          <v-text-field 
          v-model="form.hospitalPhoneNumber" 
          variant="underlined"
          type="text" 
          label="병원 전화번호를 입력해주세요.(ex.0212341234)" 
          :rules="[rules.required]"
          @input="form.hospitalPhoneNumber = form.hospitalPhoneNumber.replace(/[^0-9]/g, '')"
        >
        </v-text-field>

          <h6 class="text-left">사업자 등록번호</h6>
          <v-text-field v-model="form.businessRegistrationInfo"
            variant="underlined"
            label="사업자 등록번호를 입력해주세요." :rules="[rules.required]"></v-text-field>

          <v-row justify="center" class="button-row">
            <v-btn class="res-btn" @click="submitForm" :disabled="!formValid">가입 요청</v-btn>
          </v-row>
        </v-form>

        <v-dialog v-model="dialog" max-width="500px">
          <v-card class="custom-modal">
            <v-card-title class="text-h5 text-center">회원가입이 완료되었습니다.</v-card-title>
            <v-card-text>
              <v-row justify="center" class="mt-4">
                <v-btn class="custom-modal-btn" @click="closeModal">
                  홈
                </v-btn>
                <v-divider vertical class="vertical-divider"></v-divider>
                <v-btn class="custom-modal-btn" @click="loginModal">
                  로그인
                </v-btn>
              </v-row>
            </v-card-text>
          </v-card>
        </v-dialog>

      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';
/* global kakao, daum */

export default {
  data() {
    return {
      formValid: false,
      form: {
        hospitalName: '',
        address: '',
        dong: '',
        latitude: '',
        longitude: '',
        hospitalPhoneNumber: '',
        businessRegistrationInfo: '',
        adminName: '',
        adminEmail: '',
        adminPassword: '',
        adminPhoneNumber: '',
      },
      adminPasswordConfirm: '',
      verificationCode: '', // 사용자가 입력할 인증 코드
      verificationSent: false, // 인증 코드 발송 상태
      isVerified: false, // 인증 완료 상태
      rules: {
        required: (value) => !!value || '필수 입력 항목입니다.',
        email: (value) => /.+@.+\..+/.test(value) || '유효한 이메일을 입력하세요.',
      },
      confirmPasswordRule: value => value === this.form.adminPassword || '비밀번호가 일치하지 않습니다.', // 비밀번호 확인 검증
      geocoder: null, // Geocoder 객체
      dialog: false, // 모달 창 상태를 관리
    };
  },
  computed: {
    combinedAddress() {
      // 주소와 동을 합쳐서 표시
      return `${this.form.address} ${this.form.dong}`;
    },
  },
  mounted() {
    this.loadKakaoMapScript();
  },
  methods: {
    // 카카오맵 스크립트를 로드하는 함수
    loadKakaoMapScript() {
      const kakaoScriptUrl = `//dapi.kakao.com/v2/maps/sdk.js?appkey=${process.env.VUE_APP_KAKAO_MAP_JS_APP_KEY}&libraries=services&autoload=false`;
      this.loadScript(kakaoScriptUrl, this.initializeKakaoMaps);
    },
    // 외부 스크립트 로드 함수
    loadScript(src, callback) {
      const existingScript = document.querySelector(`script[src="${src}"]`);
      if (!existingScript) {
        const script = document.createElement('script');
        script.src = src;
        script.onload = callback.bind(this); // 스크립트가 로드된 후 콜백 실행
        script.onerror = (e) => {
          console.error('스크립트 로드 오류: ', e);
        };
        document.head.appendChild(script);
      } else {
        callback();
      }
    },
    // 카카오 맵 초기화 함수
    initializeKakaoMaps() {
      if (typeof kakao !== 'undefined') {
        kakao.maps.load(() => {
          this.geocoder = new kakao.maps.services.Geocoder(); // Geocoder 객체 초기화
          this.loadDaumPostcodeScript(); // 우편번호 스크립트 로드
        });
      } else {
        console.error('카카오 맵 스크립트가 로드되지 않았습니다.');
      }
    },
    // 다음 우편번호 스크립트 로드 함수
    loadDaumPostcodeScript() {
      const daumScriptUrl = '//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js';
      this.loadScript(daumScriptUrl, () => {
        console.log('다음 우편번호 스크립트 로드 완료.');
      });
    },
    // 주소 검색 팝업 열기
    openPostcode() {
      new daum.Postcode({
        oncomplete: (data) => {
          this.form.address = data.roadAddress;
          this.form.dong = data.bname;
          if (this.geocoder) {
            this.geocoder.addressSearch(data.roadAddress, (results, status) => {
              if (status === kakao.maps.services.Status.OK) {
                this.form.latitude = results[0].y; // 위도 저장
                this.form.longitude = results[0].x; // 경도 저장
              } else {
                console.error('주소 검색 실패: ', status);
              }
            });
          }
        },
      }).open(); // 기본 팝업 창을 사용
    },

    // 폼 제출
    submitForm() {
      if (this.$refs.form.validate()) {
        axios
          .post(`${process.env.VUE_APP_API_BASE_URL}/reservation-service/hospital/hospital-admin/register`, this.form)
          .then((response) => {
            this.dialog = true; // 가입 완료 후 모달 열기
            console.log(response);
          })
          .catch((error) => {
            console.error('가입 오류:', error.response.data.status_message);
            alert(error.response.data.status_message);
          });
      } else {
        alert('입력된 정보를 확인하세요.');
      }
    },
    closeModal() {
      this.dialog = false;
      this.$router.push('/'); // 확인 버튼 클릭 시 홈페이지로 리다이렉트
    },
    loginModal() {
      this.dialog = false;
      this.$router.push('/all/hospital/login'); // 확인 버튼 클릭 시 로그인 페이지로 리다이렉트
    },
    // 이메일 인증 코드 발송
  async sendVerificationEmail() {
    try {
      // 인증 코드 발송 로직
      const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/member/send-verification-code`, {
        memberEmail: this.form.adminEmail
      });
      if (response.status === 200) {
        this.verificationSent = true;
        alert('인증 코드가 이메일로 발송되었습니다.');
      }
    } catch (error) {
      alert(error.response?.data?.message || '이메일 인증 코드 발송에 실패했습니다.');
    }
  },
    // 인증 코드 확인
    async checkVerificationCode() {
      try {
        const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/member/verify-code`, {
          memberEmail: this.form.adminEmail,
          code: this.verificationCode
        });
        if (response.status === 200) {
          this.isVerified = true;
          alert('이메일 인증이 성공적으로 완료되었습니다.');
        }
      } catch (error) {
        alert(error.response?.data?.message || '인증 코드 확인에 실패했습니다.');
      }
    }

 },
};
</script>

<style scoped>
.hospital-container {
  max-width: 700px;
  margin: 0 auto;
}
.logo {
  font-weight: bold;
  color: inherit;
  text-decoration: none;
}

.logo-image {
  width: 250px; /* 원하는 고정 너비 */
  max-width: 150%; /* 부모 요소 너비를 넘지 않도록 설정 */
  height: auto; /* 높이는 비율에 맞춰 자동 조절 */
  object-fit: contain; /* 이미지가 고정된 크기 안에서 비율을 유지 */
  margin-bottom: 20px;
}
.bold-text {
  font-weight: bold;
}
.res-btn {
  background-color: #C2D7FF !important;
  color: #00499e;
  border-radius: 20px;
  width: 130px;
  height: 44px;
  margin-bottom: 40px;
  margin-top: 10px;
  font-family: 'Inter';
  font-style: normal;
  font-weight: 700;
  font-size: 15px;
  line-height: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.vertical-divider {
  width: 1px;
  height: 30px;
  background-color: #a7a7a7;
  margin: 0 10px;

}
.custom-modal {
  position: absolute;
  width: 500px;
  height: auto;
  background: #FFFFFF;
  border-radius: 20px;
}

.custom-modal-btn {
  background-color: #C2D7FF;
  color: #00499E;
  border-radius: 20px;
}
.address-btn{
  font-weight: bold; /* 폰트 굵게 설정 */
  color: #00499E;
  background-color: #ECF2FD;
  border-radius: 10px; /* 모서리 둥글기 */
}
.bold-text {
  font-weight: 800px; /* 글자 굵게 */
}

.email-verified-text {
  color: #1f5223; /* 원하는 색상으로 변경 (#4caf50는 초록색 예시) */
}
</style>
  