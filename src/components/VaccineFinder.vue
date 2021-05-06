<template>
  <div class="hello">
    <h1>Find Vaccines for 18-45 or 45+ candidates</h1>
    <div class="options">
      <select @change="onChange" v-model="selectedState" id="selectedState">
        <option value="0">Select State</option>
        <option
          v-for="state in states"
          :key="state.state_id"
          :value="state.state_id"
          >{{ state.state_name }}</option
        >
      </select>
      <select v-model="selectedDistrict" id="selectedDistrict">
        <option value="all">All Districts</option>
        <option
          v-for="dist in districts"
          :key="dist.district_id"
          :value="dist.district_id"
          >{{ dist.district_name }}</option
        >
      </select>
      <div class="radios">
        <span>Vaccine:</span>
        <input
          type="radio"
          v-model="vaccine"
          value="covishield"
          id="covishield"
        />
        <label for="covishield">Covishield</label>
        <input type="radio" v-model="vaccine" value="covaxin" id="covaxin" />
        <label for="covaxin">Covaxin</label>
      </div>
      <div class="radios">
        <span>Age:</span>
        <input type="radio" v-model.number="age" value="18" id="adult" />
        <label for="adult">18-45</label>
        <input type="radio" v-model.number="age" value="45" id="old" />
        <label for="old">45+</label>
      </div>
    </div>
    <table>
      <thead>
        <th>Center</th>
        <th>Availability</th>
      </thead>
      <tbody>
        <tr v-for="center in availableCenters" :key="center.center_id">
          <td>
            {{ center.name }}
            {{ center.address }} <br />
            {{ center.pincode }}
          </td>
          <span v-for="session in center.sessions" :key="session.session_id">
            <span v-if="session.available_capacity > 0">
              <td>
                {{ session.date }}<br />
                {{ session.available_capacity }}<br />
                {{ session.vaccine }}
              </td>
            </span>
          </span>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import moment from "moment";
export default {
  name: "VaccineFinder",
  props: {
    url: String,
  },
  data() {
    return {
      states: [],
      selectedState: 0,
      districts: [],
      centers: [],
      availableCenters: [],
      vaccine: "covaxin",
      age: 18,
      selectedDistrict: "all",
    };
  },
  created() {
    this.fetchStates();
  },
  methods: {
    async fetchStates() {
      const data = await fetch(`${this.url}/admin/location/states`);
      const statesData = await data.json();
      this.states = statesData.states;
    },

    async fetchDistricts(state_id) {
      const distData = await fetch(
        `${this.url}/admin/location/districts/${state_id}`
      );
      const distJson = await distData.json();
      this.districts = distJson.districts;
    },

    async getData(id) {
      let date = moment().format("DD-MM-YYYY");
      const vaccineData = await fetch(
        `${this.url}/appointment/sessions/public/calendarByDistrict?district_id=${id}&date=${date}`
      );
      const vaccineJson = await vaccineData.json();
      return vaccineJson.centers;
    },

    async fetchVaccineData(districtIds) {
      const centerList = await Promise.all(districtIds.map(this.getData));
      const array = centerList.reduce((arr, row) => arr.concat(row), []);
      this.centers = array;
    },

    getOnlyAvailableSlots() {
      const availableCenters = this.centers.filter((center) => {
        return center.sessions.some(
          (session) =>
            session.available_capacity > 0 &&
            session.min_age_limit === this.age &&
            session.vaccine.toLowerCase() === this.vaccine
        );
      });
      this.availableCenters = availableCenters;
    },

    onChange() {
      this.fetchDistricts(this.selectedState);
    },
  },
  watch: {
    districts: function() {
      const districtIds = this.districts.map(
        (district) => district.district_id
      );
      this.fetchVaccineData(districtIds);
    },
    centers: function() {
      this.getOnlyAvailableSlots();
    },
    vaccine: function() {
      this.getOnlyAvailableSlots();
    },
    age: function() {
      this.getOnlyAvailableSlots();
    },
    selectedDistrict: function() {
      let distArray = JSON.parse(JSON.stringify(this.districts));
      if (this.selectedDistrict !== "all") {
        const dist = distArray.filter(
          (district) => district.district_id == this.selectedDistrict
        );
        const dist_ids = dist.map((d) => d.district_id);
        this.fetchVaccineData(dist_ids);
      } else {
        this.fetchDistricts(this.selectedState);
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.options {
  max-width: 80%;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
}
select {
  padding: 5px;
  margin: 5px;
}
table {
  margin: 2em auto;
}
table,
th,
td {
  border: 1px solid black;
}
th,
td {
  max-width: 400px;
  padding: 5px;
}
tr {
  text-align: left;
}
</style>
