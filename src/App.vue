
<template>
  <div id="app">
    <div class="main">
      <div class="calendar-holder">
        <FullCalendar :config="config" :events="events" @event-selected="eventClick" @event-drop="eventDrop" @event-resize="eventResize"/>
      </div>
      <JsonCSV :data   = "events"  name    = "events.csv">
        <button style="height: 40px; font-size: 15px; width: 80px;  position: absolute; top: 60px; right: 400px;">Export</button>
      </JsonCSV>
      <input type="file" ref="file" style="display: none" @change="importCsv">
      <button style="height: 40px; font-size: 15px; width: 80px;  position: absolute; top: 60px; right: 500px;" @click="$refs.file.click()">Import</button>
      <button style="height: 40px; font-size: 15px; width: 80px;  position: absolute; top: 60px; right: 300px;" @click="showReportModal = true">Report</button>
      
      <div v-if="showModal" @close="showModal = false">
        <EventForm :event="currentEvent" :method="currentMethod" :list="projects" @addProject="addProject" @close="closeModal" @removeProject="removeProject"/>
      </div>

      <div v-if="showReportModal" @close="showReportModal = false">
        <DateChooser :start="startDate" :end="endDate" @date_set="dateChanged" @close="closeModal"/>
      </div>

      <b-container class="form-holder" id="external-events">
        <div id='external-events-listing'>
          <b-row v-for="item in projects" :key="item.id">
            <b-col cols="1"></b-col>
            <b-col cols="5">
            <div class="colorbox" :style="{ background: item.backgroundColor}"  @click="currentEvent = item ,currentMethod = 1, showModal = true"/>
            </b-col>
            <b-col cols="4">
              <div style="text-align: left; width: fit-content; padding: 1px; cursor: default;" :class="{selected: selected === item.id}" @click="currentEvent = item ,currentMethod = 1, showModal = true">
            {{ item.title }} <br/>
            ({{ item.id}})
            </div>
            </b-col>
          </b-row>
          <b-row>
            <b-col><button style="height: 40px; font-size: 15px;    margin-top: 30px;   margin-left: -25px;" @click="currentMethod = 0, showModal = true">+ add project</button></b-col>
            <b-col><button style="height: 40px; font-size: 15px;    margin-top: 30px;   margin-left: -35px;" @click="currentMethod = 4, showModal = true">+ add event</button></b-col>
          </b-row>
        </div>
        
      </b-container>
    </div>
  </div>
</template>

<script>
import { FullCalendar } from 'vue-full-calendar'
import EventForm from './components/EventForm'
import DateChooser from './components/DateChooser'
import JsonCSV from 'vue-json-csv'
import moment from 'moment';
import jsPDF from 'jspdf' 

