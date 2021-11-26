<template>
  <div>
    <Header title="Creating Grade" />
    <main style="margin: 46px 0px;">
        <div class="inputs">
            <input @keyup="input_grade_course_update" ref="course_name" :nav-selectable="true" :data-create_grade_input="true" type="text" placeholder="Course" v-model="input_grade_course" />
            <input @keyup="input_grade_mark_update" ref="course_grade" :nav-selectable="true" :data-create_grade_input="true" type="number" placeholder="50" v-model="input_grade_mark" min=0 max=100 />
        </div>
    </main>
    <SoftKeys :softkeys.sync="softkeys" />
  </div>
</template>

<script>
import Header from './Header.vue';
import SoftKeys from './SoftKeys.vue';

export default {
    name: "CreatingPage",
    components: {
        Header,
        SoftKeys
    },
    props: {
        "softkeys": {
            type: Object,
            required: true
        }
    },
    data: function() {
        return {
            input_grade_course: undefined,
            input_grade_mark: undefined
        };
    },
    methods: {
        input_grade_course_update: function () {
            this.$emit('input_grade_course_update', this.input_grade_course);
        },
        input_grade_mark_update: function () {
            this.$emit('input_grade_mark_update', this.input_grade_mark);
        }
    }
}
</script>
