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
        <td>{{ teacherInfo[teacher]?.avgRating }}</td>
        <td>{{ teacherInfo[teacher]?.avgDifficulty }}</td>
        <td>{{ teacherInfo[teacher]?.wouldTakeAgain.toPrecision(3) }}%</td>
        <td>{{ teacherData.students }}</td>
        <td>{{ teacherData.courses }}</td>
        <td>{{ teacherInfo[teacher]?.numRatings }}</td>
      </tr>
    </table>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import * as json from "@/assets/all.json";
import ratings from "@/assets/rmp/@mtucourses/rate-my-professors";

console.log(ratings);

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

    if (!this.teacherInfo[teacher]) {
      this.getRMP(teacher);
    }

    return {
      students,
      courses,
      gpa: totalGPA / students,
    };
  }

  TAMU_ID = "U2Nob29sLTEwMDM=";

  teacherInfo: Record<
    string,
    {
      avgRating: number;
      avgDifficulty: number;
      wouldTakeAgain: number;
      numRatings: number;
    }
  > = {};

  async getRMP(name: string) {
    let teachers = await ratings.searchTeacher(name, this.TAMU_ID);
    if (teachers.length === 0) {
      return;
    }
    let teacherID = teachers[0].id;
    let rating = await ratings.getTeacher(teacherID);
    let ret = {
      avgRating: rating.avgRating,
      avgDifficulty: rating.avgDifficulty,
      // eslint-disable-next-line @typescript-eslint/ban-ts-comment
      // @ts-ignore
      wouldTakeAgain: rating.wouldTakeAgainPercent,
      numRatings: rating.numRatings,
    };

    Vue.set(this.teacherInfo, name, ret);
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
