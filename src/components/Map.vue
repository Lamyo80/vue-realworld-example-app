<template>
  <div
    class="map-container"
    style="display: flex; justify-content: center; width: 100%"
  >
    <div id="map" style="width: 400px; height: 400px;"></div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { HERE_MAP_API_KEY } from "@/common/config";

export default {
  name: "RwvMap",
  props: {
    location: { type: String, required: true }
  },
  data() {
    return {
      geocode: {},
      error: null,
      platform: null
    };
  },
  computed: {
    isCurrentUser() {
      if (this.currentUser.username && this.comment.author.username) {
        return this.comment.author.username === this.currentUser.username;
      }
      return false;
    },
    ...mapGetters(["currentUser"])
  },
  mounted() {
    this.platform = new H.service.Platform({
      apikey: HERE_MAP_API_KEY
    });
    this.convertLocationToGeocode();
  },
  methods: {
    convertLocationToGeocode() {
      let geocoderService = this.platform.getSearchService();

      geocoderService.geocode(
        {
          q: "RENNES"
        },
        result => {
          result.items.forEach(item => {
            this.geocode = item;
          });

          this.loadMap();
        },
        error => {
          this.error = error;
        }
      );
    },
    loadMap() {
      var defaultLayers = this.platform.createDefaultLayers();

      var map = new H.Map(
        document.getElementById("map"),
        defaultLayers.vector.normal.map,
        {
          zoom: 10,
          center: {
            lat: this.geocode.position.lat,
            lng: this.geocode.position.lng
          }
        }
      );

      console.log(this.geocode.position);

      map.addObject(new H.map.Marker(this.geocode.position));
      // add a resize listener to make sure that the map occupies the whole container
      window.addEventListener("resize", () => map.getViewPort().resize());

      //Step 3: make the map interactive
      // MapEvents enables the event system
      // Behavior implements default interactions for pan/zoom (also on mobile touch environments)
      var behavior = new H.mapevents.Behavior(new H.mapevents.MapEvents(map));

      // Create the default UI components
      var ui = H.ui.UI.createDefault(map, defaultLayers);
    }
  }
};
</script>
