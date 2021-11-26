<template>
  <div v-if="page === 'main'">
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
  <div v-else-if="page === 'creating'">
    <Header title="Creating a Grade" />
    <main style="margin: 46px 0px;">
        <div class="inputs">
            <input ref="course_name" :nav-selectable="true" :data-create_grade_input="true" type="text" placeholder="Course" v-model="input_grade_course" />
            <input ref="course_grade" :nav-selectable="true" :data-create_grade_input="true" type="number" placeholder="50" v-model="input_grade_mark" min=0 max=100 />
        </div>
    </main>
    <SoftKeys :softkeys.sync="softkeys" />
  </div>
  <div v-else-if="page === 'editing'">
    <Header title="Editing a Grade" />
    <main style="margin: 46px 0px;">
        <div class="inputs">
            <input ref="course_name" :nav-selectable="true" :data-edit_grade_input="true" type="text" placeholder="Course" v-model="input_grade_course" />
            <input ref="course_grade" :nav-selectable="true" :data-edit_grade_input="true" type="number" placeholder="50" v-model="input_grade_mark" min=0 max=100 />
        </div>
    </main>
    <SoftKeys :softkeys.sync="softkeys" />
  </div>
</template>

<script>
import Header from './components/Header.vue'
import SoftKeys from './components/SoftKeys.vue'
import Navigation from './Navigation.js'

