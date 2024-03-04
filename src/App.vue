<template>
  <div class="testbox">
    <form @submit.prevent="handleSubmit">
      <div class="banner">
        
        
      </div>
      <br>
      <br>
      <div class="container">
        <label for="building">Bâtiment</label>
        <select v-model="selectedBuilding" @change="handleBuildingChange">
          <option v-for="building in buildings" :key="building.staticId" :value="building.name">{{ building.name }}
          </option>
        </select>

        <label for="floor">Étage</label>
        <select v-model="selectedFloor">
          <option v-for="floor in floors" :key="floor.staticId" :value="floor.name">{{ floor.name }}</option>
        </select>
      </div>
      <br>

      <div class="btn-block">
        <button @change="handleFloorChange">Submit</button>
      </div>


    </form>
  </div>

  <div class="testbox">
    <form @submit="handleSubmit">
      <div class="cont">
        <div class="column">
          <div class="item">


            <h2>les informations du chambre</h2>

            <ul class="grid-container">
              <li v-for="room in roomData" :key="room.staticId">
                <p><strong>{{ room.name }}</strong> : {{ room.currentValue }}</p>
              </li>
            </ul>
          </div>
        </div>
        <div class="column">
          <br>
        </div>
        <div class="column">


          <h2>Statique global du batiment {{ selectedBuilding }}</h2>
          <div class = "grid-container">
            <p>Nombre de chambres totales : {{ nbrooom }}</p>
            <p>Nombre d'étages : {{ nbFloor }}</p>
            <p>Nombre de chambres occupées : {{ roomOccupied }}</p>
            <p>Nombre de chambres non occupées : {{ roomNonOccupied }}</p>
            <p>Nombre de chambres non définies : {{ roomUndefined }}</p>
          </div>
          <br>
          <div>
            <canvas id="myChart" width="400" height="400"></canvas>
          </div>
          
          
          

        </div>
      </div>
    </form>
  </div>


  <div class="testbox">
    <form @submit="handleSubmit">
      <div class="cont">
        <div class="column">


          <h2>Statut des chambres</h2>
          <br>
          
          <table>
            <thead>
              <tr>
                <th>Nom de la chambre</th>
                <th>État</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="room in rooms" :key="room.staticId">
                <td>{{ room.name }}</td>
                <td :class="{'occupied': room.state === 'Occupé', 'not-occupied': room.state === 'Non occupé'}">{{ room.state }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="column">
          <br>
        </div>
        

        <div class="column">
          
          <h2>Statique global de l'étage {{ selectedFloor }}</h2>
          <br>
          <div class = "grid-container">
            <p>Nombre de chambres : {{this.rooms.filter(room => room.state === 'Occupé').length +this.rooms.filter(room => room.state === 'Non Occupé').length + this.rooms.filter(room => room.state === 'Non défini').length  }}</p>
            <p>Nombre de chambres occupées : {{ this.rooms.filter(room => room.state === 'Occupé').length}}</p>
            <p>Nombre de chambres non occupées : {{this.rooms.filter(room => room.state === 'Non occupé').length}}</p>
            <p>Nombre de chambres non définies : {{ this.rooms.filter(room => room.state === 'Non défini').length  }}</p>
          </div>
          <div>
            <canvas id="myChartE" width="400" height="400"></canvas>
          </div>
        </div>
      </div>

    </form>
  </div>




</template>

<script>
import Chart from 'chart.js/auto';



export default {

  data() {
    return {
      buildings: [], // Liste des bâtiments
      selectedBuilding: '', // Bâtiment sélectionné
      floors: [], // Liste des étages
      selectedFloor: '', // Étage sélectionné
      rooms: [],// Liste des chambres
      nbFloor: 0,// Nombre d'étages
      nbrooom: 0, // Nombre de chambres
      roomData: [], // Données de chaque chambre
      roomOccupied: 0, // Nombre de chambres occupées
      roomNonOccupied: 0, // Nombre de chambres non occupées
      roomUndefined: 0, // Nombre de chambres non définies
      tableData: {},
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          display: true,
        },
      },

    };
  },


  mounted() {
    // Charger les données du fichier JSON
    this.loadGeographicData();
    this.loadRoomData();




  },
  methods: {
    // Charger les données géographiques depuis le fichier JSON
    loadGeographicData() {
      const geographicData = require('./gegraphicContext_space.json');
      this.buildings = geographicData.children;
      const roomData = require('./ROOM_control_endpoin_list.json');
      this.roomData = roomData;

      let etages = this.buildings[0].children

      console.log(etages[0].name);

      this.tableData = {
        [etages[0].name]: etages[0].children.filter(child => child.type === 'geographicRoom').map(room => {
          return { ...room, state: this.generateRandomState() };
        }),
        [etages[1].name]: etages[1].children.filter(child => child.type === 'geographicRoom').map(room => {
          return { ...room, state: this.generateRandomState() };
        }),
        [etages[2].name]: etages[2].children.filter(child => child.type === 'geographicRoom').map(room => {
          return { ...room, state: this.generateRandomState() };
        })
      };

      for (let i = 0; i < etages.length; i++) {
        this.roomOccupied +=   this.tableData[etages[i].name].filter(room => room.state === 'Occupé').length;
        this.roomNonOccupied +=  this.tableData[etages[i].name].filter(room => room.state === 'Non occupé').length;
        this.roomUndefined += this.tableData[etages[i].name].filter(room => room.state === 'Non défini').length;
      }

      this.nbrooom = this.roomOccupied + this.roomNonOccupied + this.roomUndefined;
      this.nbFloor = etages.length;




    },

    handleBuildingChange() {

      if (!this.selectedBuilding) {
        return;
      } else {
        const selectedBuilding = this.buildings.find(building => building.name === this.selectedBuilding);
        this.floors = selectedBuilding.children.filter(child => child.type === 'geographicFloor');
      }
    },

    handleFloorChange() {

      this.rooms = this.tableData[this.selectedFloor];
    },


    handleSubmit() {

      this.handleFloorChange();
      this.renderChart();
      console.log(this.roomOccupied);
      console.log(this.roomNonOccupied);
      console.log(this.roomUndefined);
    },
    loadRoomData() {
      const roomData = require('./ROOM_control_endpoin_list.json');
      this.roomData = roomData[0].endpoints;
    },

    generateRandomState() {
      const states = ['Occupé', 'Non occupé', 'Non défini'];
      return states[Math.floor(Math.random() * states.length)];
    },


    renderChart() {

      if (this.myChart) {
        this.myChart.destroy();
      }
      if (this.myChartE) {
        this.myChartE.destroy();
      }
      var ctx = document.getElementById('myChart').getContext('2d');

      const nbOcc = this.rooms.filter(room => room.state === 'Occupé').length;
      const nbNonOcc = this.rooms.filter(room => room.state === 'Non occupé').length;
      const nbUndefined = this.rooms.filter(room => room.state === 'Non défini').length;
      const total = nbOcc + nbNonOcc + nbUndefined;

      this.myChart = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: ['Occupé', 'Non occupé', 'en attente'],
          datasets: [{
            data: [(this.roomNonOccupied / this.nbrooom) * 100, (this.roomNonOccupied / this.nbrooom) * 100, (this.roomUndefined / this.nbrooom) * 100], // Mettez vos pourcentages ici
            backgroundColor: [
              'rgb(255, 99, 132)',
              'rgb(54, 162, 235)',
              'rgb(255, 205, 86)'
            ],
            hoverOffset: 4
          }]
        },
        options: {
          responsive: false
        }
      });

      this.myChartE = new Chart(document.getElementById('myChartE').getContext('2d'), {
        type: 'doughnut',
        data: {
          labels: ['Occupé', 'Non occupé', 'en attente'],
          datasets: [{
            data: [(nbOcc / total) * 100, (nbNonOcc / total) * 100, (nbUndefined / total) * 100], 
            backgroundColor: [
              'rgb(255, 99, 132)', 
              'rgb(54, 162, 235)', 
              'rgb(255, 205, 86)' 
            ],
            hoverOffset: 4
          }]
        },
        options: {
          responsive: false
        }
      });
    },



  }




};
</script>

