<template>
  <div class="signup-container">
    <button class="back-btn" @click="goBack">
      ← <span class="back-text">뒤로</span>
    </button>

    <h2 class="title">회원가입</h2>

    <form @submit.prevent="handleSignup" class="form-layout">
      <div class="form-group">
        <label>이름</label>
        <input
          v-model="name"
          type="text"
          required
          class="input-box"
          @input="sanitizeName"
        />
      </div>

      <div class="form-group">
        <label>전화번호</label>
        <div class="phone-input-group">
          <input
            v-model="phone"
            type="text"
            maxlength="13"
            class="input-box"
            placeholder="010-1234-5678"
            @input="formatPhone"
          />
        </div>
        <p v-if="phoneMessage" :class="phoneMessageClass">{{ phoneMessage }}</p>
      </div>

      <div class="form-group">
        <label>이메일</label>
        <div class="email-check-group">
          <input
            v-model="email"
            type="email"
            required
            class="input-box"
            placeholder="example@domain.com"
            @input="sanitizeEmail"
          />
        </div>
        <p v-if="emailMessage" :class="emailMessageClass">{{ emailMessage }}</p>
      </div>

      <div class="form-group">
        <label>비밀번호</label>
        <div class="password-input-group">
          <input
            :type="passwordVisible ? 'text' : 'password'"
            v-model="password"
            required
            minlength="10"
            class="input-box"
            placeholder="영문+특수문자+숫자 10자 이상"
            @input="validatePassword"
          />
          <button
            type="button"
            class="eye-btn"
            @click="togglePasswordVisibility"
          >
            {{ passwordVisible ? '가리기' : '보기' }}
          </button>
        </div>
        <div class="password-rules">
          <button :class="rule.letter ? 'rule-pass' : 'rule-fail'">영문</button>
          <button :class="rule.special ? 'rule-pass' : 'rule-fail'">
            특수문자
          </button>
          <button :class="rule.number ? 'rule-pass' : 'rule-fail'">숫자</button>
        </div>
      </div>

      <div class="form-group">
        <label>비밀번호 확인</label>
        <input
          v-model="passwordConfirm"
          type="password"
          required
          class="input-box"
          placeholder="비밀번호 다시 입력"
          @input="validatePasswordConfirm"
        />
        <p v-if="passwordConfirmMessage" :class="passwordConfirmMessageClass">
          {{ passwordConfirmMessage }}
        </p>
      </div>

      <button type="submit" class="submit-btn" :disabled="!canSubmit">
        회원가입
      </button>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import Swal from 'sweetalert2';

const router = useRouter();

const name = ref('');
const email = ref('');
const phone = ref('');
const password = ref('');
const passwordConfirm = ref('');
const passwordVisible = ref(false);

const emailMessage = ref('');
const phoneMessage = ref('');
const passwordConfirmMessage = ref('');

const emailMessageClass = ref('');
const phoneMessageClass = ref('');
const passwordConfirmMessageClass = ref('');

const rule = ref({
  special: false,
  letter: false,
  number: false,
});

const canSubmit = computed(() => {
  return (
    name.value &&
    validateEmailFormat(email.value) &&
    validatePhoneFormat(phone.value) &&
    rule.value.special &&
    rule.value.letter &&
    rule.value.number &&
    password.value.length >= 10 &&
    password.value === passwordConfirm.value
  );
});

const goBack = () => {
  router.push('/login');
};

const handleSignup = () => {
  Swal.fire({
    title: '회원가입 완료!',
    text: '로그인 화면으로 이동합니다.',
    icon: 'success',
    confirmButtonText: '확인',
  }).then(() => {
    router.push('/login');
  });
};

const formatPhone = () => {
  phone.value = phone.value
    .replace(/[^\d]/g, '')
    .replace(/(\d{3})(\d{0,4})(\d{0,4})/, (_, p1, p2, p3) => {
      let result = p1;
      if (p2) result += '-' + p2;
      if (p3) result += '-' + p3;
      return result;
    });
  validatePhone();
};

const sanitizeEmail = () => {
  email.value = email.value.toLowerCase().replace(/[<>'"\\/%;&=?!]/g, '');
  validateEmail();
};

const sanitizeName = () => {
  name.value = name.value.replace(/[<>'"\\/%;&=?!]/g, '');
};

const validateEmail = () => {
  if (!validateEmailFormat(email.value)) {
    emailMessage.value = '올바른 이메일 형식을 입력하세요.';
    emailMessageClass.value = 'error-msg';
  } else {
    emailMessage.value = '사용 가능한 이메일입니다.';
    emailMessageClass.value = 'success-msg';
  }
};

const validatePhone = () => {
  if (!validatePhoneFormat(phone.value)) {
    phoneMessage.value = '올바른 전화번호 형식을 입력하세요. (010-1234-5678)';
    phoneMessageClass.value = 'error-msg';
  } else {
    phoneMessage.value = '사용 가능한 전화번호입니다.';
    phoneMessageClass.value = 'success-msg';
  }
};

const validateEmailFormat = (value) => {
  return /.+@.+\..+/.test(value);
};

const validatePhoneFormat = (value) => {
  return /^\d{2,3}-\d{3,4}-\d{4}$/.test(value);
};

const validatePassword = () => {
  rule.value.special = /[@$!%*#?&]/.test(password.value);
  rule.value.letter = /[A-Za-z]/.test(password.value);
  rule.value.number = /\d/.test(password.value);
};

const validatePasswordConfirm = () => {
  if (passwordConfirm.value === password.value) {
    passwordConfirmMessage.value = '비밀번호가 일치합니다.';
    passwordConfirmMessageClass.value = 'success-msg';
  } else {
    passwordConfirmMessage.value = '비밀번호가 일치하지 않습니다.';
    passwordConfirmMessageClass.value = 'error-msg';
  }
};

const togglePasswordVisibility = () => {
  passwordVisible.value = !passwordVisible.value;
};
</script>

<style scoped>
/* 스타일 동일, 생략 */
</style>

<style scoped>
.signup-container {
  max-width: 450px;
  margin: 50px auto;
  padding: 30px;
  border: 1px solid #ccc;
  border-radius: 12px;
  background: #fafafa;
  position: relative;
}

.back-btn {
  position: absolute;
  top: 15px;
  left: 20px;
  background: none;
  border: none;
  font-size: 1.2rem;
  color: #2d3748;
  cursor: pointer;
}

.back-text {
  font-size: 1rem;
}

.title {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 20px;
  color: #2d3748;
}

.form-layout {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.email-check-group,
.phone-input-group,
.password-input-group {
  display: flex;
  gap: 10px;
  align-items: center;
}

.input-box {
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  flex: 1;
  min-width: 0;
}

.eye-btn {
  background: none;
  border: none;
  font-size: 0.9rem;
  cursor: pointer;
}

.password-rules {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.rule-pass {
  background: #4fd1c5;
  color: white;
  border: none;
  padding: 5px 8px;
  border-radius: 5px;
}

.rule-fail {
  background: #ccc;
  color: white;
  border: none;
  padding: 5px 8px;
  border-radius: 5px;
}

.submit-btn {
  padding: 12px;
  font-size: 1.1rem;
  background-color: #4fd1c5;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.submit-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.error-msg {
  color: #e53e3e;
  font-size: 0.85rem;
  margin-top: 5px;
}

.success-msg {
  color: #4fd1c5;
  font-size: 0.85rem;
  margin-top: 5px;
}
</style>
