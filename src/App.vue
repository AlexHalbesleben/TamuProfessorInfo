<template>
  <div id="app">
    <input type="text" v-model="subject" placeholder="Subject" />
    <input type="number" v-model="courseNumber" />
    <input type="checkbox" v-model="fall" name="fallCheckbox" />
    <label for="fallCheckbox">Fall 2022</label>
    <input type="checkbox" v-model="spring" name="springCheckbox" />
    <label for="springCheckbox">Spring 2023</label>

    <table v-if="courses">
      <tr>
        <th>Teacher</th>
        <th>Average GPA</th>
        <th>Average rating</th>
        <th>Average difficulty</th>
        <th>% would take again</th>
        <th>Students taught</th>
        <th>Courses taught</th>
        <th># of ratings</th>
      </tr>
      <tr
        v-for="[teacher, teacherData] in Object.entries(teachers)"
        :key="teacher"
      >
        <td>{{ teacher }}</td>
        <td>{{ teacherData.gpa.toPrecision(4) }}</td>
        <td>
          {{
            ratings[teacher] &&
            ratings[teacher].avgRating &&
            ratings[teacher].avgRating !== -1
              ? ratings[teacher].avgRating.toFixed(1)
              : ""
          }}
        </td>
        <td>
          {{
            ratings[teacher] &&
            ratings[teacher].avgDifficulty &&
            ratings[teacher].avgDifficulty !== -1
              ? ratings[teacher].avgDifficulty
              : ""
          }}
        </td>
        <td>
          {{
            ratings[teacher] &&
            ratings[teacher].wouldTakeAgain &&
            ratings[teacher].wouldTakeAgain !== -1
              ? ratings[teacher].wouldTakeAgain.toFixed(0) + "%"
              : ""
          }}
        </td>
        <td>{{ teacherData.students }}</td>
        <td>{{ teacherData.courses }}</td>
        <td>{{ ratings[teacher]?.numRatings }}</td>
      </tr>
    </table>
    <p style="position: absolute; bottom: 0">
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

  fall = true;
  spring = true;

  get courseName(): string {
    return `${this.subject}-${this.courseNumber}`;
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
      students: number;
      courses: number;
      gpa: number;
    }
  > {
    let obj: Record<
      string,
      {
        students: number;
        courses: number;
        gpa: number;
      }
    > = {};
    let list: string[] = Array.from(
      new Set(this.courses?.map((c) => c.teacher) || [])
    );
    for (let teacher of list) {
      obj[teacher] = this.teacherInformation(teacher);
    }

    return obj;
  }

  teacherInformation(teacher: string): {
    students: number;
    courses: number;
    gpa: number;
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
}

thead th:nth-child(1) {
  width: 30%;
}

thead th:nth-child(2) {
  width: 20%;
}

thead th:nth-child(3) {
  width: 15%;
}

thead th:nth-child(4) {
  width: 35%;
}

th,
td {
  padding: 20px;
}

th {
  letter-spacing: 2px;
}

td {
  letter-spacing: 1px;
}

tbody td {
  text-align: center;
}

tfoot th {
  text-align: right;
}
</style>
