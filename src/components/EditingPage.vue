<template>
  <div>
    <Header title="Editing Grade" />
    <main style="margin: 46px 0px;">
        <div class="inputs">
            <input @keyup="input_grade_course_update" ref="course_name" :nav-selectable="true" :data-edit_grade_input="true" type="text" placeholder="Course" v-model="input_grade_course" />
            <input @keyup="input_grade_mark_update" ref="course_grade" :nav-selectable="true" :data-edit_grade_input="true" type="number" placeholder="50" v-model="input_grade_mark" min=0 max=100 />
        </div>
    </main>
    <SoftKeys :softkeys.sync="softkeys" />
  </div>
</template>

<script>
import Header from './Header.vue';
import SoftKeys from './SoftKeys.vue';

export default {
    name: "EditingPage",
    components: {
        Header,
        SoftKeys
    },
    props: {
        "softkeys": {
            type: Object,
            required: true
        },
        "grades": {
            type: Array,
            required: true
        },
        "editing_index": {
            type: Number,
            required: true
        },
    },
    data: function() {
        return {
            input_grade_course: undefined,
            input_grade_mark: undefined
        };
    },
    mounted: function() {
        // Update the Editing Inputs to have the Selected Grade Values
        this.input_grade_course = this.grades[ this.editing_index ].course;
        this.input_grade_mark = this.grades[ this.editing_index ].mark;
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
