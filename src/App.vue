<template>
  <div id="app" class="container mt-4 theme-light">
    <h2 class="header">Texas A&M Course Lookup</h2>
    <div class="form-group">
      <label for="courseInput" class="label">Subject & Course Number</label>
      <input
        id="courseInput"
        type="text"
        v-model="course"
        placeholder="MATH 251"
        class="form-control input"
        :class="{
          'border-green': isCourseValid,
          'border-red': isCourseInvalid,
          'border-black': course == '',
        }"
      />
    </div>
    <div v-if="isCourseNotFound" class="course-not-found">
      Course not found. Please try again.
    </div>

    <div class="form-check form-check-inline">
      <input
        id="fallCheckbox"
        type="checkbox"
        v-model="fall"
        class="form-check-input checkbox"
      />
      <label for="fallCheckbox" class="form-check-label label">Fall 2022</label>
    </div>
    <div class="form-check form-check-inline">
      <input
        id="springCheckbox"
        type="checkbox"
        v-model="spring"
        class="form-check-input checkbox"
      />
      <label for="springCheckbox" class="form-check-label label"
        >Spring 2023</label
      >
    </div>
    <transition name="fade">
      <table v-if="courses" class="table table-striped mt-4 table-theme">
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
            <td>
              <a
                :href="`https://ratemyprofessors.com/professor/${teacherData.id}`"
                >{{ teacher }}</a
              >
            </td>
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
                teacherData.avgDifficulty !== -1
                  ? teacherData.avgDifficulty
                  : "-"
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
    </transition>

    <p class="mt-4 footer">
      Created by
      <a href="https://github.com/AlexHalbesleben" target="_blank" class="link">
        Alex Halbesleben</a
      >
      with help from
      <a href="https://github.com/T-Lind" target="_blank" class="link">
        Tiernan Lindauer</a
      >.
      <br />
      To get started, type any course's subject and course number in the field.
      <br />
      This has all courses from fall 2022 and spring 2023 for which Texas A&M
      has released grade information.
      <br />
      Found a bug or want to contribute? File an issue or PR
      <a
        href="https://github.com/AlexHalbesleben/tamuprofessorinfo/issues"
        target="_blank"
        class="link"
        >here</a
      >.
      <br />
      <br />
      <strong>
        Disclaimer: This is not an official Texas A&M website. The goal of this
        project is to make it easier for students to find information about
        courses and professors. Students should always check official TAMU
        websites for the most up-to-date information.
      </strong>
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

  course = "";

  get courseName(): string {
    const [subject, courseNumber] = this.course.split(" ");
    return `${subject.toUpperCase()}-${courseNumber}`;
  }

  get isCourseValid(): boolean {
    return this.course !== "";
  }

  get isCourseInvalid(): boolean {
    return this.course !== "" && !this.isCourseValid;
  }

  get isCourseNotFound() {
    return this.courseName && this.courseName !== "-undefined" && !this.courses;
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
        id: number;
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
    id: number;
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
      id: this.ratings[teacher]?.id || 0,
    };
  }

  get ratings() {
    return ratings as {
      [key: string]: {
        avgRating: number;
        avgDifficulty: number;
        wouldTakeAgain: number;
        numRatings: number;
        id: number;
      };
    };
  }
}
</script>

<style scoped>
@import "@/components/styles.css";
</style>
