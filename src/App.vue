
<template>
  <div id="app">
    <div class="main">
      <div class="calendar-holder">
        <FullCalendar :config="config" :events="events" @event-selected="eventClick" @event-drop="eventDrop" @event-resize="eventResize" @event-render="eventRendered"/>
      </div>
      
      <div v-if="showModal" @close="showModal = false">
        <EventForm :event="currentEvent" :method="currentMethod" :list="projects" :events="events" @addProject="addProject" @close="closeModal" @removeProject="removeProject"/>
      </div>

      <div v-if="showReportModal" @close="showReportModal = false">
        <DateChooser :start="startDate" :end="endDate" @date_set="dateChanged" @close="closeModal"/>
      </div>

      <b-container class="form-holder" id="external-events">
        <b-row>
          <b-col>
          <JsonCSV :data   = "events"  name    = "events.csv">
            <button class="btn btn-primary">Export</button>
          </JsonCSV>
          </b-col>

          <b-col>
          <input type="file" ref="file" style="display: none" @change="importCsv">
          <button class="btn btn-primary" @click="$refs.file.click()">Import</button>
          </b-col>

          <b-col>
          <button class="btn btn-primary" @click="showReportModal = true">Report</button>
          </b-col>
        </b-row>
        <div id='external-events-listing' style="overflow: scroll; margin-top: 20px; max-height: 600px; margin-bottom: 30px; overflow-x:hidden;">
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
        </div>

        <div>
          <b-row>
            <b-col><button class="btn btn-primary" @click="currentMethod = 0, showModal = true">+ add project</button></b-col>
            <b-col><button class="btn btn-primary" @click="currentMethod = 3, showModal = true">+ add event</button></b-col>
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
import { constants } from 'crypto';

export default {
  name: 'app',
  components: {
    FullCalendar,
    EventForm,
    JsonCSV,
    DateChooser,
  },
  methods: {
    eventRendered(event, element, view){
      var text = event.title;
      if (event.tolls) 
      {
        text += '<br /> toll: ' + event.tolls;
      }
      if (event.mileage) 
      {
        text += '<br /> mileage: ' + event.mileage;
      }
      
      element[0].getElementsByClassName("fc-title")[0].innerHTML = text;
    },
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

      var enterCount = 0;
      var x = 10, y = 10;

      while (startDate <= endDate) {
        var nextDay = new Date(startDate);
        nextDay.setDate(startDate.getDate()+1);
        var flag = false;

        for (i=0; i<this.events.length; i++) {
          if (new Date(this.events[i].start) >= startDate && new Date(this.events[i].start) <= nextDay) {
            if (!flag) {
              doc.text(x, y, startDate.toDateString() + '\n\n');
              enterCount += 2;
              y += 10 * 2;
              if (enterCount >= 28) {
                enterCount = 0;
                y = 10;
                doc.addPage();
              }
              flag = true;
            }

            var j, proId;
            for (j=0;j<this.events.length; j++) {
              if (this.events[j].title == this.events[i].title)
              {
                proId = this.events[j].id;
                break;

              }
            }

            text = '\t\t' + this.events[i].title + ' - ' + proId + ' - ';
            if (this.events[i].tolls) 
            {
              text += this.events[i].tolls + ' tolls - ';
            }
            if (this.events[i].mileage) 
            {
              text += this.events[i].mileage + ' miles - ';
            }
            var duration = Math.abs(new Date(this.events[i].end) - new Date(this.events[i].start)) / 36e5;
            text += duration + ' hrs.\n'
            enterCount ++;
            doc.text(x, y, text);
            y+= 10;
            console.log(enterCount);
            if (enterCount >= 28) {
                enterCount = 0;
                y = 10;
                doc.addPage();
              }
          }
        }

        if (flag) {
          
          y+= 10;
          enterCount ++;
          if (enterCount >= 28) {
                enterCount = 0;
                y = 10;
                doc.addPage();
              }
        }
        
        startDate = nextDay;
      }


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
    get_new_id() {
      let new_id = Math.floor(Math.random() * 1000001);
          while (true) {
            let flag = false;
            let i;
            for (i=0;i <this.events.length; i++) {
              if (this.events[i].id == new_id) {
                flag = true;
                break;
              }
            }
            if (!flag) {
              break;
            }
            new_id = Math.floor(Math.random() * 1000001);
          }
          return new_id;
    },
    addProject(project) {
      if (this.currentMethod == 0 || this.currentMethod == 2) {
        if (project.id) {
          this.setUpdateEvent(project);
          this.currentEvent = null;
          this.showModal = false;
        } else {
          let newproject = {
            id: this.get_new_id(),
            title: project.title,
            start: project.start,
            end: project.end,
            phase: project.phase,
            mileage: project.mileage,
            tolls: project.tolls,
            description: project.description,
            backgroundColor: project.color
          };
          this.events.push(newproject);
          this.showModal = false;
          this.currentEvent = null;
        }
        this.updateProjectsView();
      } else if(this.currentMethod == 1){
        var i;
        var flag = true;
        for (i=0;i <this.events.length; i++) {
          if (this.events[i].title == this.currentEvent.title) {
            if (flag == true) {
              this.events[i].id = project.id;
              flag = false;
            }
            this.events[i].title = project.title;
            this.events[i].backgroundColor = project.color;
          }
        }

        this.updateProjectsView();
        this.currentEvent = null;
        this.showModal = false;
      } else if (this.currentMethod == 3) {
        let newproject = {
            id: this.get_new_id(),
            title: project.title,
            start: project.start,
            end: project.end,
            phase: project.phase,
            mileage: project.mileage,
            tolls: project.tolls,
            backgroundColor: project.color,
            description: project.description,
          };
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

.btn {
    display: inline-block;
    margin-bottom: 0;
    font-weight: 400;
    text-align: center;
    white-space: nowrap;
    vertical-align: middle;
    -ms-touch-action: manipulation;
    touch-action: manipulation;
    cursor: pointer;
    background-image: none;
    border: 1px solid transparent;
    padding: 6px 12px;
    font-size: 14px;
    line-height: 1.42857143;
    border-radius: 4px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}

.btn-info {
    color: #fff;
    background-color: #5bc0de;
    border-color: #46b8da;
}

.btn-success {
    color: #fff;
    background-color: #5cb85c;
    border-color: #4cae4c;
}

.btn-danger {
    color: #fff;
    background-color: #d9534f;
    border-color: #d43f3a;
}
</style>
