<script>
import { defineComponent } from "vue";
import moment from "moment";
import GoogleMapsModal from "./GoogleMapsModal.vue";

export default defineComponent({
  props: ["post"],
  data() {
    return {
      iframesrc: this.post.location
        ? `https://www.google.com/maps/embed/v1/place?key=AIzaSyDPvCQ4RXgvhbboTmKh2qLnfY50aJxcD0E&q=${this.post.location._latitude}, ${this.post.location._longitude}`
        : "",
      reverseGeo: "",
      revbgeo: "",
      showModal: false,
    };
  },
  methods: {
    postdate() {
      return moment(this.post.creationDate._seconds * 1000).format(
        "MM-DD-YYYY h:mm:ss"
      );
    },
    boldusername(s) {
      return s.replace(
        /@\w+/g,
        "<span class='font-bold text-blue-200'>$&</span>"
      );
    },
  },
  computed: {
    color() {
      var letters = "0123456789ABCDEF";
      var color = "";
      for (var i = 0; i < 6; i++) {
        color += letters[this.post.user.username.charCodeAt(i) % 16];
      }
      return color;
    },
  },
  async beforeMount() {
    if (this.post.location) {
      // `https://nominatim.openstreetmap.org/reverse?format=json&lat=${this.post.location._latitude}&lon=&zoom=18&addressdetails=1`,
      fetch(
        `https://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer/reverseGeocode?location=${this.post.location._longitude},${this.post.location._latitude}&langCode=fr&outSR=&forStorage=false&f=pjson`
      )
        .then((res) => res.json())
        .then((data) => {
          this.revbgeo = data;
          this.reverseGeo = data.address.City + ", " + data.address.CntryName;
        })
        .catch((err) => {
          console.log("error in reverse geocoding");
        });
    }
  },
  components: { GoogleMapsModal },
});
</script>
<template>
  <div
    class="block p-3 w-[100%] sm:p-6, sm:w-auto bg-white rounded-lg border border-gray-200 shadow-md hover:bg-gray-100 dark:bg-gray-800 dark:border-gray-700 dark:hover:bg-gray-700"
  >
    <div class="flex flex-col">
      <div class="flex items-center sm:justify-center">
        <img
          referrerpolicy="no-referrer"
          v-bind:src="
            post.user.profilePicture
              ? post.user.profilePicture.url
              : 'https://ui-avatars.com/api/?length=1' +
                '&name=' +
                post.user.username +
                '&background=' +
                color
          "
          class="w-10 rounded-[50%] sm:w-28"
          @error="
            'https://ui-avatars.com/api/?length=1' +
              '&name=' +
              post.user.username +
              '&background=' +
              color
          "
        />
        <div>
          <div>
            <span class="font-bold ml-3">
              {{ post.user.username }}
            </span>
          </div>
          <div class="mt-[-3%]">
            <span class="ml-3 text-sm cursor-pointer" @click="showModal = true">
              {{ reverseGeo }}
            </span>
          </div>
        </div>
        <GoogleMapsModal v-if="showModal" @close="showModal = false">
          <template v-slot:body>
            <iframe
              class="ml-3"
              width="300"
              height="300"
              style="border: 0"
              loading="lazy"
              v-if="post.location"
              allowfullscreen
              referrerpolicy="no-referrer-when-downgrade"
              :src="iframesrc"
            >
            </iframe>
          </template>
        </GoogleMapsModal>
      </div>
      <div class="flex items-center justify-center flex-col mt-4 sm:flex-row">
        <img
          referrerpolicy="no-referrer"
          v-bind:src="post.photoURL"
          class="h-full w-full ml-3 sm:w-64 rounded-md mb-3"
        />
        <img
          referrerpolicy="no-referrer"
          v-bind:src="post.secondaryPhotoURL"
          class="h-full w-full ml-3 sm:w-64 rounded-md mb-3"
        />
      </div>
      <div class="flex items-center font-bold mt-2 justify-center">
        <span> {{ postdate() }} </span>
        <span class="ml-3">Retakes - {{ post.retakeCounter }}</span>
      </div>
      <div class="flex flex-col">
        <span v-if="post.caption">
          <span class="font-bold">{{ post.user.username + ": " }} </span>
          {{ post.caption }}
        </span>
      </div>
      <div v-if="post.comment" class="flex flex-col">
        <div v-for="c in post.comment">
          <span class="font-bold"> {{ c.userName + ": " }}</span>
          <span v-html="boldusername(c.text)"></span>
        </div>
      </div>

      <div class="flex flex-col mt-4 ml-[25%]">
        <div v-for="e in post.realMojis" class="flex items-center mb-2">
          <div class="relative">
            <img
              referrerpolicy="no-referrer"
              v-bind:src="e.uri"
              class="w-24 rounded-[50%]"
            />

            <span class="absolute top-[50px] left-[60px] text-5xl"
              >{{ e.emoji }}
            </span>
          </div>
          <span class="font-bold ml-4">{{ e.userName }}</span>
        </div>
      </div>
    </div>
  </div>
</template>
