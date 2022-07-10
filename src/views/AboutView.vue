<template>
<!--{{rows}}-->
<br/>
<div class="row">
  <div class="col-10">
    <template v-for="(row, rowIndex) in rows" :key="row.id">
      <div class="container">
        <div class="row shadow-row"> 
          <div class="col" v-for="index in 12" :key="index" @drop="drop($event, rowIndex, index)" @dragover="allowDrop($event)">
            <div>{{index}}</div>	
          </div>
        </div>
        <div class="row align-items-start mark-row grabbable">
          <div :class="col.cs" v-for="(col, colIndex) in row.cols" :key="col.id" draggable="true" @dragstart="dragStart($event, col)">
            <div>{{col.content}}<!--{{col.cs}} S:{{col.start}} E:{{col.end}}--></div>		
          </div>
        </div>
      </div>
    </template>
  </div>
  <div class="col-2">

    <div class="card">
      <div class="card-header">
        Module
      </div>
      <ul class="list-group list-group-flush">
        <li class="list-group-item" draggable="true" @dragstart="dragNew($event, 'Text')">Text</li>
        <li class="list-group-item" draggable="true" @dragstart="dragNew($event, 'Bild')">Bild</li>
        <li class="list-group-item" draggable="true" @dragstart="dragNew($event, 'Video')">Video</li>
      </ul>
    </div>

  </div>
</div>
</template>

<script>

export default {
  components: {
  },
  data() {
    return {
      rowRepo: [],
      forceRecomputeDealListsCounter: 0
    };
  },
  computed: {
    rows: function() {
        this.forceRecomputeDealListsCounter;
		    let rows = this.rowRepo.getRows();

        //calculate bootstrap classes
        rows.forEach(row => {
          let lastEnd = 0;
          let cols = row.cols;
          cols.sort((a, b) => a.start - b.start);

          cols.forEach(col => {
            let cs = "ID:"+col.id+" ";
            if(col.start==1){
              let width = col.end-col.start+1;
              cs += "1) col-"+width.toString();
            }else if(col.start>1 && lastEnd<col.start){
              let width = col.end-col.start+1;
              cs += "2) offset-"+(col.start-lastEnd-1)+" col-"+width.toString();
            }else if(col.start>1){
              let width = col.end-col.start+1;
              cs += "3) offset-"+(lastEnd-1)+" col-"+width.toString();
            }else{
              let width = col.end-col.start+1;
              cs += "4) col-"+width.toString();
            }
            lastEnd = col.end;
            col.cs = cs+" lastEnd-"+lastEnd;
          });
        });

        console.log(rows);
        return Object.assign({}, rows);
    },
  },
  methods: {
    dragNew: function(event, type) {
      event.dataTransfer.setData("type", type);
      console.log(type);
    },
    dragStart: function(event, col) {
      event.dataTransfer.setData("col.id", col.id);
      console.log(col.id);
    },
    allowDrop: function(event) {
      event.preventDefault();
    },
    drop: function(event, rowId, start) {
      event.preventDefault();

      var type = event.dataTransfer.getData("type");
      if(type){
        console.log("newCol",type, rowId);
        this.rowRepo.newCol(type, rowId, start);
      }

      var colId = event.dataTransfer.getData("col.id");
      if(colId){
        console.log("moveCol",colId, rowId);
        this.rowRepo.moveCol(colId, rowId, start);
      }

      this.forceRecomputeDealListsCounter++;
    }
  },
  watch: {
  },
  created: function() {
    this.rowRepo = new RowRepo();
  },
  mounted: function() {
  },
  updated: function() {
  } 
};

class RowRepo {
		constructor() {
			this.rows = [
        {
          id:1,
          cols: [
            {id: 3, start:1, end:2, content:'Text', cs:''},
            {id: 4, start:3, end:5, content:'Text', cs:''},
            {id: 5, start:6, end:12, content:'Video', cs:''}
          ]
        },
        {
          id:2,
          cols: [
            {id: 0, start:2, end:3, content:'Bild', cs:''},
          ]
        },
        {
          id:3,
          cols: [
            {id: 6, start:4, end:5, content:'Video', cs:''},
          ]
        },
      ];
		}

		getRows() {
			let arrayCopy = this.rows.map(i => Object.assign({}, i));
			return arrayCopy;
		}

		getRowById(id) {
			let row = this.rows.find((i) => i.id == id);
			return Object.assign({}, row);
		}

		getRowsByProject(project) {
			let rows = this.rows.filter((i) => i.project == project);
			return Object.assign({}, rows);
		}

		getRowByLeadStatus(searchValue) {
			return this.rows.filter(row => {
				if (row.lead_status != null)
					return row.lead_status.toLowerCase().indexOf(searchValue.toLowerCase()) > -1
			})
		}

		updateRow(id, data) {
			let toChange = this.rows.findIndex(i => i.id == id);
			this.rows[toChange] = Object.assign(this.rows[toChange], data);
		}

		deleteById(id) {
			let toDelete = this.rows.findIndex(i => i.id == id);
			this.rows.splice(toDelete, 1);
		}

		createEmpty() {
			let emptyRows = {};
			return emptyRows;
		}

		add(row) {
			//append
			//this.rows.push();
			//prepend
			this.rows = [row, ...this.rows];
		}

    findRowForColId(colId){
      let foundIndex = -1;
      this.rows.forEach((row, index) => {
        let col = row.cols.find((i) => i.id == colId);
        if(col){
          console.log("found", index);
          foundIndex = index;
        }else{
          console.log("not found");
        }
      });
      return foundIndex;
    }

    getMaxId(){
      const ids = this.rows.map(row => {
        return row.id;
      });
      return Math.max(...ids);
    }

	  newCol(type, rowId, start) { 
      //find max id
      let newId = this.getMaxId();

      //create col
			let col = {id: newId, start:start, end:start+1, content:type, cs:''};

      //add col
			this.rows[rowId].cols = [col, ...this.rows[rowId].cols];
		}

    

	  moveCol(colId, rowId, start) { 
      console.log("before",this.rows);
      let prevRowIndex = this.findRowForColId(colId);
      
      console.log("prevRowIndex",prevRowIndex);
      if(prevRowIndex==-1) return;

      //presave col
			let col = this.rows[prevRowIndex].cols.find(i => i.id == colId);

      //change start
      let width = col.end-col.start;
      col.start = start;
      col.end = start+width;
      
      //delete col
			let toDelete = this.rows[prevRowIndex].cols.findIndex(i => i.id == colId);
			this.rows[prevRowIndex].cols.splice(toDelete, 1);

      //add col
			this.rows[rowId].cols = [col, ...this.rows[rowId].cols];
      
      console.log("after",this.rows);

		}
	}

</script>

<style>
.container{
  margin-bottom: 4rem;
}
.mark-row > div{
  min-height: 4rem;
}
.mark-row > div > div{
  padding:10px;
  border:1px solid #999;
  min-height: 4rem;
  background-color: #999;
  opacity: 0.9;
}

.shadow-row{
  top:4rem;
}

.shadow-row > div > div{
  border:1px solid #333;
  min-height: 6rem;
}

</style>
