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
        <h4>Vaccine:</h4>
        <input
          type="radio"
          v-model="vaccine"
          value="COVISHIELD"
          id="COVISHIELD"
        />
        <label for="COVISHIELD">Covishield</label>
        <input type="radio" v-model="vaccine" value="COVAXIN" id="COVAXIN" />
        <label for="COVAXIN">Covaxin</label>
      </div>
      <div class="radios">
        <h4>Age:</h4>
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
      <tbody v-if="availableCenters.length > 0">
        <tr v-for="center in availableCenters" :key="center.center_id">
          <td>
            {{ center.name }}
            {{ center.address }} <br />
            {{ center.pincode }}
          </td>
          <span v-for="session in center.sessions" :key="session.session_id">
            <span
              v-if="
                session.available_capacity > 0 && session.vaccine === vaccine
              "
            >
              <td>
                {{ session.date }}<br />
                {{ session.available_capacity }}<br />
                {{ session.vaccine }}
              </td>
            </span>
          </span>
        </tr>
      </tbody>
      <tbody v-else>
        <tr>
          <td><b>Not Available</b></td>
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
      vaccine: "COVAXIN",
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
            session.vaccine === this.vaccine
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
.hello {
  max-width: 80%;
  min-height: 100vh;
  margin: 0 auto;
  background: white;
}
.options {
  margin-top: 50px;
  margin-left: 20px;
  margin-right: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
@media only screen and (max-width: 600px) {
  .options {
    flex-direction: column;
  }
}
select {
  margin: 5px;
}

label {
  margin: 0 5px;
}
table {
  margin: 2em auto;
}
table,
th,
td {
  border: 1px solid #ddd;
  border-collapse: collapse;
}
th,
td {
  max-width: 400px;
  padding: 5px;
}
th {
  background: #4caf50;
  color: #fff;
}
tr {
  text-align: left;
}
tr:nth-child(even) {
  background-color: #f2f2f2;
}
h4 {
  padding: 2px;
}
</style>