export default {
  name: 'app',
  components: {
    FullCalendar,
    EventForm,
    JsonCSV,
    DateChooser,
  },
  methods: {
    updateProjectsView() {
      var i, j;
      this.projects = [];
      for (i=0; i<this.events.length; i++) {
        var flag = true;
        for (j=0; j<this.projects.length; j++) {
          if (this.projects[j].title == this.events[i].title) {
            flag = false;
            break;
          }
        }

        if (flag) {
          var new_project = {
            id: this.events[i].id,
            title: this.events[i].title,
            backgroundColor: this.events[i].backgroundColor,
          }
          this.projects.push(new_project);
        }
      }
    },
    download() {
      let pdfName = 'report'; 
      var doc = new jsPDF();
      var text = "";
      var startDate = new Date(this.startDate);
      var endDate = new Date(this.endDate);
      var i;
      startDate.setHours(0);
      endDate.setHours(0);

      while (startDate <= endDate) {
        var nextDay = new Date(startDate);
        nextDay.setDate(startDate.getDate()+1);
        var flag = false;

        for (i=0; i<this.events.length; i++) {
          if (new Date(this.events[i].start) >= startDate && new Date(this.events[i].end) <= nextDay) {
            if (!flag) {
              text += startDate.toDateString() + '\n\n\n';
              flag = true;
            }

            text += '\t\t' + this.events[i].title + ' - ' + this.events[i].id + ' - ';
            if (this.events[i].tolls) 
            {
              text += this.events[i].tolls + ' tolls - ';
            }
            if (this.events[i].mileage) 
            {
              text += this.events[i].mileage + ' miles - ';
            }
            var duration = new Date(this.events[i].end).valueOf() - new Date(this.events[i].start).valueOf();
            duration = duration /1000/60/60; 
            text += duration + 'hrs.\n\n'
          }
        }

        if (flag) {
          text += '\n';
        }
        
        startDate = nextDay;
      }

      doc.text(text, 10, 10);
      doc.save(pdfName + '.pdf');
      this.showReportModal = false;
    },
    csvJSON(csv){

      var lines=csv.split("\n");

      var result = [];

      var headers=lines[0].split(",");
      headers[headers.length-1] = headers[headers.length-1].slice(0, headers[headers.length-1].length - 1);

      for(var i=1;i<lines.length;i++){

          var obj = {};
          var currentline=lines[i].split(",");

          for(var j=0;j<headers.length;j++){
              obj[headers[j]] = currentline[j];
          }

          result.push(obj);

      }

      //return result; //JavaScript object
      return JSON.stringify(result); //JSON
    },
    importCsv(e) {

      var file = e.target.files[0];
      var reader = new FileReader();
      var vm = this;
      reader.onload = (e) => {

        vm.fileinput = reader.result;
        

        var json =  vm.csvJSON(vm.fileinput);

        var newEvents = JSON.parse(json);

        vm.events = newEvents;
        vm.maxId += newEvents.length;
        vm.updateProjectsView();
      }
      reader.readAsText(file);

    },
    closeModal() {
      this.currentEvent = null;
      this.showModal = false;
      this.showReportModal = false;
    },
    setUpdateEvent(item) {
      let i;
      for (i=0; i<this.events.length; i++)
      {
        if (this.events[i].id === item.id) {
          this.events[i].title = item.title;
          this.events[i].start = moment(item.start).format("YYYY-MM-DD HH:mm:ss");
          this.events[i].end = moment(item.end).format("YYYY-MM-DD HH:mm:ss");
          this.events[i].backgroundColor = item.color || item.backgroundColor;
          this.events[i].phase = item.phase;
          this.events[i].mileage = item.mileage;
          this.events[i].tolls = item.tolls;
          this.events[i].description = item.description;
          break;
        }
      }
      this.updateProjectsView();
    },
    removeProject(project) {
      if (project.type == "project") {
        var i;
        for (i=this.events.length - 1;i >= 0; i--) {
          if (this.events[i].title == project.title) {
            this.events.splice(i, 1);
          }
        }

        this.updateProjectsView();
        this.currentEvent = null;
        this.showModal = false;
      } else if( project.type == "entry") {
        var i;
        for (i=0;i <this.events.length; i++) {
          if (this.events[i].title == project.title && this.events[i].id == project.id) {
            this.events.splice(i, 1);
            break;
          }
        }

        this.updateProjectsView();
        this.currentEvent = null;
        this.showModal = false;
      }
    },
    addProject(project) {
      if (this.currentMethod == 0 || this.currentMethod == 2) {
        if (project.id) {
          this.setUpdateEvent(project);
          this.currentEvent = null;
          this.showModal = false;
        } else {
          let newproject = {
            id: this.maxId + 1,
            title: project.title,
            start: project.start,
            end: project.end,
            phase: project.phase,
            mileage: project.mileage,
            tolls: project.tolls,
            description: project.description,
            backgroundColor: project.color
          };
          this.maxId ++;
          this.events.push(newproject);
          this.showModal = false;
          this.currentEvent = null;
        }
        this.updateProjectsView();
      } else if(this.currentMethod == 1){
        var i;
        for (i=0;i <this.events.length; i++) {
          if (this.events[i].title == this.currentEvent.title) {
            this.events[i].title = project.title;
            this.events[i].backgroundColor = project.color;
          }
        }

        this.updateProjectsView();
        this.currentEvent = null;
        this.showModal = false;
      } else if (this.currentMethod == 4) {
        let newproject = {
            id: this.maxId + 1,
            title: project.title,
            start: project.start,
            end: project.end,
            phase: project.phase,
            mileage: project.mileage,
            tolls: project.tolls,
            backgroundColor: project.color,
            description: project.description,
          };
          this.maxId ++;
          this.events.push(newproject);
          this.showModal = false;

        this.currentEvent = null;
      }
    },
    eventClick(item) {
      var i;
      for (i=0; i<this.events.length; i++) {
        if (this.events[i].title == item.title) {
          this.selected = this.events[i].id;
          break;
        }
      }
      this.currentEvent = item;
      this.currentMethod = 2;
      this.showModal = true;
    },
    eventDrop(item) {
      var i;
      for (i=0; i<this.events.length; i++) {
        if (this.events[i].title == item.title) {
          this.selected = this.events[i].id;
          break;
        }
      }
      this.setUpdateEvent(item);
    },
    eventResize(item) {
      var i;
      for (i=0; i<this.events.length; i++) {
        if (this.events[i].title == item.title) {
          this.selected = this.events[i].id;
          break;
        }
      }
      this.setUpdateEvent(item);
    },
    dateChanged(arg) {
      this.startDate = arg.start;
      this.endDate = arg.end;
      this.download();
    }
  },
  data () {
    return {
      currentMethod: 0,
      startDate: '2019-07-10',
      endDate: '2019-07-20',
      fileinput: null,
      currentEvent: null,
      showModal: false,
      showReportModal: false,
      selected: 0,
      maxId: 123123,
      events: [
      ],
      projects: [
      ],
      config: {
        defaultView: 'agendaWeek',
        slotDuration: '00:15:00',
        snapDuration: '00:15:00',
        selectable: false,
      },
    }
  }
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
.colorbox {
  width: 90px !important;
  height: 30px !important;
  flex: auto;
  cursor: default;
  margin-top: 10px;
}
.main {
  display: flex;
  align-items: center;
}
.calendar-holder {
  width: 100%;
}
.form-holder {
  max-width: 350px !important;
  min-width: 250px !important;
}
.form-holder > h3 {
  color: orangered;
  text-transform: uppercase;
  font-size: 16px;
  text-align: left;
  margin-left: 30px;
  margin-bottom: 10px;
}
.project-view {
  margin-left: 10%;
  width: 100%;
  height: 100%;
}
.fc-right {
  visibility: hidden;
}
.selected {
  border-style: solid;
  border-color: red;
}
a:not([href]):not([tabindex]) {
    color: azure !important;
    text-decoration: none;
}
</style>
