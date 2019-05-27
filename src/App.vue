<template>
  <div id="app">
    <div v-if="!initFinished">
      <span>Downloading and analyzing data</span>
      <div class="spinner">
        <div class="bounce1"></div>
        <div class="bounce2"></div>
        <div class="bounce3"></div>
      </div>
    </div>
    <div v-if="initFinished">
      <div>
        <button class="button" v-bind:class="buttonClassObj('all')" v-on:click="buttonClick('all')">
          All Data
        </button>
        <button class="button" v-bind:class="buttonClassObj('keyword')" v-on:click="buttonClick('keyword')">
          Queries
        </button>
        <button class="button" v-bind:class="buttonClassObj('url')" v-on:click="buttonClick('url')">
          Pages
        </button>
        <button class="button" v-bind:class="buttonClassObj('country')" v-on:click="buttonClick('country')">
          Countries
        </button>
        <button class="button" v-bind:class="buttonClassObj('device')" v-on:click="buttonClick('device')">
          Devices
        </button>
      </div>
      <GridComponentServerSide ref="gridComponent" v-on:init="gridInitFinished" :row-data="rowData" :column-defs="columnDefs"></GridComponentServerSide>
    </div>
  </div>
</template>

<script>
/*import GridComponent from './components/GridComponent.vue';
import GridComponentInfiniteScroll from './components/GridComponentInfiniteScroll';*/
import GridComponentServerSide from './components/GridComponentServerSide'

export default {
  name: 'app',
  components: {
    GridComponentServerSide
  },
  data: function () {
    return {
      initFinished: false,
      rowData: [],
      columnDefs: [],
      keywordGrouped: [],
      pagesGrouped: [],
      countryGrouped: [],
      deviceGrouped: [],
      rowDataAll: [],
      commonDefs: [
        {headerName: 'Data', children:[
            {headerName: 'Total Clicks', field: 'clicks', sortable: true, filter:false},
            {headerName: 'Total Impressions', field: 'impressions', sortable: true, filter: false},
            {headerName: 'CTR', field: 'ctr', sortable: true, filter: false, cellStyle: function (params) {
                let ctrValue = params.value;
                let retObj = {};
                if(ctrValue < 0.3333) {
                  retObj = {
                    backgroundColor: '#c42c38'
                  }
                } else if (ctrValue >= 0.3333 && ctrValue < 0.6666) {
                  retObj = {
                    backgroundColor: '#c4a415'
                  }
                } else {
                  retObj = {
                    backgroundColor: '#0cc40e'
                  }
                }
                return retObj;
              }}
          ]
        }
      ],
      clickedButton: null,
      infiniteButtonClicked: false
    };
  },
  beforeMount() {
    this.httpGetData();
  },
  methods: {
    httpGetData: function() {
      /*
      let username = 'interview_1';
      let password = 'int_candidate12';
      this.$http.get('https://cors-anywhere.herokuapp.com/http://35.243.142.103:5000/get_data', {
        headers: {
          'Authorization': 'Basic ' +  btoa(username + ":" + password)
        }
      })
       */
      // Getting the copy of data from localhost as the server is down now.
      this.$http.get('/data/data.json').then((response) => {
        this.analyzeData(response.body);
        this.initFinished = true;
      });
    },
    analyzeData: function (dataJson) {
      for(let index in dataJson) {
        this.checkExists(this.keywordGrouped, dataJson[index], 'keyword');
        this.checkExists(this.pagesGrouped, dataJson[index], 'url');
        this.checkExists(this.countryGrouped, dataJson[index], 'country');
        this.checkExists(this.deviceGrouped, dataJson[index], 'device');
        let newObj = {...dataJson[index]};
        newObj['ctr'] = (newObj.clicks / newObj.impressions).toFixed(4);
        this.rowDataAll.push(newObj);
      }
    },
    checkExists: function(checkArray, checkObj, checkStr) {
      let found = checkArray.find(x => x[checkStr] === checkObj[checkStr]);
      if(found) {
        found.clicks += checkObj.clicks;
        found.impressions += checkObj.impressions;
        found.ctr = (found.clicks / found.impressions).toFixed(4);
      } else {
        let newObj = {
          clicks: checkObj.clicks,
          impressions: checkObj.impressions,
          ctr: (checkObj.clicks / checkObj.impressions).toFixed(4)
        };
        newObj[checkStr] = checkObj[checkStr];
        checkArray.push(newObj);
      }
    },
    buttonClick: function (clickedButton) {
      let columnDefs = [];
      switch (clickedButton) {
        case 'keyword':
          this.rowData = this.keywordGrouped;
          columnDefs = [{headerName:'Grouped By', children:[{headerName: 'Queries', field:'keyword', pinned: 'left',
              filterParams: {values: this.getFilterData(this.keywordGrouped, 'keyword'), newRowsAction: "keep"}}]}, ...this.commonDefs];
          break;
        case 'url':
          this.rowData = this.pagesGrouped;
          columnDefs = [{headerName:'Grouped By', children:[{headerName: 'Pages', field:'url', pinned: 'left',
              filterParams: {values: this.getFilterData(this.pagesGrouped, 'url'), newRowsAction: "keep"}}]}, ...this.commonDefs];
          break;
        case 'country':
          this.rowData = this.countryGrouped;
          columnDefs = [{headerName:'Grouped By', children:[{headerName: 'Countries', field:'country', pinned: 'left',
              filterParams: {values: this.getFilterData(this.countryGrouped, 'country'), newRowsAction: "keep"}}]}, ...this.commonDefs];
          break;
        case 'device':
          this.rowData = this.deviceGrouped;
          columnDefs = [{headerName:'Grouped By', children:[{headerName: 'Devices', field:'device', pinned: 'left',
              filterParams: {values: this.getFilterData(this.deviceGrouped, 'device'), newRowsAction: "keep"},
            cellStyle: function(params){
              let device = params.value;
              let retObj = {};
              if(device === 'DESKTOP') {
                retObj = {
                  backgroundColor: '#FFD765'
                };
              } else if(device === 'MOBILE') {
                retObj = {
                  backgroundColor: '#CDAEFF'
                };
              } else if(device ==='TABLET') {
                retObj = {
                  backgroundColor: '#FB9BE3'
                }
              }
              return retObj;
            }}]}, ...this.commonDefs];
          break;
        case 'all':
          this.rowData = this.rowDataAll;
          columnDefs = [
            {headerName: 'Date', field: 'date', enableValue: false, filterParams: {values: this.getFilterData(this.rowDataAll, 'date'), newRowsAction: "keep"},
              comparator: dateComparator, sort: 'desc'},
            {headerName: 'Keyword', field: 'keyword', filterParams: {values: this.getFilterData(this.rowDataAll, 'keyword'), newRowsAction: "keep"}},
            {headerName: 'Country', field: 'country', filterParams: {values: this.getFilterData(this.rowDataAll, 'country'), newRowsAction: "keep"}},
            {headerName: 'Device', field: 'device', filterParams: {values: this.getFilterData(this.rowDataAll, 'device'), newRowsAction: "keep"},
              cellStyle: function(params){
                let device = params.value;
                let retObj = {};
                if(device === 'DESKTOP') {
                  retObj = {
                    backgroundColor: '#FFD765'
                  };
                } else if(device === 'MOBILE') {
                  retObj = {
                    backgroundColor: '#CDAEFF'
                  };
                } else if(device ==='TABLET') {
                  retObj = {
                    backgroundColor: '#FB9BE3'
                  }
                }
                return retObj;
              }},
            {headerName: 'URL', field: 'url', filterParams: {values: this.getFilterData(this.rowDataAll, 'url'), newRowsAction: "keep"}},
            {headerName: 'Position', field: 'position', filterParams: {values: this.getFilterData(this.rowDataAll, 'position'), newRowsAction: "keep"}},
            {headerName: 'Clicks', field: 'clicks', filter: false},
            {headerName: 'Impressions', field: 'impressions', filter: false},
            {headerName: 'CTR', field: 'ctr', sortable: true, filter: false, cellStyle: function (params) {
                let ctrValue = params.value;
                let retObj = {};
                if(ctrValue < 0.3333) {
                  retObj = {
                    backgroundColor: '#c42c38'
                  }
                } else if (ctrValue >= 0.3333 && ctrValue < 0.6666) {
                  retObj = {
                    backgroundColor: '#c4a415'
                  }
                } else {
                  retObj = {
                    backgroundColor: '#0cc40e'
                  }
                }
                return retObj;
              }}
          ];
          break;
        default:
          break;
      }
      this.columnDefs = columnDefs;
      this.clickedButton = clickedButton;
    },
    gridInitFinished: function () {
      this.$refs.gridComponent.gridApi.sizeColumnsToFit();
    },
    buttonClassObj: function (button) {
      return {
        selectedButton: this.clickedButton === button
      }
    },
    getFilterData: function (source, str) {
      let length = source.length, result = [], seen = new Set();
      for (let index = 0; index < length; index++) {
        let value = source[index][str];
        if (seen.has(value)) continue;
        seen.add(value);
        result.push(value);
      }
      return result;
    }
  }
}
function dateComparator(date1, date2) {
  let date1Number = monthToComparableNumber(date1);
  let date2Number = monthToComparableNumber(date2);
  if (date1Number === null && date2Number === null) {
    return 0;
  }
  if (date1Number === null) {
    return -1;
  }
  if (date2Number === null) {
    return 1;
  }
  return date1Number - date2Number;
}

