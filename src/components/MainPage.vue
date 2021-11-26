<template>
<div>
    <Header title="Grade Keeper" />
    <main style="margin: 46px 0px 56px 0px;">
        <div class="buttons">
            <button id="create-grade-btn" ref="create" :nav-selectable="true" > Create Grade </button>
        </div>
        <div class="inputs">
            <input id="search-grades" type="text" placeholder="Search Courses" v-model="search_term" :nav-selectable="true" />
        </div>
        <div class="filter-buttons"> 
            <div>
                <button id="sort-grade-course-btn" ref="sort_course" :nav-selectable="true" > 
                    <i v-if="sort_grade_course" width="16" height="16" fill="currentColor" class="bi bi-sort-alpha-down" viewBox="0 0 16 16"> </i>
                    <i v-else width="16" height="16" fill="currentColor" class="bi bi-sort-alpha-up" viewBox="0 0 16 16"> </i>
                </button>

                <button id="sort-grade-mark-btn" ref="sort_mark" :nav-selectable="true" > 
                    <i v-if="sort_grade_mark" width="16" height="16" fill="currentColor" class="bi bi-sort-numeric-up" viewBox="0 0 16 16"> </i>
                    <i v-else width="16" height="16" fill="currentColor" class="bi bi-sort-numeric-down" viewBox="0 0 16 16"> </i>
                </button>
            </div>
            <div> 
                <button id="no-filter-btn" :nav-selectable="true" :class="{ 'selected-filter' : filter_applied === 0 }" > 
                    <i width="16" height="16" fill="currentColor" class="bi bi-slash-circle" viewBox="0 0 16 16"> </i>
                </button>
                <button id="honors-filter-btn" :nav-selectable="true" :class="{ 'selected-filter' : filter_applied === 1 }"> 
                    <i width="16" height="16" fill="currentColor" class="bi bi-mortarboard-fill" viewBox="0 0 16 16"> </i>
                </button>

                <button id="fails-filter-btn" :nav-selectable="true" :class="{ 'selected-filter' : filter_applied === 2 }"> 
                    <i width="16" height="16" fill="currentColor" class="bi bi-hand-thumbs-down-fill" viewBox="0 0 16 16"> </i>
                </button>
            </div>
        </div>
        <div>
            <div class="task" v-for="(grade) in filtered_grades" :key="grade.id" >
                <span :data-grade_id="grade.id" :nav-selectable="true"> <span>{{ grade.course }}</span> <span>{{ grade.mark }}%</span> </span>
            </div>
        </div>
    </main>
    <footer class="footer-stats"> 
        <p style="display: flex; justify-content: space-between;">
            <small> 
                Max: {{ filtered_grades.length === 0 ? 0 : Math.max( ...filtered_grades.map( (grade) => grade.mark ) ) }}
            </small> 
             <small> 
                Mean: {{ filtered_grades.length === 0 ? 0 : (filtered_grades.reduce( (p, grade) => p + grade.mark, 0 ) / filtered_grades.length).toPrecision(4) }}
            </small> 
            <small> 
                Min: {{ filtered_grades.length === 0 ? 0 : Math.min( ...filtered_grades.map( (grade) => grade.mark ) ) }} 
            </small> 
        </p>
    </footer>
    <SoftKeys :softkeys.sync="softkeys" />
  </div>
</template>

<script>
import Header from './Header.vue';
import SoftKeys from './SoftKeys.vue';

export default {
    name: "MainPage",
    components: {
        Header,
        SoftKeys
    },
    props: {
        "grades": {
            type: Array,
            required: true
        },
        "softkeys": {
            type: Object,
            required: true
        },
        "filter_applied": {
            type: Number,
            required: true
        },
        "sort_grade_mark": {
            required: true
        },
        "sort_grade_course": {
            required: true
        },
    },
    data: function() {
        return {
            search_term: "",
        };
    },
    computed: {
      filtered_grades: function() {

            let the_grades = [...this.grades];


            // Filter by Search Input
            const regex = new RegExp( this.search_term, 'gi' );
            the_grades = the_grades.filter( grade => regex.exec( grade.course ) );

            // Filter by Honors || Fails 
            if ( this.filter_applied === 1 ) {
                the_grades = the_grades.filter( grade => grade.mark >= 80 );

            } else if ( this.filter_applied === 2) {
                the_grades = the_grades.filter( grade => grade.mark < 50 );
            }


            // Sorting By Course
            if ( this.sort_grade_course !== undefined ) {
                the_grades.sort( ( a, b ) => {
                      if ( a.course < b.course ){
                        return this.sort_grade_course ? -1 : 1;
                      }
                      if ( a.course > b.course ){
                        return this.sort_grade_course ? 1 : -1;
                      }
                      return 0;
                } );
            }

            // Sorting By Mark
            if ( this.sort_grade_mark !== undefined ) {
                the_grades.sort( ( a, b ) => {
                      if ( a.mark < b.mark ){
                        return this.sort_grade_mark ? -1 : 1;
                      }
                      if ( a.mark > b.mark ){
                        return this.sort_grade_mark ? 1 : -1;
                      }
                      return 0;
                } );
            }

            return the_grades;

      }
  },
    methods: {
        onEnter: function() {
            this.emit("onEnter");
        }
    }
}
</script>
