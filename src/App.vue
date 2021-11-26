<template>
  <MainPage v-if="page === 'main'" 
    :grades="grades" 
    :softkeys="softkeys" 
    :filter_applied="filter_applied"
    :sort_grade_mark="sort_grade_mark"
    :sort_grade_course="sort_grade_course" />
  <EditingPage v-else-if="page === 'editing'" 
    :softkeys="softkeys"
    :grades="grades"
    :editing_index="editing_grade_index"
    @input_grade_course_update="(value) => { input_grade_course = value } " 
    @input_grade_mark_update=" (value) => { input_grade_mark = value } " />
  <CreatingPage v-else-if="page === 'creating'" 
    :softkeys="softkeys"
    @input_grade_course_update="(value) => { input_grade_course = value } " 
    @input_grade_mark_update=" (value) => { input_grade_mark = value } " />
</template>

<script>
import MainPage from './components/MainPage.vue'
import EditingPage from './components/EditingPage.vue'
import CreatingPage from './components/CreatingPage.vue'
import Navigation from './Navigation.js'

export default {
  name: 'App',
  components: {
    EditingPage,
    CreatingPage,
    MainPage
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
        sort_grade_mark: undefined,
        sort_grade_course: true,
        editing_grade_index: -1,

        //Inputs
        input_grade_course: undefined, // Used for editing and creating pages
        input_grade_mark: undefined, // Used for editing and creating pages
    };
  },
  methods: {
    checkEnter: function() {
        const [currentElement] = Navigation.getCurrentItem();

        if ( currentElement.id === "create-grade-btn") {
            //const isFocused = this.$refs.create.getAttribute('nav-selected');
            //isFocused == "true" ? this.page = "creating" : "";
        
            this.page = "creating";
            setTimeout( function() {
                Navigation.init(); // Because elements were removed, we need to reinitialze our array of elements we can navigate again
            }, 10 );
              
        } else if ( currentElement.dataset.create_grade_input !== undefined ) {

            // Input Validation
            if ( this.input_grade_mark == undefined  || this.input_grade_course.trim() == "" ) return;
            if ( this.input_grade_mark > 100 || this.input_grade_mark < 0 || isNaN(this.input_grade_mark) ) return;

            this.grades.push( {
                "id": Date.now(),
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

            // Input Validation
            if ( this.input_grade_mark == undefined  || this.input_grade_course.trim() == "" ) return;
            if ( this.input_grade_mark > 100 || this.input_grade_mark < 0 || isNaN(this.input_grade_mark) ) return;


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

            Navigation.init(); // Theres a weird issue, this helps 
        }

    },
    checkLeft: function() {
        
        const [currentElement] = Navigation.getCurrentItem();

        if ( currentElement.dataset.grade_id !== undefined ) {
            //const grade_index = this.grades.indexOf( (grade) => grade.id == currentElement.dataset.grade_id );
            // Idk why the code above doesnt work here... yet it works in checkRight()

            let grade_index = -1;
            for(let i = 0; i < this.grades.length; i++) {
                if ( this.grades[i].id == currentElement.dataset.grade_id ) {
                    grade_index = i;
                    break;
                }
            }

            this.editing_grade_index = grade_index;

            this.page = "editing";
            const editSoftKeys = { left: "", center: "Save", right: "" };
            this.softkeys = editSoftKeys;

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