function monthToComparableNumber(date) {
  if (date === undefined || date === null || date.length !== 10) {
    return null;
  }
  let yearNumber = date.substring(0, 4);
  let monthNumber = date.substring(5, 7);
  let dayNumber = date.substring(8, 10);
  let result = yearNumber * 10000 + monthNumber * 100 + dayNumber;
  return result;
}

</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.button {
  background-color: #555555;
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}
.selectedButton {
  background-color: #008CBA;
}

.spinner {
  margin: 100px auto 0;
  width: 70px;
  text-align: center;
}

.spinner > div {
  width: 18px;
  height: 18px;
  background-color: #333;

  border-radius: 100%;
  display: inline-block;
  -webkit-animation: sk-bouncedelay 1.4s infinite ease-in-out both;
  animation: sk-bouncedelay 1.4s infinite ease-in-out both;
}

.spinner .bounce1 {
  -webkit-animation-delay: -0.32s;
  animation-delay: -0.32s;
}

.spinner .bounce2 {
  -webkit-animation-delay: -0.16s;
  animation-delay: -0.16s;
}

@-webkit-keyframes sk-bouncedelay {
  0%, 80%, 100% { -webkit-transform: scale(0) }
  40% { -webkit-transform: scale(1.0) }
}

@keyframes sk-bouncedelay {
  0%, 80%, 100% {
    -webkit-transform: scale(0);
    transform: scale(0);
  } 40% {
      -webkit-transform: scale(1.0);
      transform: scale(1.0);
    }
}
</style>
