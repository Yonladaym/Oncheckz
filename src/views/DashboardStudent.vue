<template>
  <v-card class="mx-auto overflow-y-auto overflow-x-hidden" height="100vh" width="100%">
    <v-app-bar
      class="dark-blue accent-4 h-70 d-flex justify-center flex-column position-fixed"
      dark
      prominent
    >
      <v-toolbar-title>
        <img src="../assets/logo2.svg" width="150px" @click="goHome()" />
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-app-bar-nav-icon @click.stop="drawer = !drawer" class="d-flex align-self-center"></v-app-bar-nav-icon>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" absolute right temporary style="width:300px;">
      <v-list nav dense>
        <v-list-item-group
          v-model="group"
          active-class="deep-purple--text text--accent-4"
          class="d-flex flex-column"
        >
          <div class="align-self-center text-center mrg-t">
            <HomePhoto></HomePhoto>
          </div>

          <v-list-item @click="goHome()">
            <v-list-item-title class="menu d-flex align-items-center">
              <div style="margin:10px;">
                <img src="../assets/home-run.svg" width="30px" />
              </div>
              <div>Home</div>
            </v-list-item-title>
          </v-list-item>

          <v-list-item @click="goDashboard()">
            <v-list-item-title class="menu d-flex align-items-center">
              <div style="margin:10px;">
                <img src="../assets/edit.svg" width="30px" />
              </div>
              <div>Dashboard</div>
            </v-list-item-title>
          </v-list-item>

          <v-list-item @click="logOut()">
            <v-list-item-title class="menu d-flex align-items-center" style="color:#E97B6E;">
              <div style="margin:10px;">
                <img src="../assets/logout.svg" width="30px" />
              </div>
              <div>log out</div>
            </v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>

    <div class="up d-flex align-content-around flex-column">
      <v-container class="layout">
        <div class="align-self-center text-center mrg-t">
          <HomePhoto></HomePhoto>
        </div>
        <div class="align-self-center" style="width:90%;">
          <h5 style="color:red;font-size:16px" v-show="isSelected">Subject is required</h5>
          <v-select
            v-model="subject"
            :items="items"
            :menu-props="{ bottom: true, offsetY: true }"
            label="รายวิชา"
          ></v-select>
          <div class="d-flex justify-content-end">
            <v-btn @click="search()">Search</v-btn>
          </div>
        </div>
        <div class="d-flex justify-content-between" style="margin-top:30px;">
          <h5 class="text-left" style="opacity:0.5">ประวัติการเช็คชื่อ</h5>
        </div>

        <!-- card ประวัติเช็คชื่อ -->
        <div
          class="d-flex align-items-center flex-column"
          style="margin-top:15px; overflow-y: auto; width:100%;"
        >
          <div
            class="card border-0 rounded-pill"
            style="width:100%; margin-bottom: 20px; height:65px;"
            v-for="(h, index) in histories"
            :key="index"
          >
            <div class="card-body d-flex justify-content-between">
              <div class="card-title">
                <h6>{{h}}</h6>
              </div>
              <div>
                <h6>เช็คชื่อสำเร็จ</h6>
              </div>
            </div>
          </div>
        </div>
      </v-container>
    </div>
  </v-card>
</template>

<script>
import HomePhoto from "@/components/HomePhoto.vue";
import firebase from "firebase";
export default {
  name: "DashboardStudent",
  components: {
    HomePhoto
  },
  data: () => ({
    drawer: false,
    group: null,
    subject: "",
    items: [],
    histories: [],
    isSelected: false
  }),

  watch: {
    group() {
      this.drawer = false;
    }
  },
  created() {
    this.getHistories();
    this.getCourses();
  },
  methods: {
    async search() {
      if (this.subject) {
        this.isSelected = false;
        const courseId = this.subject.split(" | ")[0];
        const course = this.$store.state.courses.filter(c => {
          return c.courseId == courseId;
        })[0];
        await this.$store.dispatch("getHistory", {
          classId: course.id,
          email: this.$store.state.user.email
        });
        location.reload();
      } else {
        this.isSelected = true;
      }
    },
    getHistories() {
      this.histories = this.$store.getters.getHistories();
    },
    getCourses() {
      const dumps = this.$store.state.courses;
      dumps.forEach(dump => {
        if (dump.students) {
          for (let i = 0; i < dump.students.length; i++) {
            if (dump.students[i] == this.$store.state.user.email) {
              this.items.push(dump.courseId + " | " + dump.courseName);
            }
          }
        } else {
          if (!this.courses) {
            this.courses = [];
          }
        }
      });
    },
    goHome() {
      this.$router.push({ name: "HomeStudent" });
    },
    goDashboard() {
      this.$router.push({ name: "DashboardStudent" });
    },
    async logOut() {
      await firebase.auth().signOut();
      this.$store.commit("CLEAR_STATE");
      this.$router.push({ name: "Login" });
    }
  }
};
</script>

<style lang="scss" scoped>
.up {
  // height: 100vh;
  background: #ffff;
}

.layout {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.mrg-t {
  margin-top: 5rem !important;
}

.card {
  background-color: #f2f5f9 !important;
  // border-radius: 25px;
  box-shadow: 7px 17px 8px -7px rgba(0, 0, 0, 0.09);
}
.dark-blue {
  background: #3b5185 !important;
}
.h-70 {
  height: 70px !important;
}
.menu {
  font-size: 1.2rem !important;
  padding: 20px !important;
}
</style>