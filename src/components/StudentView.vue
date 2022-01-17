<template>
  <div>
    <v-container class="mt-5">
      <v-row justify="center">
        <v-stepper v-model="step" alt-labels min-width="400px">
          <v-stepper-header>
            <v-stepper-step :complete="step > 1" step="1"
              >日付指定</v-stepper-step
            >
            <v-divider></v-divider>
            <v-stepper-step :complete="step > 2" step="2"
              >氏名選択</v-stepper-step
            >
            <v-divider></v-divider>
            <v-stepper-step step="3">コメント</v-stepper-step>
          </v-stepper-header>
          <v-stepper-items>
            <v-stepper-content step="1">
              <v-card class="mb-5" max-hight="300px" flat>
                <v-date-picker
                  v-model="picker"
                  :landscape="false"
                ></v-date-picker>
              </v-card>
              <v-btn color="primary" @click="step = 2">次へ</v-btn>
            </v-stepper-content>
            <!--  -->

            <v-stepper-content step="2">
              <v-card class="mb-5" max-height="100px">
                <v-select
                  v-model="student"
                  :items="students"
                  required
                  label="氏名"
                ></v-select>
              </v-card>
              <v-btn @click="step = 1">前へ</v-btn>
              <v-btn color="primary" @click="step = 3">次へ</v-btn>
            </v-stepper-content>
            <!--  -->
            <v-stepper-content step="3">
              <v-card class="mb-5" max-height="200px">
                <v-text-field
                  v-model="comment"
                  label="コメント"
                  type="text"
                  auto-grow
                ></v-text-field>
              </v-card>
              <v-btn @click="step = 2">前へ</v-btn>
              <v-btn
                color="primary"
                @click="
                  step = 1;
                  save();
                "
                >登録</v-btn
              >
            </v-stepper-content>
          </v-stepper-items>
        </v-stepper>
      </v-row>
    </v-container>
  </div>
</template>

<script>
export default {
  props: ["students"],
  data() {
    return {
      step: 1,
      picker: new Date().toISOString().substr(0, 10),
      student: "",
      comment: "",
    };
  },
  methods: {
    save() {
      this.$emit("addrecord", {
        id: new Date().getTime(),
        date: this.picker,
        student: this.student,
        comment: this.comment,
      });
      this.student = "";
      this.comment = "";
    },
  },
};
</script>
