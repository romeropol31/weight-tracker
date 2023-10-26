<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'montserrat', sans-serif;
}

body {
	background-color: #eee;
}

main {
	padding: 1.5rem;
}

h1 {
	font-size: 2em;
	text-align: center;
	margin-bottom: 2rem;
}

h2 {
	margin-bottom: 1rem;
	color: #888;
	font-weight: 400;
}

.current {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	width: 200px;
	height: 200px;
	
	text-align: center;
	background-color: white;
	border-radius: 999px;
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	border: 5px solid hwb(330 41% 0%);
	
	margin: 0 auto 2rem;
}

.current span {
	display: block;
	font-size: 2em;
	font-weight: bold;
	margin-bottom: 0.5rem;
}

.current small {
	color: #888;
	font-style: italic;
}

form {
	display: flex;
	margin-bottom: 2rem;
	border: 1px solid #AAA;
	border-radius: 0.5rem;
	overflow: hidden;
	transition: 200ms linear;
}

form:focus-within,
form:hover {
	border-color: hotpink;
	border-width: 2px;
}

form input[type="number"] {
	appearance: none;
	outline: none;
	border: none;
	background-color: white;
	width: 100%;
	padding: 1rem 1.5rem;
	font-size: 1.25rem;
}

form input[type="submit"] {
	appearance: none;
	outline: none;
	border: none;
	cursor: pointer;
	background-color: hotpink;

	padding: 0.5rem 1rem;

	color: white;
	font-size: 1.25rem;
	font-weight: 700;
	transition: 200ms linear;
	border-left: 3px solid transparent;
}

form input[type="submit"]:hover {
	background-color: white;
	color: hotpink;
	border-left-color: hotpink;
}

.canvas-box {
	width: 100%;
	background-color: white;
	padding: 1rem;
	border-radius: 0.5rem;
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	margin-bottom: 2rem;
}

.weight-history ul {
	list-style: none;
	padding: 0;
	margin: 0;
}

.weight-history ul li {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0.5rem;
	cursor: pointer;
}

.weight-history ul li:nth-child(even) {
	background-color: #dfdfdf;
}

.weight-history ul li:hover {
	background-color: #f8f8f8;
}

.weight-history ul li:last-of-type {
	border-bottom: none;
}

.weight-history ul li span {
	display: block;
	font-size: 1.25rem;
	font-weight: 700;
	margin-right: 1rem;
}

.weight-history ul li small {
	color: #888;
	font-style: italic;
}
</style>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current Weight (kg)</small>
    </div>
	
	<div class="canvas-box">
	  <canvas ref="weightChartEl"></canvas>
	</div>

    <form @submit.prevent="addWeight">
      <input type="number" v-model="weightInput" step="0.1" />
      <input type="submit" value="Add weight" />
    </form>
	
	<div class="weight-history">
	  <h2>Weight History</h2>
	  <v-data-table :items="weights" :headers="headersTest" class="elevation-1">
		<template v-slot:item.weight="{item}">
			<v-text-field v-if="item.raw.id === editedItem.id" v-model="editedItem.weight" :hide-details="true" dense single-line :autofocus="true" @blur="saveItem"></v-text-field>
    	<span v-else>{{item.raw.weight}}</span>
		</template>
		<template v-slot:item.date="{item}">
    	{{new Date(item.raw.date).toLocaleDateString()}}
		</template>
		<template v-slot:item.actions="{item}">
			<div style="display:flex;flex-direction: row;">
				<v-icon color="green" class="mr-3" @click="editItem(item.raw)">
					mdi-pencil
				</v-icon>
				<v-icon color="red" @click="deleteItem(item.raw)">
					mdi-delete
				</v-icon>
			</div>
		</template>
	  </v-data-table>
	</div>

  </main>
</template>

<script setup>
import { ref,shallowRef,computed,watch,nextTick,onMounted } from 'vue';
import Chart from 'chart.js/auto';

const weights = ref([]);
const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(60.0);

const editedIndex = ref(-1);
const editedItem = ref({
	id:0,
	weight:'',
	date:'',
})

const headersTest = ref([
	{title:'Weight',key:'weight'},
	{title:'Date',key:'date'},
	{title:'',key:'actions',sortable:false,width:'5px'},
])
const currentWeight = computed(() => {
  return weights.value.sort((a,b) => b.date - a.date)[0] || { weight:0 };
})

const editItem = (item) => {
	editedIndex.value = weights.value.indexOf(item);
	editedItem.value = Object.assign({}, item);
}

const deleteItem = (item) => {
	const index = weights.value.indexOf(item);
	weights.value.splice(index, 1);
}
const saveItem = () => {
	if (editedIndex.value > -1) {
		Object.assign(weights.value[editedIndex.value], editedItem.value)
	}
	editedItem.value = Object.assign({}, {id:0,weight:'',date:''});
	editedIndex.value = -1;
}

const addWeight = () => {
  weights.value.push({
		id: weights.value.length + 1,
    weight:weightInput.value,
    date: new Date().getTime()
  });
}
onMounted(() => {
  let ls = (JSON.parse(localStorage.getItem("data")));
  if(ls != null && ls.weights.length)weights.value = ls.weights;
  weightChart.value = new Chart(weightChartEl.value.getContext('2d'),{
	type:'line',
	data:{
		labels:ls !=null && ls.weights.length ? weights.value.sort((a,b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()) : [],
		datasets:[
			{label:'weight',
			data:ls !=null && ls.weights.length ? weights.value.sort((a,b) => a.date - b.date).map(w => w.weight) : [],
			backgroundColor:'rgba(255,105,180,0.2)',
          	borderColor:'rgb(255,105,180)',
          	borderWidth:1,
          	fill:true}
		]
	},
	options:{
        responsive:true,
        maintainAspectRatio:false
      }
  })
})

watch(weights, newWeights => {
  const ws = [...newWeights];
  localStorage.setItem("data",JSON.stringify({weights:weights.value}));

  if(weightChart.value){
    weightChart.value.data.labels = ws.sort((a,b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()).slice(-7);
    weightChart.value.data.datasets[0].data = ws.sort((a,b) => a.date - b.date).map(w => w.weight).slice(-7);
    weightChart.value.update();

    return;
  }
},{deep:true})
 
</script>