<style scoped>
html,
body {
  min-height: 100%;
}

body,
div,
form,
input,
textarea,
p {
  padding: 0;
  margin: 0;
  outline: none;
  font-family: Roboto, Arial, sans-serif;
  font-size: 14px;
  color: #666;
  line-height: 22px;
}

h1 {
  position: absolute;
  margin: 0;
  font-size: 32px;
  color: #fff;
  z-index: 2;
}

h3 {
  margin: 20px 0 10px;
  color: #050911;
}
.occupied {
  color: red;
}

.not-occupied {
  color: green;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-gap: 10px;
}

.testbox {
  display: flex;
  justify-content: center;
  align-items: center;
  height: inherit;
  padding: 20px;
}

form {
  width: 100%;
  padding: 20px;
  border-radius: 6px;
  background: #fff;
  box-shadow: 0 0 10px 0 #3e3304;
}

.banner {
  position: relative;
  height: 415px;
  background-image: url("spinalcom.png");
  background-size: contain;
  background-repeat: no-repeat; 
  background-position: center center; 
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}


.banner::after {
  content: "";
  background-color: rgba(0, 0, 0, 0.4);
  position: absolute;
  width: 100%;
  height: 100%;
}



input,
textarea {
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

input {
  width: calc(100% - 10px);
  padding: 5px;
}

textarea {
  width: calc(100% - 12px);
  padding: 5px;
}

.item:hover p,
input:hover::placeholder {
  color: #564508;
}

.item input:hover,
.item textarea:hover {
  border: 1px solid transparent;
  box-shadow: 0 0 6px 0 #cc0052;
  color: #cc0052;
}

.item {
  position: relative;
  margin: 10px 0;
}

.btn-block {
  margin-top: 10px;
  text-align: center;
}

.container {
  text-align: center;
  
  margin: 0 auto;
 
  max-width: 1000px;
  
}

.cont {
  display: grid;
  grid-template-columns: 1fr 0.5fr 2fr;

}

.colunm {
  flex: 30%;

}

#university {
  margin-right: 50px;
  /* Ajoute une marge à gauche */
}

#pays {
  margin-left: 20px;
  /* Ajoute une marge à gauche */
}

button {
  width: 150px;
  padding: 10px;
  border: none;
  border-radius: 5px;
  background: #6e5a09;
  font-size: 16px;
  color: #fff;
  cursor: pointer;
}

button:hover {
  background: #0f0107;
}

select {
  width: 15%;
  /* Utilise tout l'espace disponible dans la deuxième colonne */
  padding: 5px;
  border: 3px solid #e7e0e0;
  border-radius: 8px;
  box-sizing: border-box;
}

@media (min-width: 568px) {

  .name-item,
  .contact-item {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .contact-item .item {
    width: calc(50% - 8px);
  }

  .name-item input {
    width: calc(50% - 20px);
  }

  .contact-item input {
    width: calc(100% - 12px);
  }
  table {
  width: 100%;
  border-collapse: collapse;
}

table th, table td {
  padding: 8px;
  border: 1px solid #ccc;
}

table th {
  background-color: #f2f2f2;
}

table tbody tr:nth-child(even) {
  background-color: #f2f2f2;
}

table tbody tr:hover {
  background-color: #ddd;
}

}
</style>
