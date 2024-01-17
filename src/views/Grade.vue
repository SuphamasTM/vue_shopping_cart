<template>
  <div id="main">
    <div class="form">
      <h1>Grade calculator</h1>
      <v-form>
        <v-text-field v-model="score" label="Score" outlined></v-text-field>
        <v-text-field
          v-model="grade"
          label="Grade"
          outlined
          readonly
        ></v-text-field>
        <v-btn text color="teal accent-4" @click="calculateGrade(score)"
          >Calculate</v-btn
        >
      </v-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      score: "",
      grade: "",
    };
  },
  methods: {
    calculateGrade(score) {
      if (score > 100 || score < 0 || isNaN(score)) {
        this.grade = "Invalid Input";
      } else if (score >= 90) {
        this.grade = "A";
      } else if (score >= 80) {
        this.grade = "B";
      } else if (score >= 70) {
        this.grade = "C";
      } else if (score >= 60) {
        this.grade = "D";
      } else {
        this.grade = "F";
      }
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Patrick+Hand&display=swap");
#main {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: mintcream;
}

.form {
  padding: 20px;
  width: 300px;
  text-align: center;
}

* {
  font-family: "Patrick Hand", cursive;
}
</style>
