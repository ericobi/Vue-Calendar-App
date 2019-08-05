<template>
    <div class="modal-mask">
        <div class="modal-wrapper">
            <div class="modal-container">
                <h2>{{ formTitle }}</h2>

                <b-row v-if="method == 1">
                    <b-col sm="2">
                    <label >Id:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-input
                    v-model="id"
                    type="number"
                    trim
                    ></b-form-input>
                    </b-col>
                </b-row>

                <b-row v-if="method != 2 && method != 3">
                    <b-col sm="2">
                    <label >Title:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-input
                    v-model="title"
                    trim
                    autofocus
                    ></b-form-input>
                    </b-col>
                </b-row>

                <b-row v-if="method == 3">
                    <b-col sm="2">
                    <label >Title:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-select v-model="selected" :options="options"></b-form-select>
                    </b-col>
                </b-row>

                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >Start:</label>
                    </b-col>
                    <b-col sm="10">
                    <Datepicker format="YYYY-MM-DD H:i:s"  v-model="start" />
                    </b-col>
                </b-row>

                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >End:</label>
                    </b-col>
                    <b-col sm="10">
                    <Datepicker format="YYYY-MM-DD H:i:s"  v-model="end" />
                    </b-col>
                </b-row>

                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >Phase:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-input
                    v-model="phase"
                    trim
                    ></b-form-input>
                    </b-col>
                </b-row>

                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >Mileage:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-input
                    v-model="mileage"
                    type="number"
                    ></b-form-input>
                    </b-col>
                </b-row>

                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >Tolls:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-input
                    v-model="tolls"
                    trim
                    type="number"
                    ></b-form-input>
                    </b-col>
                </b-row>


                <b-row v-if="method != 1">
                    <b-col sm="2">
                    <label >Notes:</label>
                    </b-col>
                    <b-col sm="10">
                    <b-form-textarea
                    v-model="description"
                    rows="3"
                    max-rows="6"
                    ></b-form-textarea>
                    </b-col>
                </b-row>

                <b-row v-if="method != 2 && method != 3">
                    <b-col sm="2">
                    <label >Color:</label>
                    </b-col>
                    <b-col sm="10">
                    <colorpicker :color="color" v-model="color"/>
                    </b-col>
                </b-row>

                <b-row v-if="method == 1">
                    <b-col sm="2">
                    <label >Total Time:</label>
                    </b-col>
                    <b-col sm="10">
                    <label >{{ totalTime }} hrs</label>
                    </b-col>
                </b-row>


                <b-row>
                    <b-col sm="4">
                    <button class="btn btn-primary" type="button" @click="add">Save</button>
                    </b-col>
                    <b-col sm="4">
                    <button class="btn btn-primary" v-if="method == 1 || method == 2" type="button" @click="remove">Remove</button>
                    </b-col>
                    <b-col sm="4">
                    <button class="btn btn-primary" type="button" @click="close">Back</button>
                    </b-col>
                </b-row>
            </div>
        </div>
    </div>
</template>

<script>
import Datepicker from 'vuejs-datetimepicker';
import colorpicker from './ColorPicker';
import moment from 'moment';

export default {
    name: 'EventForm',
    props: ['event', 'method', 'list', 'events'],
    data(){
        return {
            formTitle: 'Add new project',
            id: '',
            title: '',
            start: '',
            end: '',
            color: '',
            phase: '',
            mileage: '',
            tolls: '',
            description: '',
            name: '',
            selected: null,
            options: null,
            totalTime: 0,
        }
    },
    created() {
        this.resetValues();
    },
    methods: {
    remove() {
        let project = {
            id: this.id,
            title: this.title
        }
        if (this.method == 1)
        {
            project.type = "project";
        } else {
            project.type = "entry";
        }
        this.$emit('removeProject', project);
    },
    add() {
        let project = {
            id: this.id,
            title: this.title,
            description: this.description,
            start: this.start,
            end: this.end,
            phase: this.phase,
            mileage: this.mileage,
            tolls: this.tolls,
            color: this.color,
        }

        if (this.method != 3) {
            var i;
            if (!project.title) {
                alert('Input project title.');
            }
            else {
                if (this.method == 0) {
                    for (i=0;i<this.list.length; i++) {
                        if (project.title == this.list[i].title) {
                            alert('Project with the same name already exists..');
                            return;
                        } 
                    }
                } 
                else if (this.event.id != project.id) {
                    if (this.method == 1) {
                        for (i=0;i<this.list.length; i++) {
                            if (project.id == this.list[i].id) {
                                alert('Entry with the same id already exists..');
                                return;
                            } 
                        }
                    }
                }
                this.$emit('addProject', project);
            }
        } else {
            if (this.selected || this.selected == 0) {
                project.title = this.list[this.selected].title;
                project.color = this.list[this.selected].backgroundColor;
                this.$emit('addProject', project);
            } else {
                alert('Select a project');
            }
        }
    },
    close() {
        this.$emit('close');
    },
    resetValues(){
        var i;
        if (this.method == 0) {
            this.formTitle = "Add New Project"
            this.id = null
            this.title= ''
            this.start= moment().format("YYYY-MM-DD HH:mm:ss")
            this.end= moment().add(2, 'h').format("YYYY-MM-DD HH:mm:ss")
            this.phase = ''
            this.mileage = ''
            this.tolls = ''
            this.color= '#FF0000'
            this.description = ''
        } else if (this.method == 1) {
            this.formTitle = "Edit Project"
            this.id = this.event.id
            this.title= this.event.title || ''
            this.color= this.event.backgroundColor || '#FF0000'
            this.totalTime = 0;
            for (i = 0; i < this.events.length; i++) {
                if (this.events[i].title == this.title) {
                    this.totalTime += Math.abs(new Date(this.events[i].end) - new Date(this.events[i].start)) / 36e5;
                }
            }
            this.totalTime = Number(this.totalTime.toFixed(2));
        } else if(this.method == 2){
            this.formTitle = "Edit Entry"
            this.id = this.event.id
            this.title= this.event.title || ''
            this.start= this.event.start.format("YYYY-MM-DD HH:mm:ss")
            this.end= this.event.end.format("YYYY-MM-DD HH:mm:ss")
            this.phase = this.event.phase || ''
            this.mileage = this.event.mileage || ''
            this.tolls = this.event.tolls || ''
            this.color= this.event.backgroundColor || '#FF0000'
            this.description = this.event.description || ''
        } else {
            this.formTitle = "Add New Event"
            this.id = null
            this.start= moment().format("YYYY-MM-DD HH:mm:ss")
            this.end= moment().add(2, 'h').format("YYYY-MM-DD HH:mm:ss")
            this.phase = ''
            this.mileage = ''
            this.tolls = ''
            this.options = [{ value: null, text: 'Please select a project' }]
            for (i = 0; i < this.list.length; i++) {
                this.options.push({value : i, text: this.list[i].title})
            }
            
        }
    }
    },
    components: {
    Datepicker,
    colorpicker
    }
}
</script>

<style scoped>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  width: 400px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
  font-family: Helvetica, Arial, sans-serif;
}
.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}

.row {
    margin-top: 10px !important;
    margin-bottom: 10px !important;
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

.btn-info {
    color: #fff;
    background-color: #5bc0de;
    border-color: #46b8da;
}

.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
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