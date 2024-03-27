<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-toolbar>
          <v-toolbar-title>Your Followers</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            label="Search"
            placeholder="Search followers"
          ></v-text-field>
          <v-menu
            v-model="dateMenu"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
          >
            <template v-slot:activator="{ on }">
              <v-text-field
                v-model="dateFilter"
                label="Date Filter"
                prepend-icon="mdi-calendar"
                readonly
                v-on="on"
              ></v-text-field>
            </template>
            <flat-pickr
              v-model="selectedDates"
              range
              @input="dateMenu = false"
              :config="dateConfig"
            ></flat-pickr>
          </v-menu>
          <v-spacer></v-spacer>
          <v-btn-toggle v-model="sortBy" mandatory>
            <v-btn @click="toggleSort('join_date')">Join Date</v-btn>
            <v-btn @click="toggleSort('twubric.total')">Twubric Score</v-btn>
            <v-btn @click="toggleSort('username')">Username</v-btn>
          </v-btn-toggle>
        </v-toolbar>
      </v-col>
    </v-row>
    <v-row>
      <v-col
        v-for="follower in filteredFollowers"
        :key="follower.uid"
        cols="12"
        sm="6"
        md="4"
        lg="3"
      >
        <v-card>
          <v-img :src="follower.image" height="200"></v-img>
          <v-card-text>
            <h2>{{ follower.username }}</h2>
            <p v-if="follower.fullname">Full Name: {{ follower.fullname }}</p>
            <p v-if="follower.twubric">
              Twubric Score: {{ follower.twubric.total }}
            </p>
            <p>Join Date: {{ formatDate(follower.join_date) }}</p>
            <v-btn @click="removeFollower(follower.uid)" color="error"
              >Remove</v-btn
            >
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import "@mdi/font/css/materialdesignicons.min.css";
import "vuetify/dist/vuetify.min.css";
import "flatpickr/dist/flatpickr.min.css";
// import flatpickr from 'flatpickr';

export default {
  data() {
    return {
      followers: [
        {
          uid: 1,
          username: "sampleuser1",
          image: "https://randomuser.me/api/portraits/men/1.jpg",
          fullname: "Sample User One",
          twubric: {
            total: 3.5,
            friends: 1,
            influence: 1,
            chirpiness: 1.5,
          },
          join_date: 1358899200,
        },
        {
          uid: 2,
          username: "sampleuser2",
          image: "https://randomuser.me/api/portraits/women/2.jpg",
          fullname: "Sample User Two",
          twubric: {
            total: 5,
            friends: 1,
            influence: 1,
            chirpiness: 1.5,
          },
          join_date: 1355270400,
        },
        {
          uid: 3,
          username: "sampleuser3",
          image: "https://randomuser.me/api/portraits/women/3.jpg",
          fullname: "Sample User Three",
          twubric: {
            total: 7,
            friends: 1,
            influence: 1,
            chirpiness: 1.5,
          },
          join_date: 1289433600,
        },
        {
          uid: 4,
          username: "sampleuser4",
          image: "https://randomuser.me/api/portraits/men/4.jpg",
          fullname: "Sample User Four",
          twubric: {
            total: 9,
            friends: 2,
            influence: 3,
            chirpiness: 4,
          },
          join_date: 1300838400,
        },
        {
          uid: 5,
          username: "sampleuser5",
          image: "https://randomuser.me/api/portraits/men/5.jpg",
          fullname: "Sample User Five",
          twubric: {
            total: 9,
            friends: 1,
            influence: 4,
            chirpiness: 4,
          },
          join_date: 1230768000,
        },
        {
          uid: 6,
          username: "sampleuser6",
          image: "https://randomuser.me/api/portraits/men/6.jpg",
          fullname: "Sample User Six",
          twubric: {
            total: 6,
            friends: 2,
            influence: 3,
            chirpiness: 1,
          },
          join_date: 1252454400,
        },
        {
          uid: 7,
          username: "sampleuser7",
          image: "https://randomuser.me/api/portraits/women/7.jpg",
          fullname: "Sample User Seven",
          twubric: {
            total: 8,
            friends: 2,
            influence: 4,
            chirpiness: 2,
          },
          join_date: 1278201600,
        },
        {
          uid: 8,
          username: "sampleuser8",
          image: "https://randomuser.me/api/portraits/women/8.jpg",
          fullname: "Sample User Eight",
          twubric: {
            total: 7,
            friends: 2,
            influence: 3,
            chirpiness: 2,
          },
          join_date: 1331510400,
        },
        {
          uid: 9,
          username: "sampleuser9",
          image: "https://randomuser.me/api/portraits/men/9.jpg",
          fullname: "Sample User Nine",
          twubric: {
            total: 8,
            friends: 1,
            influence: 4,
            chirpiness: 3,
          },
          join_date: 1367971200,
        },
        
      ],
      filteredFollowers: [],
      search: "",
      dateFilter: "",
      dateMenu: false,
      selectedDates: [],
      sortBy: null,
      sortDesc: false,
    };
  },
  computed: {
    dateConfig() {
      return {
        mode: "range",
        dateFormat: "Y-m-d",
        defaultDate: this.selectedDates,
      };
    },
  },
  mounted() {
    this.filteredFollowers = [...this.followers];
  },
  watch: {
    search() {
      this.filterFollowers();
    },
    selectedDates() {
      this.updateDateFilter();
      this.filterFollowers();
    },
  },
  methods: {
    toggleSort(field) {
      if (this.sortBy === field) {
        this.sortDesc = !this.sortDesc;
      } else {
        this.sortBy = field;
        this.sortDesc = false;
      }
      this.sortFollowers();
    },
    sortFollowers() {
      if (this.sortBy) {
        this.filteredFollowers.sort((a, b) => {
          const valA = this.getValue(a, this.sortBy);
          const valB = this.getValue(b, this.sortBy);
          if (valA < valB) return this.sortDesc ? 1 : -1;
          if (valA > valB) return this.sortDesc ? -1 : 1;
          return 0;
        });
      }
    },
    getValue(obj, key) {
      return key.split(".").reduce((acc, curr) => acc[curr], obj);
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return `${date.getFullYear()}-${(date.getMonth() + 1)
        .toString()
        .padStart(2, "0")}-${date.getDate().toString().padStart(2, "0")}`;
    },
    filterFollowers() {
      let filtered = [...this.followers];
      if (this.search) {
        const searchTerm = this.search.toLowerCase();
        filtered = filtered.filter((follower) => {
          return (
            follower.username.toLowerCase().includes(searchTerm) ||
            (follower.fullname &&
              follower.fullname.toLowerCase().includes(searchTerm))
          );
        });
      }
      if (this.selectedDates.length === 2) {
        const fromDate = this.selectedDates[0].getTime() / 1000;
        const toDate = this.selectedDates[1].getTime() / 1000;
        filtered = filtered.filter((follower) => {
          return follower.join_date >= fromDate && follower.join_date <= toDate;
        });
      }
      this.filteredFollowers = filtered;
      this.sortFollowers();
    },
    removeFollower(uid) {
      this.followers = this.followers.filter(
        (follower) => follower.uid !== uid
      );
      this.filterFollowers();
    },
    updateDateFilter() {
      if (this.selectedDates.length === 2) {
        const fromDate = this.formatDate(this.selectedDates[0]);
        const toDate = this.formatDate(this.selectedDates[1]);
        this.dateFilter = `${fromDate} to ${toDate}`;
      } else {
        this.dateFilter = "";
      }
    },
  },
};
</script>

<style>
/* Add your custom styles here */
</style>
