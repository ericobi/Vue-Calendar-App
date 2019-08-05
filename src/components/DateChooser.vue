<template>
    <div class="modal-mask">
        <div class="modal-wrapper">
            <div class="modal-container">
                <h4>Report projects</h4>
                <b-row>
                    <b-col sm="2">
                    <label >Start:</label>
                    </b-col>
                    <b-col sm="10">
                    <Datepicker format="YYYY-MM-DD"  v-model="startDate" />
                    </b-col>
                </b-row>

                <b-row>
                    <b-col sm="2">
                    <label >End:</label>
                    </b-col>
                    <b-col sm="10">
                    <Datepicker format="YYYY-MM-DD"  v-model="endDate" />
                    </b-col>
                </b-row>

                <b-row>
                    <b-col sm="12">
                        <b-form-checkbox
                        v-model="condensed"
                        style="text-align: left;"
                        >
                        condensed mode
                        </b-form-checkbox>
                    </b-col>
                </b-row>

                <b-row>
                    <b-col sm="6">
                    <button class="btn btn-primary" type="button" @click="add">Okay</button>
                    </b-col>
                    <b-col sm="6">
                    <button class="btn btn-primary" type="button" @click="close">Back</button>
                    </b-col>
                </b-row>
            </div>
        </div>
    </div>
</template>

<script>
import Datepicker from './datetime_picker';

export default {
    name: 'DateChooser',
    props: ['start', 'end'],
    data(){
        return {
            startDate: '',
            endDate: '',
            condensed: false,
        }
    },
    mounted() {
        this.startDate = this.start;
        this.endDate = this.end;
    },
    methods: {
    add() {
        let project = {
            start: this.startDate,
            end: this.endDate,
            condensed: this.condensed,
        }
        this.$emit('date_set', project);
    },
    close() {
        this.$emit('close');
    },
    },
    components: {
    Datepicker
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

.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}
</style>