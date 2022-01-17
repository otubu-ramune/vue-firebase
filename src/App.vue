<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span class="font-weight-light ml-4">レッスン記録</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <span>{{ name }}</span>
      <v-btn
        text
        @click.stop="
          message = '';
          dialog = true;
        "
        v-if="!(isStudent || isTeacher)"
        >ログイン</v-btn
      >
      <v-btn
        text
        @click.stop="
          logout();
          Msgdialog = true;
          msg = 'ログアウトしました';
        "
        v-if="isStudent || isTeacher"
        >ログアウト</v-btn
      >
    </v-app-bar>
    <v-main>
      <v-dialog
        v-model="Msgdialog"
        scrollable
        persistent
        :overlay="true"
        max-width="300px"
        transition="dialog-transition"
      >
        <v-card>
          <v-toolbar dark color="primary">
            <v-toolbar-title><span>お知らせ</span></v-toolbar-title>
          </v-toolbar>
          <v-spacer></v-spacer>
          <v-card-text class="mt-5 text--primary text-h6">
            {{ msg }}</v-card-text
          >

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              text
              @click="
                Msgdialog = false;
                msg = '';
              "
              >閉じる</v-btn
            >
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-model="dialog" max-width="400">
        <v-card class="elevation-12">
          <v-toolbar dark color="primary">
            <v-toolbar-title><span>ログイン</span></v-toolbar-title>
          </v-toolbar>

          <v-card-text>
            <v-form>
              <v-text-field
                prepend-icon="mdi-account"
                v-model="email"
                label="E-mail"
                type="text"
                required
                autofocus
              ></v-text-field>
              <v-text-field
                prepend-icon="mdi-lock"
                v-model="password"
                label="パスワード"
                type="password"
                required
              ></v-text-field>
            </v-form>
            <span>{{ message }}</span>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="login"> ログイン </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <student-view v-on:addrecord="addRecord" :students="students" />

      <teacher-view
        v-if="isTeacher"
        v-on:updateRecord="updateRecord"
        :students="students"
        :teachers="teachers"
        :records="records"
      />

      <span>
        <v-btn color="success" @click="printData">表示</v-btn>
        <v-textarea
          name="input-7-1"
          filled
          label="Label"
          auto-grow
          v-model="testmessage"
        ></v-textarea>
      </span>
    </v-main>
  </v-app>
</template>

<script>
import { initializeApp } from "firebase/app";
import {
  getAuth,
  connectAuthEmulator,
  onAuthStateChanged,
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut,
} from "firebase/auth";
import {
  getFirestore,
  connectFirestoreEmulator,
  collection,
  onSnapshot,
  getDocs,
  getDoc,
  doc,
  addDoc,
  query,
  where,
  updateDoc,
} from "firebase/firestore";

const Config = {
  apiKey: process.env.VUE_APP_apiKey,
  authDomain: process.env.VUE_APP_authDomain,
  projectId: process.env.VUE_APP_projectId,
  storageBucket: process.env.VUE_APP_storageBucket,
  messagingSenderId: process.env.VUE_APP_messagingSenderId,
  appId: process.env.VUE_APP_appId,
  measurementId: process.env.VUE_APP_measurementId,
};
console.log(Config);
const firebaseConfig = {
  apiKey: "AIzaSyAe7tP_70p4Qrcz-gkH8GYQzVSkjLnr_20",
  authDomain: "auth-test-c6e4d.firebaseapp.com",
  projectId: "auth-test-c6e4d",
  storageBucket: "auth-test-c6e4d.appspot.com",
  messagingSenderId: "276070869624",
  appId: "1:276070869624:web:b4ea185d920db01e551b8d",
  measurementId: "G-9QLNTYX96T",
};

// Initialize Firebase

const app = initializeApp(Config); //process.env.VUE_APP_CONFIGS);
// firebaseApps previously initialized using initializeApp()
const db = getFirestore(app);
connectFirestoreEmulator(db, "192.168.0.5", 8081);

