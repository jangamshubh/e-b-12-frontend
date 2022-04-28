<template>
    <div class="row">
        <div class="col-12">
            <div class="card">
                <Toast />
                <div class="card-header flex justify-content-between flex-column sm:flex-row">
                    <h5>Add Feedback for lecture</h5>
                    <Button type="button" icon="pi pi-arrow-left" label="Back To Index Page" class="p-button-outlined mb-2" @click="redirectToIndexPage()" />
                </div>
                <div class="card">
                    <div class="card-img">
                        <img class="image-fit" v-bind:src="user.profile_pic" />
                    </div>
                    <b>{{ user.name }}</b>
                </div>
                <div class="grid formgrid" v-for="(question, index) in questions" :key="question.id">
                    <InputText id="name1" type="hidden" v-model="data.student_feedback[index].student_id" :value='question.id' :disabled="true"/>
                    <div class="mr-2 lg:col-12">
                        {{ question.title }}
                    </div>
                    <div class="lg:col-12">
                        <SelectButton v-model="data.student_feedback[index].feedback" :options="selectButtonValues1" optionLabel="name" optionValue="value"/>
                    </div>
                </div>
                <Button label="Submit" v-on:click="saveFeedback()"></Button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import authHeader from '../../services/auth-header';
export default {
    name:"Login",
    components: {
    },
    data() {
        return {
            data: {
                student_feedback: [],
            },
            user: {},
            reaction: 'natural',
            isActive: false,
            isDisabled: false,
            feedback: '',
            questions: [],
            selectButtonValues1: [
                { name: '😡 Very Unsatisfied', value: 1 },
                { name: '😠 Unsatisfied', value: 2 },
                { name: '😐 Neutral', value: 3 },
                { name: '😀 Satisfied', value: 4 },
                { name: '😊 Very Satisfied', value: 5 },
            ],

        };
    },
    computed: {
        loggedIn() {
            return this.$store.state.auth.status.loggedIn;
        },
        checkUser() {
            return this.$store.state.auth.user;
        },
        checkRole() {
            return this.$store.state.auth.role;
        }
    },
    created() {
        this.checkUserLogin();
        this.checkUserRole();
        this.getQuestions();
        this.checkFeedbackPermission();
        this.getTeacherProfilePic();
    },
    methods: {
        checkUserLogin() {
            if (!this.loggedIn) {
                this.$router.push("/login");
            }
        },
        checkUserRole() {
            if(this.checkRole != 'Super Admin' && this.checkRole != 'Student') {
                this.$router.push("/");
            }
        },
        toastMessage(message,status) {
            this.$toast.add({ severity: status, summary: message, detail:'', life: 3000 });
        },
        redirectAfterSuccess() {
            setTimeout(() => this.$router.push({ name: 'attendances.index' }), 3000);
        },
        getQuestions() {
            axios.get(`${process.env.VUE_APP_API_URL}/feedback/get-questions`, { headers: authHeader()}).then(data => {
                let response = data.data;
                if (response.status == 'success') {
                    this.questions = response.data;
                    for(let i = 0; i < this.questions.length; i++) {
                        let feedback_array = { question_id: '', feedback: '' };
                        this.data.student_feedback.push(feedback_array);
                    }
                    for(let i = 0; i < this.questions.length; i++) {
                        this.data.student_feedback[i]['question_id'] = this.questions[i]['id'];
                        this.data.student_feedback[i]['feedback'] = '';
                    }
                } else {
                    this.toastMessage(response.message,response.status);
                    this.$router.push({ name: 'attendances.index' });
                }
            })
        },
        checkFeedbackPermission() {
            axios.get(`${process.env.VUE_APP_API_URL}/feedback/${this.$route.params.attendance_id}/add-feedback-permission`, { headers: authHeader() }).then(data => {
                let response = data.data;
                if(response.status != 'success') {
                    this.toastMessage(response.message,response.status);
                    this.redirectToIndexPage();
                }
            })
        },
        getTeacherProfilePic() {
            axios.get(`${process.env.VUE_APP_API_URL}/feedback/${this.$route.params.attendance_id}/get-teacher-profile-pic`, { headers: authHeader() }).then(data => {
                let response = data.data;
                // if(response.status == 'success') {
                    this.user = response.data;
                    console.log(this.user.profile_pic);
                // }
            })
        },
        saveFeedback() {
            axios.post(`${process.env.VUE_APP_API_URL}/feedback/${this.$route.params.attendance_id}/add-feedback`,this.data,{ headers: authHeader() }).then(data => {
                let response = data.data;
                if(response.status == 'success') {
                    this.toastMessage(response.message,response.status);
                    this.redirectAfterSuccess();
                } else {
                    console.log(response);
                }
            })
        },
        redirectToIndexPage() {
            this.$router.push({ name: 'attendances.index' });
        },

    },
}
</script>
