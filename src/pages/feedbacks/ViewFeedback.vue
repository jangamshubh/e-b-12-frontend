<template>
    <div class="grid">
        <div class="col-12">
            <Card>
				<template v-slot:title>
					<div class="flex align-items-center justify-content-between mb-0">
						<h5>View Feedback Details</h5>
                        <Button icon="pi pi-arrow-left" class="p-button-rounded mr-2 mb-2" @click="redirectToIndex()"/>
                    </div>
                </template>
                <template v-slot:content>
                    <table class="min-w-full border divide-y divide-gray-200">
                        <tbody class="bg-white divide-y divide-gray-200 divide-solid">
                            <tr class="bg-white">
                                <th>No. of Feedback Received / Total Students</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ data.feedback_filled_by }} / {{ data.total_students }}
                                </td>
                            </tr>
                            <tr class="bg-white">
                                <th>Question 1:</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ question.first_question.title }}
                                </td>
                                <th>Average Rating</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ rating.first_rating }}
                                </td>
                            </tr>
                            <tr class="bg-white">
                                <th>Question 2:</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ question.second_question.title }}
                                </td>
                                <th>Average Rating</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ rating.second_rating }}
                                </td>
                            </tr>
                            <tr class="bg-white">
                                <th>Question 3:</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ question.third_question.title }}
                                </td>
                                <th>Average Rating</th>
                                <td class="px-6 py-4 text-sm leading-5 text-gray-900 whitespace-no-wrap">
                                    {{ rating.third_rating }}
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </template>
            </Card>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import authHeader from '../../services/auth-header';
export default {
    data() {
        return {
            question: {
                first_question: {},
                second_question: {},
                third_question: {},
            },
            rating: {
                first_rating: '',
                second_rating: '',
                third_rating: '',
            },
            data: {
                feedback_filled_by: '',
                total_students: '',
            },
            subject:{
                name:'',
                code: '',
            },
        }
    },
    computed: {
        loggedIn() {
            return this.$store.state.auth.status.loggedIn;
        },
        checkRole() {
            return this.$store.state.auth.role;
        }
    },
    created() {
        this.checkUserLogin();
        this.checkUserRole();
        this.getData();
    },
    methods: {
        checkUserLogin() {
            if (!this.loggedIn) {
                this.$router.push("/login");
            }
        },
        checkUserRole() {
            if(this.checkRole != 'Super Admin' && this.checkRole != 'Teacher') {
               this.$router.push("/");
            }
        },
        getData() {
            axios.get(`${process.env.VUE_APP_API_URL}/feedback/${this.$route.params.attendance_id}/view-feedback`,{ headers: authHeader() }).then(data => {
                let response = data.data;
                if(response.status == 'success') {
                    this.question.first_question = response.first_question;
                    this.question.second_question = response.second_question;
                    this.question.third_question = response.third_question;
                    this.rating.first_rating = response.first_question_average;
                    this.rating.second_rating = response.second_question_average;
                    this.rating.third_rating = response.third_question_average;
                    this.data.total_students = response.total_students;
                    this.data.feedback_filled_by = response.feedback_filled_by;
                }
            })
        },
        redirectToIndex() {
            this.$router.push({ name: 'attendances.index' });
        },
    },
}
</script>