const auth = getAuth();
connectAuthEmulator(auth, "http://192.168.0.5:9099");

import studentView from "./components/StudentView.vue";
import teacherView from "./components/TeacherView.vue";
export default {
  components: {
    studentView,
    teacherView,
  },
  mounted() {
    // const db = getFirestore();
    // connectFirestoreEmulator(db, "localhost", 8081);
    onSnapshot(collection(db, "records"), (snapshot) => {
      this.records = [];
      snapshot.forEach((doc) => {
        this.records.push({ id: doc.id, ...doc.data() });
      });
      console.log("records", this.records);
    });

    onSnapshot(doc(db, "settings", "students"), (doc) => {
      this.students = doc.data().names;
      console.log("Current data: ", doc.data().names);
    });

    onSnapshot(collection(db, "users"), (snapshot) => {
      this.user = [];

      snapshot.forEach((doc) => {
        let user = doc.data();
        if (user.role == "teacher") {
          this.teachers.push(user.name);
        }
      });
      console.log("teacher", this.teachers);
    });

    onAuthStateChanged(auth, (user) => {
      this.isStudent = false;
      this.isTeacher = false;
      if (user) {
        onSnapshot(doc(db, "users", user.uid), (doc) => {
          let user = doc.data().name;
          let role = doc.data().role;

          console.log("Auth state changed: ", user);
          if (role == "teacher") {
            this.isTeacher = true;
          } else {
            this.isStudent = true;
          }
        });
      }
    });
  },

  data: () => ({
    name: "",
    email: "",
    password: "",
    dialog: false,
    Msgdialog: false,
    students: [],
    teachers: [],
    records: [],
    message: "",
    msg: "",
    isStudent: false,
    isTeacher: false,
    testmessage: "test",
  }),
  methods: {
    async setUsrName(userCre) {
      const docSnap = await getDoc(doc(db, "users", userCre.user.uid));
      let user = docSnap.data().name;
      let role = docSnap.data().role;
      this.msg = "ようこそ" + user;
      if (role == "teacher") {
        this.msg += "先生";
      } else {
        this.msg += "さん";
      }
    },
    login() {
      this.dialog = true;

      signInWithEmailAndPassword(auth, this.email, this.password)
        .then((userCredential) => {
          // Signed in
          this.dialog = false;
          console.log("login");
          this.setUsrName(userCredential);
          this.Msgdialog = true;
          // ...
        })
        .catch((error) => {
          const errorCode = error.code;
          this.message = error.message;
          // ..
        });
    },
    logout() {
      signOut(auth)
        .then(() => {
          // Sign-out successful.
        })
        .catch((error) => {
          // An error happened.
        });
      this.name = "";
      this.email = "";
      this.password = "";
      this.dialog = false;
      this.isStudent = false;
      this.isTeacher = false;
    },

    async addRecord(record) {
      this.records.push(record);
      await addDoc(collection(db, "records"), record);
    },
    updateRecord(record) {
      this.records.forEach((r) => {
        if (record.id == r.id) {
          // this.$set(r, "memo", record.memo);
          // this.$set(r, "teacher", record.teacher);
          (async function () {
            await updateDoc(doc(db, "records", record.id), {
              memo: record.memo,
              teacher: record.teacher,
            });
          });
          // await updateDoc(doc(db, "records", record.id), {
          //   memo: record.memo,
          //   teacher: record.teacher,
          // });
        }
      });
    },
    async printData() {
      const docRef = doc(db, "settings", "students");
      const docSnap = await getDoc(docRef);

      if (docSnap.exists()) {
        console.log({ name: docSnap.data().names });
        docSnap.data().names.forEach((name) => {
          console.log(name);
        });
        this.testmessage = docSnap.data().names.join("\n");
      } else {
        // doc.data() will be undefined in this case
        console.log("No such document!");
      }
    },
  },
};
</script>