export default {
  name: 'App',
  components: {
    Header,
    SoftKeys
  },
  data: function() {
    return {
       page: "main", // "main", "creating", "editing"
       grades: [],
       softkeys: {
            left: "",
            center: "Create",
            right: ""
        },

        filter_applied: 0, // 0 = None, 1 = Honors, 2 = Fails
        search_term: "",
        sort_grade_mark: undefined,
        sort_grade_course: true,
        editing_grade_index: -1,

        //Inputs
        input_grade_course: undefined, // Used for editing and creating pages
        input_grade_mark: undefined, // Used for editing and creating pages
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
    create: function() {
        console.log("Create a Grade");
        const isFocused = this.$refs.create.getAttribute('nav-selected');
        console.log( { isFocused } );
        if ( isFocused == "true" ) {
            console.log("Create is Focused");
        }
    },
    checkEnter: function() {
        const [currentElement] = Navigation.getCurrentItem();

        if ( currentElement.id === "create-grade-btn") {
            const isFocused = this.$refs.create.getAttribute('nav-selected');
            isFocused == "true" ? this.page = "creating" : "";
        
            setTimeout( function() {
                Navigation.init(); // Because elements were removed, we need to reinitialze our array of elements we can navigate again
            }, 10 );
              
        } else if ( currentElement.dataset.create_grade_input !== undefined ) {

            // Input Validation
            if ( this.input_grade_mark == undefined  || this.input_grade_course.trim() == "" ) return;
            if ( this.input_grade_mark > 100 || this.input_grade_mark < 0 ) return;

            this.grades.push( {
                "id": this.grades.length + 1,
                "course": this.input_grade_course.trim(),
                "mark": parseInt(this.input_grade_mark) 
            } );

            this.input_grade_course = undefined;
            this.input_grade_mark = undefined;

            this.page = "main";
            setTimeout( function() {
                Navigation.init(); // Because elements were removed, we need to reinitialze our array of elements we can navigate again
            }, 10 );

        }  else if ( currentElement.dataset.edit_grade_input !== undefined ) {

            // This normally would'nt be okay due to Vue's Reactivity System; however, since our grades will be rerenderd
            // when we naviagate to the Main page, out updates will be rendered 
            this.grades[ this.editing_grade_index ].course = this.input_grade_course.trim();
            this.grades[ this.editing_grade_index ].mark = this.input_grade_mark;

            this.input_grade_course = undefined;
            this.input_grade_mark = undefined;

            this.page = "main";
            setTimeout( function() {
                Navigation.init(); // Because elements were removed, we need to reinitialze our array of elements we can navigate again
            }, 10 );

        }
        else if ( currentElement.id === "sort-grade-course-btn") {
            this.sort_grade_mark = undefined;
            this.sort_grade_course = !this.sort_grade_course;
        }
        else if ( currentElement.id === "sort-grade-mark-btn") {
            this.sort_grade_course = undefined;
            this.sort_grade_mark = !this.sort_grade_mark;
        }
        // Below are the 3 Filter Options
        else if ( currentElement.id === "no-filter-btn") {
            this.filter_applied  = 0;
        }
        else if ( currentElement.id === "honors-filter-btn") {
            this.filter_applied  = 1;
        }
        else if ( currentElement.id === "fails-filter-btn") {
            this.filter_applied  = 2;
        }

    },
    checkRight: function() {
        
        const [currentElement] = Navigation.getCurrentItem();

        if ( currentElement.dataset.grade_id !== undefined ) {
            const grade_index = this.grades.indexOf( (grade) => grade.id == currentElement.dataset.grade_id );

            this.grades.splice(grade_index, 1);

            Navigation.Up(); // Theres a weird issue, this helps 
            this.updateSoftkeys();
        }

    },
    checkLeft: function() {
        
        const [currentElement] = Navigation.getCurrentItem();

        if ( currentElement.dataset.grade_id !== undefined ) {
            const grade_index = this.grades.indexOf( (grade) => grade.id == currentElement.dataset.grade_id );
            this.editing_grade_index = grade_index;

            this.page = "editing";
            const editSoftKeys = { left: "", center: "Save", right: "" };
            this.softkeys = editSoftKeys;

            // Update the Editing Inputs to have the Selected Grade Values
            this.input_grade_course = this.grades[ grade_index ].course;
            this.input_grade_mark = this.grades[ grade_index ].mark;

            setTimeout( function() {
                Navigation.init(); // Because elements were removed, we need to reinitialze our array of elements we can navigate again
            }, 10 );
        }

    },
    updateSoftkeys: function() {
        const inputSoftKeys = { left: "", center: "Create", right: "" };
        const taskSoftKeys = { left: "Edit", center: "", right: "Delete" };
        const toggleSoftKeys = { left: "", center: "Toggle", right: "" };

        const [currentElement ] = Navigation.getCurrentItem();

        if ( currentElement.id === "create-grade-btn") {
            this.softkeys = inputSoftKeys;
        } else if ( currentElement.dataset.grade_id !== undefined ){
            this.softkeys = taskSoftKeys;
        }
        else if (   currentElement.id === "sort-grade-course-btn" || 
                    currentElement.id === "sort-grade-mark-btn" ||
                    currentElement.id === "no-filter-btn" ||
                    currentElement.id === "honors-filter-btn" ||
                    currentElement.id === "fails-filter-btn"
            ) {
            this.softkeys = toggleSoftKeys;
        }
        else if ( currentElement.id === "search-grades" ) {
            this.softkeys = { left: "", center: "Search", right: "" };
        }

    },
    onKeyDown(event) {
      switch (event.key) {
        case "ArrowDown":
          Navigation.Down();
          this.updateSoftkeys();
          break;
        case "ArrowUp":
          Navigation.Up();
          this.updateSoftkeys();
          break;
        case "Enter":
          return this.checkEnter();
        case "ArrowRight":
        case "SoftRight":
          return this.checkRight();
        case "ArrowLeft":
        case "SoftLeft":
          return this.checkLeft();
        default:
          break;
      }
    },
  },
  mounted() {
      console.clear();
      Navigation.init();
      document.addEventListener("keydown", this.onKeyDown);
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.onKeyDown);
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

html,
body,
#app {
  font-family: "Open Sans", sans-serif;
  margin: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background-color: #f8f8f8;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

button {
    background-color: #c5c5c5;
    border: 1px solid #222;
    border-radius: 6px;
    margin: 4px 2px;
}

input {
    background-color: #e5e5e5;
    border: none;
    border-radius: 6px;
    margin: 4px 2px;
}

.task {
    margin: 6px 2px;
    width: calc( 100% - 4px ); /* 4px from the margin */
    border-radius: 6px;

    background-color: #E5E5E5;
}

.task span {
    border-radius: 6px;
    padding: 4px;
    display: flex;
    justify-content: space-between;
    align-item: center;
}

.task span[nav-selected="true"] {
  background-color: indianred;
  color: #FFF;
}

.buttons button, .inputs input {
    width: calc( 100% - 4px ); /* 4px from the button margin */
}

.buttons button[nav-selected="true"] {
  background-color: indianred;
  color: #FFF;
}
.inputs input[nav-selected="true"] {
  background-color: indianred;
  color: #FFF;
}

.filter-buttons {
    display: flex;
    justify-content: space-between;
}

.filter-buttons button[nav-selected="true"] {
    background-color: indianred;
    color: #FFF;
}

.selected-filter {
    color: green;
    background-color: rgb(200, 255, 200);
    border: none;
    border-radius: 6px;
}

.footer-stats {
    width: 100%;
    background-color: white;
    position: fixed;
    bottom: 30px;
    border-top: 1px solid #c5c5c5;
}
.footer-stats p {
    margin: 2px;
}

</style>
