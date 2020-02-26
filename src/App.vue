<template>
  <div id='app'>
    <p> 都道府県 </p>
    <ul>
      <li v-for="(pref, index) in prefs" :key="index">
        <label>
          <input @change="pref_check(pref)" type="checkbox" class="option"> {{pref.name}}
        </label>
      </li>
    </ul>
    <MainChart v-if="loaded" :data="this.graph_data" :options="this.chartOption"></MainChart>
    <form @submit.prevent="writeData" >
  </form>
  </div>


</template>


<script>

  import MainChart from '@/components/LineChart.js';
  import 'chartjs-plugin-colorschemes';

  export default{
  data() {
    return{
      graph_data: "",
      pop_list: [],
      prefs: [],
      results_year: [],
      results_value: [],
      loaded: false,
    }
  },
  mounted: function() {
    this.axios.get("https://opendata.resas-portal.go.jp/api/v1/prefectures",
      {headers: {'X-API-KEY': 'y6IIhvNvOgoMXxUmPgL35J29NlpkRweZZohCqtp0'}, data: {}})
    .then(response => {
      response.data.result.forEach((i) => {
        var item = {
          code: i.prefCode,
          name: i.prefName,
          checked: false
        };
        this.prefs.push(item);
      })
    })
  },
  methods : {
    writeData: function(val){
      this.loaded=false;
      this.axios.get("https://opendata.resas-portal.go.jp/api/v1/population/sum/estimate?prefCode="+val.code,
        {headers: {'X-API-KEY': 'y6IIhvNvOgoMXxUmPgL35J29NlpkRweZZohCqtp0'}, data: {}})
      .then(response => {
        response.data.result.data[0].data.forEach((i) => {
          this.results_year.push(i.year);
          this.results_value.push(i.value);
        }),

        this.pop_list.push({
          label: val.name,
          pointRadius: 5,
          fill: false,
          data: this.results_value
        });

        this.graph_data={
          labels: this.results_year,
          datasets: this.pop_list
        }
        this.loaded=true;
        this.results_year = [];
        this.results_value = [];
      })
    },
    deleteData: function(val){
      this.loaded = false;
      this.pop_list.some((i, index) => {
        if(val.name === i.label){
          this.pop_list.splice(index, 1);
          return true;
        }
      })

      this.graph_data.datasets = this.pop_list;
      this.loaded = true;

    },
    pref_check: function(val) {
      if(!val.checked){
        val.checked = true;
        this.writeData(val);
      }else{
        val.checked = false;
        this.deleteData(val);
      }
    }
  },
  components: {
    'MainChart' : MainChart
  },
  computed: {
    chartOption(){
      return{
        plugins: {
          colorschemes: {
            scheme: 'tableau.Tableau20'
          }
        },
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          xAxes: [{
            scaleLabel: {
               display: true,
              labelString	: '年度'
            }
          }],
          yAxes: [{
            scaleLabel: {
               display: true,
              labelString	: '人口数'
            }
          }]
        }
      }
    }
  }
};
</script>

<style>
body{
  background-color: #ebeced;
}
#app{
  font-family: "メイリオ";
	width: 90%;
	margin: 2em auto;
	text-align: left;
}

#app p {
    font-size: 25px;
    margin: 0px;
}

#app ul {
	list-style: none;
}

#app label {
  font-family: "メイリオ";
  height: 10px;
  margin-bottom: 15px;
	position: relative;
	margin: 0.5rem;
	cursor: pointer;
  display: flex;
  float: left;
  width: 110px;
}
#app .option {
	position: relative;
	margin: 0 1rem 0 0;
	cursor: pointer;
}
#app .option:before {
	position: absolute;
	z-index: 1;
	top: 0.125rem;
	left: 0.1875rem;
	width: 0.75rem;
	height: 0.375rem;
	content: '';
	-webkit-transition: -webkit-transform 0.4s cubic-bezier(0.45, 1.8, 0.5, 0.75);
	transition: transform 0.4s cubic-bezier(0.45, 1.8, 0.5, 0.75);
	-webkit-transform: rotate(-45deg) scale(0, 0);
	transform: rotate(-45deg) scale(0, 0);
	border: 2px solid #ff6a5c;
	border-top-style: none;
	border-right-style: none;
}
#app .option:checked:before {
	-webkit-transform: rotate(-45deg) scale(1, 1);
	transform: rotate(-45deg) scale(1, 1);
}
#app .option:after {
	position: absolute;
	top: -0.125rem;
	left: 0;
	width: 1rem;
	height: 1rem;
	content: '';
	cursor: pointer;
	border: 2px solid #847072;
	background: #ffffff;
}

#app .chartjs-size-monitor{
  position: relative;
  max-height: 300px;


}
#line-chart{
 max-height:300px;

}
</style>
