<template>
  <div id="app">
    <input type="text" v-model="subject" placeholder="Subject" />
    <input type="number" v-model="courseNumber" />
    <input type="checkbox" v-model="fall" name="fallCheckbox" />
    <label for="fallCheckbox">Fall 2022</label>
    <input type="checkbox" v-model="spring" name="springCheckbox" />
    <label for="springCheckbox">Spring 2023</label>

    <table v-if="courses">
      <thead>
        <tr>
          <th
            v-for="heading in tableHeadings"
            :key="heading.value"
            @click="updateSort(heading.value)"
            :class="{
              active: sort.sortBy === heading.value,
              ascending: sort.sortBy === heading.value && !sort.descending,
              descending: sort.sortBy === heading.value && sort.descending,
            }"
          >
            {{ heading.text }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="[teacher, teacherData] in sortedTeachers" :key="teacher">
          <td>{{ teacher }}</td>
          <td>{{ teacherData.gpa.toPrecision(4) }}</td>
          <td>
            {{
              teacherData.avgRating !== -1
                ? teacherData.avgRating.toFixed(1)
                : "-"
            }}
          </td>
          <td>
            {{
              teacherData.avgDifficulty !== -1 ? teacherData.avgDifficulty : "-"
            }}
          </td>
          <td>
            {{
              teacherData.wouldTakeAgain !== -1
                ? teacherData.wouldTakeAgain.toFixed(0) + "%"
                : "-"
            }}
          </td>
          <td>{{ teacherData.students }}</td>
          <td>{{ teacherData.courses }}</td>
          <td>{{ teacherData.numRatings }}</td>
        </tr>
      </tbody>
    </table>
    <p>
      Created by
      <a href="https://github.com/AlexHalbesleben" target="_blank"
        >Alex Halbesleben</a
      >.<br />To get started, type any course's subject in the subject field and
      course number in the number field.<br />This has all courses from fall
      2022 and spring 2023 for which Texas A&M has released grade information.
    </p>
  </div>
</template>

<script lang="ts">
import * as json from "@/assets/all.json";
import * as ratings from "@/assets/ratings.json";
import { Component, Vue } from "vue-property-decorator";

@Component
export default class App extends Vue {
  subject = "";
  courseNumber = 0;
  get courseData() {
    return json as {
      [key: string]: {
        students: number;
        gpa: number;
        teacher: string;
        semester: string;
      }[];
    };
  }

  tableHeadings = [
    { text: "Teacher", value: "teacher" },
    { text: "GPA", value: "gpa" },
    { text: "Rating", value: "avgRating" },
    { text: "Difficulty", value: "avgDifficulty" },
    { text: "% would take again", value: "wouldTakeAgain" },
    { text: "Students", value: "students" },
    { text: "Courses", value: "courses" },
    { text: "Ratings", value: "numRatings" },
  ];

  sort = {
    sortBy: "teacher",
    descending: true,
  };

  updateSort(heading: string) {
    if (this.sort.sortBy === heading) {
      this.sort.descending = !this.sort.descending;
    } else {
      this.sort.sortBy = heading;
      this.sort.descending = true;
    }
  }

  fall = true;
  spring = true;

  get courseName(): string {
    return `${this.subject.toUpperCase()}-${this.courseNumber}`;
  }

  get courses():
    | { students: number; gpa: number; teacher: string; semester: string }[]
    | null {
    if (Object.keys(this.courseData).includes(this.courseName)) {
      return this.courseData[this.courseName].filter((c) => {
        if (c.semester === "fall_2022" && !this.fall) {
          return false;
        }
        if (c.semester === "spring_2023" && !this.spring) {
          return false;
        }
        return true;
      });
    }
    return null;
  }

  get teachers(): Record<
    string,
    {
      teacher: string;
      students: number;
      courses: number;
      gpa: number;
      avgRating: number;
      avgDifficulty: number;
      wouldTakeAgain: number;
      numRatings: number;
    }
  > {
    let obj: Record<
      string,
      {
        teacher: string;
        students: number;
        courses: number;
        gpa: number;
        avgRating: number;
        avgDifficulty: number;
        wouldTakeAgain: number;
        numRatings: number;
      }
    > = {};
    let list: string[] = Array.from(
      new Set(this.courses?.map((c) => c.teacher) || [])
    );
    for (let teacher of list) {
      obj[teacher] = { teacher, ...this.teacherInformation(teacher) };
    }

    return obj;
  }

  get sortedTeachers() {
    let base = Object.entries(this.teachers) as [
      string,
      Record<string, string | number>
    ][];

    base.sort((a, b) => {
      let aData = a[1][this.sort.sortBy];
      let bData = b[1][this.sort.sortBy];

      let comparison = 0;

      if (typeof aData === "string" && typeof bData === "string") {
        comparison = (aData as string).localeCompare(bData as string) * -1;
      } else if (typeof aData === "number" && typeof bData === "number") {
        comparison = (aData as number) - (bData as number);
      }
      return comparison * (this.sort.descending ? -1 : 1);
    });

    return base as [
      string,
      {
        students: number;
        courses: number;
        gpa: number;
        avgRating: number;
        avgDifficulty: number;
        wouldTakeAgain: number;
        numRatings: number;
      }
    ][];
  }

  teacherInformation(teacher: string): {
    students: number;
    courses: number;
    gpa: number;
    avgRating: number;
    avgDifficulty: number;
    wouldTakeAgain: number;
    numRatings: number;
  } {
    let students = 0;
    let totalGPA = 0;
    let courses = 0;

    for (let course of this.courses || []) {
      if (course.teacher !== teacher) {
        continue;
      }
      students += course.students;
      totalGPA += course.gpa * course.students;
      courses += 1;
    }

    return {
      students,
      courses,
      gpa: totalGPA / students,
      avgRating: this.ratings[teacher]?.avgRating || -1,
      avgDifficulty: this.ratings[teacher]?.avgDifficulty || -1,
      wouldTakeAgain: this.ratings[teacher]?.wouldTakeAgain || -1,
      numRatings: this.ratings[teacher]?.numRatings || 0,
    };
  }

  get ratings() {
    return ratings as {
      [key: string]: {
        avgRating: number;
        avgDifficulty: number;
        wouldTakeAgain: number;
        numRatings: number;
      };
    };
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

input {
  max-width: 3.5em;
}

table {
  table-layout: fixed;
  width: 100%;
  border-collapse: collapse;
  border: 1px solid gray;
  margin-top: 2px;
}

th,
td {
  padding: 10px;
  padding-top: 3px;
  padding-bottom: 3px;
}

tbody td {
  text-align: center;
}

tfoot th {
  text-align: right;
}

tbody tr:nth-child(even),
thead {
  background-color: #f2f2f2;
}

th.active::after {
  color: black;
  font-size: 1.25rem;
}

th.ascending::after {
  content: "↑";
}

th.descending::after {
  content: "↓";
}

/* 
Max width before this PARTICULAR table gets nasty
This query will take effect for any screen smaller than 760px
and also iPads specifically.
*/
@media only screen and (max-width: 760px),
  (min-device-width: 768px) and (max-device-width: 1024px) {
  /* Force table to not be like tables anymore */
  table,
  thead,
  tbody,
  th,
  td,
  tr {
    display: block;
  }

  td {
    padding: inherit;
  }

  /* Hide table headers (but not display: none;, for accessibility) */
  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr {
    border: 1px solid #ccc;
  }

  td {
    /* Behave  like a "row" */
    border: none;
    border-bottom: 1px solid #eee;
    position: relative;
    padding-left: 50%;
  }

  tr:nth-child(even) td {
    border-bottom: 1px solid white;
  }

  td:before {
    /* Now like a table header */
    position: absolute;
    /* Top/left values mimic padding */
    top: 0px;
    left: 6px;
    width: 45%;
    padding-right: 10px;
    white-space: nowrap;
  }

  /*
	Label the data
	*/
  td:nth-of-type(1):before {
    content: "Teacher";
  }
  td:nth-of-type(2):before {
    content: "Average GPA";
  }
  td:nth-of-type(3):before {
    content: "Average rating";
  }
  td:nth-of-type(4):before {
    content: "Average difficulty";
  }
  td:nth-of-type(5):before {
    content: "% would take again";
  }
  td:nth-of-type(6):before {
    content: "Students taught";
  }
  td:nth-of-type(7):before {
    content: "Courses taught";
  }
  td:nth-of-type(8):before {
    content: "Number of ratings";
  }
}
</style>
