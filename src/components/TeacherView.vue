<template>
  <div>
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-text>
          <v-container grid-list-md>
            <v-select
              :items="teachers"
              v-model="teacher"
              label="教師"
              required
            ></v-select>
            <v-textarea
              v-model="memo"
              label="メモ"
              required
              auto-grow
              box
            ></v-textarea>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="dialog = false"
            >Cancel</v-btn
          >
          <v-btn color="blue darken-1" text @click="saveItem">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-container class="mt-5">
      <v-select
        :items="students"
        v-model="student"
        label="氏名"
        required
      ></v-select>
      <span>{{ records }}</span>
      <v-data-table
        :headers="headers"
        :items="selectedRecords"
        class="elevation-1"
      >
        <template v-slot:item.edit="{ item }">
          <v-btn @click="editItem(item)">編集</v-btn>
        </template>
      </v-data-table>
    </v-container>
  </div>
</template>

<script>
let headers = [
  {
    text: "日付",
    value: "date",
  },
  {
    text: "コメント",
    value: "comment",
  },
  {
    text: "教師",
    value: "teacher",
  },
  {
    text: "メモ",
    value: "memo",
  },
  {
    text: "登録",
    value: "edit",
  },
];
export default {
  props: ["records", "students", "teachers"],
  data() {
    return {
      headers: headers,
      dialog: false,
      student: "",
      teacher: "",
      memo: "",
      item: null,
    };
  },
  computed: {
    selectedRecords() {
      return this.records.filter((item) => {
        return item.student == this.student;
      });
    },
  },
  methods: {
    editItem(item) {
      this.item = item;
      this.memo = item.memo;
      this.dialog = true;
    },
    saveItem() {
      this.$emit("updateRecord", {
        id: this.item.id,
        teacher: this.teacher,
        memo: this.memo,
      });
      this.dialog = false;
    },
  },
};
</script>
