<template>
  <div class="main">
    <div class="form">
      <h1>Convert Unit</h1>
      <v-form>
        <v-text-field v-model="first" name="first" label="From"></v-text-field>
        <v-select
          :items="itemsFirst"
          v-model="firstUnit"
          label="Unit"
        ></v-select>

        <v-text-field
          v-model="second"
          name="second"
          label="To"
          readonly
        ></v-text-field>
        <v-select
          :items="itemsSecond"
          v-model="secondUnit"
          label="Unit"
        ></v-select>

        <v-btn color="primary" @click="convert(first, firstUnit, secondUnit)"
          >Convert</v-btn
        >
      </v-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      first: "",
      second: "",
      itemsFirst: ["mm", "cm", "m"],
      itemsSecond: ["mm", "cm", "m"],
    };
  },
  methods: {
    convert(first, firstUnit, secondUnit) {
      if (firstUnit == "mm" && secondUnit == "cm") {
        this.second = first / 10;
      } else if (firstUnit == "mm" && secondUnit == "m") {
        this.second = first / 1000;
      } else if (firstUnit == "cm" && secondUnit == "mm") {
        this.second = first * 10;
      } else if (firstUnit == "cm" && secondUnit == "m") {
        this.second = first / 100;
      } else if (firstUnit == "m" && secondUnit == "mm") {
        this.second = first * 1000;
      } else if (firstUnit == "m" && secondUnit == "cm") {
        this.second = first / 100;
      } else if (firstUnit == secondUnit) {
        this.second = first;
      }
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Patrick+Hand&display=swap");
.main {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: pink;
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
